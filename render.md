# Continuous Deployment to Render

Step-by-step instructions for **Lesson 4.12, Parts 2 and 3** on [Render](https://render.com). Read the matching part of [lesson.md](./lesson.md) first (it explains what you are doing and why), then come here for the clicks and the exact `deploy` job. The `notify` job (Discord) is the same for every platform, so it is in lesson.md.

- **Part A** - Run your Docker Hub image on Render (lesson.md Part 2)
- **Part B** - Let CircleCI deploy to Render with the Render CLI (lesson.md Part 3)

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

**How it works:** a new `deploy` job runs after `publish` and uses the **Render CLI** (`render`) to tell Render *deploy this service now*. Render pulls the image from Docker Hub again - Render pulls the image on **every** deploy - so it gets the new `latest` that `publish` just pushed.

The CLI's `--wait` option makes the deploy job **wait until Render reports the result**: the job passes when the new version is live and fails when the deploy fails. That's what lets the `notify` job (lesson.md Part 3, Step 4) tell Discord whether the deployment really worked.

| What CircleCI needs | Where it comes from | CircleCI variable |
|---|---|---|
| An **API key** - permission to use your Render account | **Account Settings → API Keys** | `RENDER_API_KEY` |
| The **service ID** - which service to deploy | Your service's page | `RENDER_SERVICE_ID` |

> **Why not just wait for Render?** Render does **not** watch Docker Hub. Pushing a new `latest` image doesn't change anything on Render until something asks it to deploy - that's the job of the CLI.

### Step B1: Create a Render API Key

1. In the Render Dashboard, click your avatar (top right) → **"Account Settings"**
2. Scroll to **API Keys** and click **"Create API Key"**
3. **Name:** `CircleCI devops-demo`
4. **Copy the key** - it is shown in full only once!

**Treat this key like your password.** An API key can do almost anything **you** can do in Render - deploy, change or delete **any** of your services. Never put it in `config.yml` or in Git. If it leaks, revoke it in **Account Settings → API Keys** and create a new one.

<!-- INSTRUCTOR: Render docs (2026-11-10) list workspace-scoped API keys as early access. Standard keys act as the account - mention least privilege (compare Railway's project token). Delete the key after the course. -->

### Step B2: Copy Your Service ID

1. Open your **devops-demo** web service
2. Look at the browser's address bar:
   ```
   https://dashboard.render.com/web/srv-xxxxxxxxxxxxxxxxxxxx
   ```
3. Copy the part that starts with **`srv-`** - that's your service ID

The service ID is not a secret on its own, but keeping it in CircleCI means `config.yml` works for everyone without edits.

### Step B3: Add the Credentials to CircleCI

1. Open [https://app.circleci.com](https://app.circleci.com) → your devops-demo project
2. Click **"Project Settings"** → **"Environment Variables"**
3. Click **"Add Environment Variable"** twice:

| Name | Value |
|---|---|
| `RENDER_API_KEY` | Your API key from Step B1 |
| `RENDER_SERVICE_ID` | Your service ID from Step B2 (`srv-...`) |

**Why `RENDER_API_KEY`?** The Render CLI automatically looks for an API key in the `RENDER_API_KEY` environment variable - no `render login` needed.

### Step B4: Add the Deploy Job

Add this `deploy` job to `.circleci/config.yml`, after the `publish` job:

```yml
  # ==========================================
  # DEPLOY JOB - Redeploy on Render with the Render CLI
  # ==========================================
  deploy:
    docker:
      - image: cimg/base:stable
    environment:
      IMAGE_URL: docker.io/YOUR_DOCKERHUB_USERNAME/devops-demo:latest
    steps:
      - run:
          name: Install the Render CLI
          command: |
            curl -fsSL https://raw.githubusercontent.com/render-oss/cli/refs/heads/main/bin/install.sh | sh
            echo 'export PATH="$HOME/.local/bin:$PATH"' >> "$BASH_ENV"
      - run:
          name: Redeploy latest image on Render and wait
          command: |
            echo "Asking Render to deploy $IMAGE_URL..."
            render deploys create "$RENDER_SERVICE_ID" \
              --image "$IMAGE_URL" \
              --wait --confirm --output text
            echo "The new version is live on Render."
```

<!-- INSTRUCTOR: install command from Render docs (2026-11-10). install.sh installs the latest release; as a non-root user (CircleCI's `circleci` user) it puts `render` in ~/.local/bin and does not change PATH - hence the $BASH_ENV line. It looks up the latest version through the GitHub API (unauthenticated), so a rate-limited GitHub can make the install step fail. Dry-run once before class: install step, that no `render workspace set` is needed with RENDER_API_KEY, that `--image` re-pulls the new :latest, and whether deploying with `--image` also changes the Image URL saved in the service's settings. Why IMAGE_URL in config.yml rather than $DOCKER_USERNAME: DOCKER_USERNAME comes from the docker-hub context, and giving deploy that context would also hand it DOCKER_TOKEN (least privilege, Lesson 4.13). Option: publish also pushes a :$CIRCLE_SHA1 tag and deploy uses --image ...:$CIRCLE_SHA1 - every deploy then names one exact build. Lesson 4.13 can use this `curl | sh` line as an example of an unpinned tool. -->

**What this does:**
1. Uses CircleCI's small `cimg/base` image, which already has `curl` and `unzip` (the install script needs both)
2. **Install the Render CLI:** runs Render's official install script, which downloads the latest CLI and saves it as `render` in `~/.local/bin`. The `$BASH_ENV` line adds that folder to the `PATH` for the **next** steps of the job (CircleCI reads `$BASH_ENV` at the start of every step)
3. **`IMAGE_URL`** is the image to deploy - the one `publish` just pushed. **Replace `YOUR_DOCKERHUB_USERNAME`** with your Docker Hub username. It must be the same image as the **Image URL** in your Render service's settings. It's not a secret, so it can live in `config.yml`
4. The CLI reads your API key from `RENDER_API_KEY`
5. `render deploys create "$RENDER_SERVICE_ID"` starts a new deploy of your service, and `--image "$IMAGE_URL"` tells Render **exactly which image** to deploy. Render pulls it again from Docker Hub, so it gets the new `latest`
6. `--wait` keeps the job running until the deploy finishes. The CLI exits with `0` when the new version is **live**, and with a non-zero code when the deploy **fails** or the CLI can't run it (wrong API key or service ID) - so CircleCI shows the real result
7. `--confirm` skips the "are you sure?" prompt - there is nobody to answer it in CI
8. `--output text` prints plain log lines instead of the CLI's interactive view

**Notice what the deploy job does NOT need:**
- No `checkout` - nothing is built or uploaded, so the source code isn't needed
- No `render login` - the API key in `RENDER_API_KEY` is enough
- The key is never printed - CircleCI also masks environment variables in its logs

Then add the job to the workflow, as shown in lesson.md Part 3:

```yml
      - deploy:
          requires:
            - publish
```

Next, add the `notify` job - it is the same for every platform: [lesson.md Part 3, Step 4](./lesson.md#step-4-notify-the-team-on-discord).

<details>
<summary>Complete <code>.circleci/config.yml</code> for Render (with the notify job)</summary>

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
    environment:
      IMAGE_URL: docker.io/YOUR_DOCKERHUB_USERNAME/devops-demo:latest
    steps:
      - run:
          name: Install the Render CLI
          command: |
            curl -fsSL https://raw.githubusercontent.com/render-oss/cli/refs/heads/main/bin/install.sh | sh
            echo 'export PATH="$HOME/.local/bin:$PATH"' >> "$BASH_ENV"
      - run:
          name: Redeploy latest image on Render and wait
          command: |
            echo "Asking Render to deploy $IMAGE_URL..."
            render deploys create "$RENDER_SERVICE_ID" \
              --image "$IMAGE_URL" \
              --wait --confirm --output text
            echo "The new version is live on Render."

  notify:
    parameters:
      result:
        type: enum
        enum: ["success", "failed"]
    docker:
      - image: cimg/base:stable
    steps:
      - run:
          name: Send the deployment result to Discord
          command: |
            if [ "<< parameters.result >>" = "success" ]; then
              MESSAGE="✅ devops-demo deployed to production - commit ${CIRCLE_SHA1:0:7} by ${CIRCLE_USERNAME}"
            else
              MESSAGE="❌ devops-demo deployment FAILED - commit ${CIRCLE_SHA1:0:7} by ${CIRCLE_USERNAME}. Production is still running the previous version."
            fi
            curl --fail --silent --show-error \
              -H "Content-Type: application/json" \
              -d "{\"content\": \"${MESSAGE}\"}" \
              "$DISCORD_WEBHOOK_URL"
            echo "Posted to Discord: ${MESSAGE}"

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
      - notify:
          name: notify-success
          result: success
          requires:
            - deploy: success
      - notify:
          name: notify-failure
          result: failed
          requires:
            - deploy: failed
```

</details>

### Step B5: Check the Deployment on Render

After you push (lesson.md Part 3, Step 5):

1. In CircleCI, open the running **deploy** job - it stays running while Render deploys, and finishes when the new version is live
2. On Render, open the **devops-demo** web service. In **Events** (or **Deploys**), a new deploy started while the CircleCI `deploy` job was running
3. Open **Logs** - the image is pulled, not built
4. Once Render shows the service is **live**, test your URL:
   ```bash
   curl https://YOUR-SERVICE.onrender.com/hello
   ```

Go back to [lesson.md](./lesson.md) Part 3.

---

## Troubleshooting

**Deploy job fails in "Install the Render CLI"**
- **Cause:** the install script couldn't reach GitHub (for example GitHub's API rate limit), or the URL was mistyped
- **Solution:** compare the `curl` line with the job above, then click **"Rerun job"** in CircleCI - a temporary GitHub problem usually clears on its own

**Deploy job fails with `render: command not found`**
- **Cause:** the `echo 'export PATH=...' >> "$BASH_ENV"` line is missing, so the next step can't find `render` in `~/.local/bin`
- **Solution:** add the line back to the install step, exactly as above

**Deploy job fails with "unauthorized" or `401`**
- **Cause:** `RENDER_API_KEY` is wrong, missing or was revoked
- **Solution:** create a new API key (Step B1), delete `RENDER_API_KEY` in CircleCI and add it again - the whole key, no extra spaces or quotes

**Deploy job fails with an image error, or Render can't pull the image**
- **Cause:** `IMAGE_URL` still says `YOUR_DOCKERHUB_USERNAME`, or doesn't match your Docker Hub repository
- **Solution:** set `IMAGE_URL` to `docker.io/<your Docker Hub username>/devops-demo:latest` - the same image as the **Image URL** in your Render service's settings

**Deploy job fails with "not found" or `404`**
- **Cause:** `RENDER_SERVICE_ID` is wrong - for example you deleted and recreated the service - or the API key belongs to a different Render account
- **Solution:** copy the `srv-...` ID of your **current** devops-demo service (Step B2) and update CircleCI

**Deploy job fails after Render reports the deploy failed (`update_failed`, `canceled`)**
- **Cause:** CircleCI reached Render, but the new version didn't start on Render
- **Solution:** open the deploy's **Logs** on Render (see "Image Pull Failed" and "no open ports detected" below). Your previous version keeps running, and `notify-failure` posts ❌ to Discord - that's the notification working as intended

**Deploy job takes a few minutes**
- That's expected: `--wait` keeps the job running until Render has pulled the image and the new version is live

**"Image Pull Failed" on Render**
1. Check the **Image URL** in **Settings** matches Docker Hub exactly: `docker.io/YOUR_DOCKERHUB_USERNAME/devops-demo:latest`
2. Make sure the Docker Hub repository is **Public**
3. Confirm the CircleCI **publish** job pushed the image (check the `latest` tag on Docker Hub)

**Deploy succeeds but the old version is still running**
1. Check **publish** finished before **deploy** (the workflow has `requires: - publish`)
2. On Docker Hub, check `latest` was updated at the time of your push
3. On Render, check a new deploy started during the CircleCI deploy job and that it finished
4. Remember the free service may be asleep - give the first request about a minute

**Deploy fails with "no open ports detected" or the URL doesn't respond**
- Check the **Environment** page has `PORT` = `8080`, then redeploy

**Render deploys on every push, even when tests fail**
- **Cause:** the service was created from your **GitHub repository** (Git Provider), not the Docker image - Render builds and deploys every push by itself
- **Solution:** create a new web service from **Existing Image** (Step A2), update `RENDER_SERVICE_ID` with the new service's ID, and delete the Git-based service

Only CircleCI should trigger deployments.

---

## Resources

- [Render - Deploy a Prebuilt Docker Image](https://render.com/docs/deploying-an-image)
- [The Render CLI](https://render.com/docs/cli)
- [Render API keys](https://render.com/docs/api)
- [Render - Deploy for Free](https://render.com/docs/free)
- [Render Status](https://status.render.com/)
