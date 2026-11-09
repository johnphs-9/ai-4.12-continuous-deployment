# Continuous Deployment to Railway

Step-by-step instructions for **Lesson 4.12, Parts 2 and 3** on [Railway](https://railway.app). Read the matching part of [lesson.md](./lesson.md) first (it explains what you are doing and why), then come here for the clicks and the exact `deploy` job.

- **Part A** - Run your Docker Hub image on Railway (lesson.md Part 2)
- **Part B** - Let CircleCI deploy to Railway (lesson.md Part 3)

> Using Render instead? Follow [render.md](./render.md).

---

## Before You Start

- Your CircleCI pipeline from Lesson 4.7 shows build ✅ → test ✅ → publish ✅
- `YOUR_DOCKERHUB_USERNAME/devops-demo:latest` is on Docker Hub, in a **Public** repository - pulling from a private registry needs a paid Railway plan

---

## Part A - Run Your Docker Hub Image on Railway

### Step A1: Log In to Railway

**If you already have a Railway account from Lesson 4.10:** just log in at [https://railway.app](https://railway.app).

**If you don't:**

1. Go to [https://railway.app](https://railway.app)
2. Click **"Login"** or **"Start a New Project"**
3. Sign up with **GitHub** (recommended)
4. Verify your email if prompted

### Step A2: Create a Service from Your Docker Image

1. Click **"New Project"** (or "Start a New Project")
2. Select **"Deploy from Docker Image"**
3. Enter your image name **without** a `docker.io/` prefix:
   ```
   YOUR_DOCKERHUB_USERNAME/devops-demo:latest
   ```
4. Click **"Deploy"**

Railway pulls the image from Docker Hub and starts a container. There is **no build step** - the image is already built.

5. If the project is marked as temporary, click **"Claim Project"** to keep it
6. (Optional) Click the service → **Settings** → rename it to `devops-demo`

> **Do not choose "Deploy from GitHub repo".** That makes Railway build and deploy every push by itself - before your tests have run, and outside your CI pipeline. In this lesson, **only CircleCI** decides when to deploy.

### Step A3: Generate a Public Domain

1. Click on the **devops-demo** service (in the project view)
2. Click the **"Settings"** tab
3. Scroll down to the **"Networking"** section
4. Under **"Public Networking"**, click **"Generate Domain"**
5. In the dialog, enter **Port** `8080` (Spring Boot default port) and click **"Generate Domain"**

Railway assigns a URL like:
```
https://devops-demo-production.up.railway.app
```

### Step A4: Check the Logs and Test

Click the **"Deployments"** tab, then open the latest deployment. You should see the image being pulled (not built), then Spring Boot starting:

```
Pulling image YOUR_DOCKERHUB_USERNAME/devops-demo:latest
...
Started DevopsDemoApplication in X.XXX seconds
```

Then test it:

```bash
curl https://YOUR-RAILWAY-URL.up.railway.app/hello
```

You should see: **"DevOps demo application is running!"** ✅ Go back to [lesson.md](./lesson.md) Part 2.

---

## Part B - Let CircleCI Deploy to Railway

**How it works:** a new `deploy` job runs after `publish` and uses the **Railway CLI** to tell Railway *redeploy this service*. Railway then pulls the latest image from Docker Hub and replaces the running container.

| What CircleCI needs | Where it comes from | CircleCI variable |
|---|---|---|
| A **project token** - permission to deploy to this project | Railway project settings | `RAILWAY_TOKEN` |
| The **service ID** - which service to redeploy | The service's variables | `RAILWAY_SERVICE_ID` |

### Step B1: Create a Railway Project Token

A **project token** can only deploy to one project and environment - safer for CI than a token for your whole account.

1. Open your Railway project
2. Click **"Settings"** (project settings, not service settings)
3. Click the **"Tokens"** tab
4. Click **"Create Token"**:
   - **Name:** `CircleCI Deployment`
   - **Environment:** `production`
5. **Copy the token** - it won't be shown again!

### Step B2: Get Your Railway Service ID

1. Click on your **devops-demo** service
2. Click the **"Variables"** tab
3. Expand the variables **added by Railway**
4. Find `RAILWAY_SERVICE_ID` and copy its value (format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)

You don't need the project ID - the project token already identifies the project and environment.

### Step B3: Add the Credentials to CircleCI

1. Open [https://app.circleci.com](https://app.circleci.com) → your devops-demo project
2. Click **"Project Settings"** → **"Environment Variables"**
3. Click **"Add Environment Variable"** twice:

| Name | Value |
|---|---|
| `RAILWAY_TOKEN` | Your project token from Step B1 |
| `RAILWAY_SERVICE_ID` | Your service ID from Step B2 |

**Why `RAILWAY_TOKEN`?** The Railway CLI automatically looks for a project token in the `RAILWAY_TOKEN` environment variable - no `railway login` needed.

### Step B4: Add the Deploy Job

Add this `deploy` job to `.circleci/config.yml`, after the `publish` job:

```yml
  # ==========================================
  # DEPLOY JOB - Tell Railway to redeploy
  # ==========================================
  deploy:
    docker:
      - image: cimg/node:lts
    steps:
      - run:
          name: Redeploy latest image on Railway
          command: |
            echo "Asking Railway to redeploy the latest image..."
            npx --yes @railway/cli redeploy --service "$RAILWAY_SERVICE_ID" --yes
            echo "Railway is now pulling $DOCKER_USERNAME/devops-demo:latest from Docker Hub."
```

<!-- INSTRUCTOR TODO: confirm that `railway redeploy` pulls the NEW :latest image from Docker Hub (not the previously pulled one). If it does not, switch to commit-SHA image tags and update the service image via the Railway API. -->

**What this does:**
1. Uses a CircleCI image that has Node.js, so `npx` can run the Railway CLI without installing it
2. The CLI reads your project token from `RAILWAY_TOKEN`
3. `redeploy --service` tells Railway to redeploy your devops-demo service
4. `--yes` skips the "are you sure?" prompt - there is nobody to answer it in CI
5. Railway pulls the latest image from Docker Hub and replaces the running container

**Notice what the deploy job does NOT need:**
- No `checkout` - nothing is built or uploaded, so the source code isn't needed
- No `railway login` or `railway link` - the project token is enough

Then add the job to the workflow, as shown in lesson.md Part 3:

```yml
      - deploy:
          requires:
            - publish
```

Next, add the `notify` job - it is the same for every platform: [lesson.md Part 3, Step 4](./lesson.md#step-4-notify-the-team-on-discord).

<details>
<summary>Complete <code>.circleci/config.yml</code> for Railway (with the notify job)</summary>

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
      - image: cimg/node:lts
    steps:
      - run:
          name: Redeploy latest image on Railway
          command: |
            echo "Asking Railway to redeploy the latest image..."
            npx --yes @railway/cli redeploy --service "$RAILWAY_SERVICE_ID" --yes
            echo "Railway is now pulling $DOCKER_USERNAME/devops-demo:latest from Docker Hub."

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

### Step B5: Check the Deployment on Railway

After you push (lesson.md Part 3, Step 5):

1. Open your Railway project → the **devops-demo** service → **"Deployments"** tab
2. A new deployment starts just after the CircleCI `deploy` job
3. Open its logs - the image is pulled, not built
4. Once it shows **"Active"**, test your URL:
   ```bash
   curl https://YOUR-RAILWAY-URL.up.railway.app/hello
   ```

Go back to [lesson.md](./lesson.md) Part 3.

---

## Troubleshooting

**Deploy job fails - "Unauthorized" or "Invalid token"**
1. Verify `RAILWAY_TOKEN` in CircleCI (no extra spaces)
2. Make sure it is a **project token** (Project Settings → Tokens), not an account token
3. Check the token was created for the **production** environment
4. Regenerate the token in Railway if needed and update CircleCI

**"Service not found"**
1. Verify `RAILWAY_SERVICE_ID` in CircleCI - a UUID like `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
2. Copy it again from the **service's** variables (not the project settings)
3. Make sure the project token belongs to the same project as the service

**Railway can't pull the image - "not found" or "access denied"**
1. Check the image name in the service **Settings → Source** matches Docker Hub exactly: `YOUR_DOCKERHUB_USERNAME/devops-demo:latest`
2. Make sure the Docker Hub repository is **Public**
3. Confirm the CircleCI **publish** job pushed the image (check the `latest` tag on Docker Hub)

**Deploy succeeds but the old version is still running**
1. Check **publish** finished before **deploy** (the workflow has `requires: - publish`)
2. On Docker Hub, check `latest` was updated at the time of your push
3. In Railway → Deployments, check a new deployment started after the deploy job

**`/hello` returns 404 or the URL doesn't respond**
1. Check the Railway logs show "Started DevopsDemoApplication"
2. Check the port in **Settings → Networking** is `8080`

**Railway deploys on every push, even when tests fail**

Your service is connected to your GitHub repository instead of the Docker image:
1. Open the service → **Settings** → **Source**
2. Disconnect the GitHub repository
3. Set the source to your Docker image: `YOUR_DOCKERHUB_USERNAME/devops-demo:latest`

Only CircleCI should trigger deployments.

---

## Resources

- [Railway Docs](https://docs.railway.app/)
- [Deploying with the Railway CLI](https://docs.railway.com/cli/deploying)
- [Railway Status](https://status.railway.app/)
