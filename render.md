# Continuous Deployment to Render

Step-by-step instructions for **Lesson 4.12, Parts 2 and 3** on [Render](https://render.com). Read the matching part of [lesson.md](./lesson.md) first (it explains what you are doing and why), then come here for the clicks and the exact `deploy` job.

- **Part A** - Run your Docker Hub image on Render (lesson.md Part 2)
- **Part B** - Let CircleCI deploy to Render (lesson.md Part 3)

> Using Railway instead? Follow [railway.md](./railway.md).

---

## Before You Start

- Your CircleCI pipeline from Lesson 4.7 shows build ✅ → test ✅ → publish ✅
- `YOUR_DOCKERHUB_USERNAME/devops-demo:latest` is on Docker Hub, in a **Public** repository - a private image needs registry credentials on Render
- The image is built for `linux/amd64` - Render only runs `linux/amd64` images. CircleCI's `publish` job runs on amd64 machines, so your CI image already is

**Free plan reminder:** a free Render web service **spins down after 15 minutes without traffic**. The next request wakes it up, which takes about a minute. That's normal - if your URL is slow, wait and try again.

---

## Part A - Run Your Docker Hub Image on Render

### Step A1: Log In to Render

**If you already have a Render account from Lesson 4.10:** just log in at [https://render.com](https://render.com).

**If you don't:**

1. Go to [https://render.com](https://render.com)
2. Click **"Get Started"** (or **"Sign In"**)
3. Sign up with **GitHub** (recommended) or your email
4. Verify your email if prompted

> Still have the devops-demo service from Lesson 4.10? It runs a different image (`v4.10.x`). Create a **new** service below, and delete the old one if you like (**Settings → Delete Web Service**).

### Step A2: Create a Web Service from Your Docker Image

1. In the Dashboard, click **"+ New"** → **"Web Service"**
2. Under **Source Code**, click **"Existing Image"**
3. In **Image URL**, enter:
   ```
   docker.io/YOUR_DOCKERHUB_USERNAME/devops-demo:latest
   ```
   (`docker.io/` is optional. Leave the credential empty for a public image.)
4. Click **"Connect"**. Render checks that it can pull the image.
5. Fill in the service details:
   - **Name:** `devops-demo`
   - **Region:** **Singapore** (or the region closest to you)
   - **Instance Type:** **Free**
6. Under **Environment Variables**, add:

   | Key | Value |
   |---|---|
   | `PORT` | `8080` |

   Render sends web traffic to the port in `PORT` (`10000` by default). Spring Boot listens on `8080`.

7. Click **"Deploy Web Service"** (or **"Deploy"**)

Render pulls the image from Docker Hub and starts a container. There is **no build step** - the image is already built.

> **Do not choose "Git Provider" as the source.** That makes Render build and deploy every push by itself - before your tests have run, and outside your CI pipeline. In this lesson, **only CircleCI** decides when to deploy.

### Step A3: Check the Logs and Test

Click **"Logs"** in the left pane. You should see the image being pulled (not built), then Spring Boot starting:

```
==> Pulling image docker.io/YOUR_DOCKERHUB_USERNAME/devops-demo:latest
...
Started DevopsDemoApplication in X.XXX seconds
==> Your service is live 🎉
```

Copy your URL from the top of the service page (`https://devops-demo-xxxx.onrender.com`) and test it:

```bash
curl https://YOUR-SERVICE.onrender.com/hello
```

You should see: **"DevOps demo application is running!"** ✅ Go back to [lesson.md](./lesson.md) Part 2.

---

## Part B - Let CircleCI Deploy to Render

**How it works:** every Render service has a secret **deploy hook URL**. Sending a request to it tells Render *deploy this service now*. A new `deploy` job runs after `publish` and calls the hook with `curl`. Render then pulls the image from Docker Hub again - Render pulls the image on **every** deploy - so it gets the new `latest` that `publish` just pushed.

| What CircleCI needs | Where it comes from | CircleCI variable |
|---|---|---|
| The **deploy hook URL** - permission to deploy *and* which service | The service's **Settings** | `RENDER_DEPLOY_HOOK_URL` |

> **Why not just wait for Render?** Render does **not** watch Docker Hub. Pushing a new `latest` image doesn't change anything on Render until something asks it to deploy - that's the job of the deploy hook.

### Step B1: Copy the Deploy Hook URL

1. Open your **devops-demo** web service
2. Click **"Settings"**
3. Find **Deploy Hook** and copy the URL. It looks like:
   ```
   https://api.render.com/deploy/srv-xxxxxxxxxxxx?key=yyyyyyyyyyyy
   ```

**Treat this URL like a password.** Anyone who has it can deploy your service. Never put it in `config.yml` or in Git. If it leaks, click **"Regenerate Hook"** and update CircleCI.

### Step B2: Add the Deploy Hook to CircleCI

1. Open [https://app.circleci.com](https://app.circleci.com) → your devops-demo project
2. Click **"Project Settings"** → **"Environment Variables"**
3. Click **"Add Environment Variable"**:

| Name | Value |
|---|---|
| `RENDER_DEPLOY_HOOK_URL` | Your deploy hook URL from Step B1 |

### Step B3: Add the Deploy Job

Add this `deploy` job to `.circleci/config.yml`, after the `publish` job:

```yml
  # ==========================================
  # DEPLOY JOB - Tell Render to redeploy
  # ==========================================
  deploy:
    docker:
      - image: cimg/base:stable
    steps:
      - run:
          name: Redeploy latest image on Render
          command: |
            echo "Asking Render to redeploy the latest image..."
            curl --fail --silent --show-error -X POST "$RENDER_DEPLOY_HOOK_URL"
            echo ""
            echo "Render is now pulling the latest devops-demo image from Docker Hub."
```

<!-- INSTRUCTOR: Render docs (deploy hooks, 2026-11-04): with no imgURL parameter the hook deploys the tag in the service settings, and Render pulls the image on every deploy - so this picks up the new :latest. A successful call returns 200 with {"deploy": {"id": "dep-..."}}. Dry-run once before class. -->

**What this does:**
1. Uses CircleCI's small `cimg/base` image, which already has `curl`
2. `curl -X POST` calls your deploy hook. Render starts a new deploy and replies with the deploy's ID, e.g. `{"deploy":{"id":"dep-..."}}`
3. `--fail` makes the job **fail** if Render rejects the call (for example a wrong or regenerated URL returns `401`) - without it, CircleCI would show green even though nothing was deployed
4. `--silent --show-error` hides the progress bar but still prints errors
5. Render pulls `devops-demo:latest` from Docker Hub and replaces the running container

**Notice what the deploy job does NOT need:**
- No `checkout` - nothing is built or uploaded, so the source code isn't needed
- No Render CLI or login - the secret in the hook URL is enough
- The URL is never printed - CircleCI also masks environment variables in its logs

Then add the job to the workflow, as shown in lesson.md Part 3:

```yml
      - deploy:
          requires:
            - publish
```

<details>
<summary>Complete <code>.circleci/config.yml</code> for Render</summary>

```yml
version: 2.1

jobs:
  build:
    docker:
      - image: cimg/openjdk:21.0
    steps:
      - checkout
      - restore_cache:
          keys:
            - maven-deps-{{ checksum "pom.xml" }}
            - maven-deps-
      - run:
          name: Install dependencies and build
          command: |
            echo "Building the application..."
            mvn clean install -DskipTests
      - save_cache:
          paths:
            - ~/.m2
          key: maven-deps-{{ checksum "pom.xml" }}
      - persist_to_workspace:
          root: .
          paths:
            - target/*.jar

  test:
    docker:
      - image: cimg/openjdk:21.0
    steps:
      - checkout
      - restore_cache:
          keys:
            - maven-deps-{{ checksum "pom.xml" }}
            - maven-deps-
      - run:
          name: Run tests
          command: |
            echo "Running tests..."
            mvn test
      - store_test_results:
          path: target/surefire-reports
      - store_artifacts:
          path: target/surefire-reports

  publish:
    docker:
      - image: cimg/base:stable
    steps:
      - checkout
      - attach_workspace:
          at: .
      - setup_remote_docker
      - run:
          name: Build Docker image
          command: |
            echo "Building Docker image..."
            docker build -t $DOCKER_USERNAME/devops-demo:latest .
      - run:
          name: Push to Docker Hub
          command: |
            echo "Logging in to Docker Hub..."
            echo $DOCKER_TOKEN | docker login -u $DOCKER_USERNAME --password-stdin
            echo "Pushing image to Docker Hub..."
            docker push $DOCKER_USERNAME/devops-demo:latest

  deploy:
    docker:
      - image: cimg/base:stable
    steps:
      - run:
          name: Redeploy latest image on Render
          command: |
            echo "Asking Render to redeploy the latest image..."
            curl --fail --silent --show-error -X POST "$RENDER_DEPLOY_HOOK_URL"
            echo ""
            echo "Render is now pulling the latest devops-demo image from Docker Hub."

workflows:
  build_test_publish_deploy:
    jobs:
      - build
      - test:
          requires:
            - build
      - publish:
          context: docker-hub
          requires:
            - test
      - deploy:
          requires:
            - publish
```

</details>

### Step B4: Check the Deployment on Render

After you push (lesson.md Part 3, Step 4):

1. Open the **devops-demo** web service on Render
2. In **Events** (or **Deploys**), a new deploy starts just after the CircleCI `deploy` job
3. Open **Logs** - the image is pulled, not built
4. Once Render shows the service is **live**, test your URL:
   ```bash
   curl https://YOUR-SERVICE.onrender.com/hello
   ```

Go back to [lesson.md](./lesson.md) Part 3.

---

## Troubleshooting

**Deploy job fails with `401`**
- **Cause:** the `key` in the hook URL is wrong or missing - often because the hook was regenerated
- **Solution:** copy the deploy hook URL from **Settings** again and update `RENDER_DEPLOY_HOOK_URL` in CircleCI (the whole URL, including `?key=...`, no extra spaces or quotes)

**Deploy job fails with `404`**
- **Cause:** the service in the URL doesn't exist - for example you deleted and recreated the service
- **Solution:** copy the hook URL of your **current** devops-demo service

**Deploy job fails with `curl: (3) URL using bad/illegal format or missing URL`**
- **Cause:** `RENDER_DEPLOY_HOOK_URL` isn't set in CircleCI, or the name is misspelled
- **Solution:** check **Project Settings → Environment Variables** - the name must match exactly

**"Image Pull Failed" on Render**
1. Check the **Image URL** in **Settings** matches Docker Hub exactly: `docker.io/YOUR_DOCKERHUB_USERNAME/devops-demo:latest`
2. Make sure the Docker Hub repository is **Public**
3. Confirm the CircleCI **publish** job pushed the image (check the `latest` tag on Docker Hub)

**Deploy succeeds but the old version is still running**
1. Check **publish** finished before **deploy** (the workflow has `requires: - publish`)
2. On Docker Hub, check `latest` was updated at the time of your push
3. On Render, check a new deploy started after the CircleCI deploy job and that it finished
4. Remember the free service may be asleep - give the first request about a minute

**Deploy fails with "no open ports detected" or the URL doesn't respond**
- Check the **Environment** page has `PORT` = `8080`, then redeploy

**Render deploys on every push, even when tests fail**
- **Cause:** the service was created from your **GitHub repository** (Git Provider), not the Docker image - Render builds and deploys every push by itself
- **Solution:** create a new web service from **Existing Image** (Step A2), update `RENDER_DEPLOY_HOOK_URL` with the new service's hook, and delete the Git-based service

Only CircleCI should trigger deployments.

---

## Resources

- [Render - Deploy a Prebuilt Docker Image](https://render.com/docs/deploying-an-image)
- [Render - Deploy Hooks](https://render.com/docs/deploy-hooks)
- [Render - Deploy for Free](https://render.com/docs/free)
- [Render Status](https://status.render.com/)
