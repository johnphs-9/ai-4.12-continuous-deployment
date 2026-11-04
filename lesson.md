# Lesson 4.12: Continuous Deployment - Automated Deployment to the Cloud

## Learning Objectives

By the end of this lesson, you will be able to:
1. **Distinguish** between Continuous Delivery and Continuous Deployment
2. **Deploy** a Docker image from Docker Hub to a cloud platform (such as Render or Railway)
3. **Configure** CircleCI to automatically deploy applications after successful tests
4. **Implement** a complete CI/CD pipeline from code push to production

---

## Prerequisites

Before starting this lesson, ensure you have:
- Completed Lesson 4.7 (Continuous Integration with CircleCI)
- Completed Lesson 4.10 (Hosting Options - Manual Hosting), with a Render or Railway account
- Your **devops-demo** GitHub repository from Lesson 4.7, connected to CircleCI
- Docker Hub account with your `devops-demo:latest` image published (public repository)
- CircleCI pipeline showing build ✅ → test ✅ → publish ✅

---

## Introduction

In Lesson 4.7, you built a CI pipeline that automatically builds, tests, and publishes Docker images when you push code to GitHub.

**But there's one problem:** The Docker image sits in Docker Hub. Your application is NOT running anywhere. Users can't access it!

**Today, you'll complete the pipeline:**
- Code push → GitHub
- CircleCI builds, tests, publishes
- **CircleCI tells your cloud platform to deploy the new image** ← NEW!
- Users access your live application

**This is Continuous Deployment (CD)!**

---

## Part 1 - Understanding Continuous Delivery vs Continuous Deployment

### What is Continuous Delivery?

**Continuous Delivery (CD)** is an extension of Continuous Integration. After building and testing, code changes are **automatically prepared** for release to production, but **deployment requires manual approval**.

**Flow:**
```
Code Push → Build → Test → Ready to Deploy → [Manual Button Click] → Production
```

**Example:**
- CircleCI builds and tests successfully
- Deployment package is ready
- Team lead clicks "Deploy to Production" button
- Application goes live

**Why manual approval?**
- Critical applications (banking, healthcare)
- Compliance requirements
- Business timing (deploy during maintenance window)

---

### What is Continuous Deployment?

**Continuous Deployment** goes one step further: **Every** code change that passes tests is **automatically deployed** to production. No manual intervention.

**Flow:**
```
Code Push → Build → Test → Automatically Deploy → Production
```

**Example:**
- Developer pushes code to GitHub
- CircleCI builds and tests
- If tests pass, **automatically** deploys to production
- Users see changes within minutes

**When to use:**
- Startups moving fast
- Non-critical applications
- Teams with strong testing
- DevOps culture

---

### Principles of Continuous Delivery/Deployment

**1. Repeatable Reliable Process**
- Deployment should work the same way every time
- No manual steps that can be forgotten
- Automated deployment scripts

**2. Automate Everything**
- Build, test, deploy - all automated
- No human errors
- Fast and consistent

**3. Version Control Everything**
- Code, configuration, infrastructure
- Can rollback to any previous version
- Track what changed when

**4. Build Quality In**
- Comprehensive tests (unit, integration, E2E)
- Code reviews before merge
- Automated quality checks

**5. Do the Hardest Parts First**
- Deploy frequently (reduces risk)
- Small changes easier to debug
- Don't wait for "big bang" release

**6. Everyone is Responsible**
- Developers own deployment
- No "throw over the wall" to ops
- DevOps culture

**7. "Done" Means Released**
- Feature isn't done until users have it
- Not done when code is merged
- Done when deployed to production

**8. Continuous Improvement**
- Monitor, learn, improve
- Faster deployments over time
- Reduce deployment failures

---

### Benefits of Continuous Deployment

**1. Automate the Software Release Process**
- No manual deployment steps
- Deployment happens in minutes, not days
- Consistent process every time

**2. Improve Developer Productivity**
- Developers focus on features, not deployment
- No waiting for deployment windows
- Fast feedback loop

**3. Find and Address Bugs Quicker**
- Small changes easier to debug
- Issues caught immediately
- Quick rollbacks if needed

**4. Deliver Updates Faster**
- Features reach users immediately
- Competitive advantage
- Rapid iteration based on feedback

---

### CD Pipeline

A typical Continuous Deployment pipeline contains these stages:

**1. Detect New Release**
- Triggered by code push or successful CI
- Identifies what to deploy (Docker image tag, commit hash)

**2. Pull Image from Container Registry**
- Gets the Docker image built by CI
- Verifies image integrity

**3. Deploy to Environment**
- Stops old version
- Starts new version
- Health checks verify deployment

**4. Post-Deployment**
- Run smoke tests
- Monitor for errors
- Notify team of deployment status

---

### Deployment Environments

Professional teams typically have multiple environments:

**Development**
- Developers' local machines
- Rapid changes, may be unstable
- Connected to local/test database

**Staging**
- Mirror of production environment
- User Acceptance Testing (UAT) by stakeholders
- Test with production-like data
- Final verification before production

**Production**
- Live environment where real users access the application
- Must be stable and monitored
- Zero downtime deployments preferred

**Disaster Recovery (DR)**
- Backup environment in different geographic location
- Activated if production fails (earthquake, fire, server failure)
- Keeps business running during disasters

**For this lesson:** We'll deploy directly to **Production** on your cloud platform (simple setup for learning).

---

### Use Case: A/B Testing with Continuous Deployment

**What is A/B Testing?**
- Show two versions of a feature to different users
- Measure which performs better
- Make data-driven decisions

**Example:**
- Version A: Blue "Buy Now" button
- Version B: Green "Buy Now" button
- Deploy both, measure which gets more clicks

**How CD Enables A/B Testing:**
- Change one variable (button color)
- Push code → Automatic deployment
- Users immediately see new version
- Collect metrics quickly
- Deploy winning version to all users

**Without CD:** Would take days/weeks to deploy and test versions.
**With CD:** Deploy and test in hours.

---

## Part 2 - Run Your Docker Hub Image on a Cloud Platform

In Lesson 4.7, you configured CircleCI to build, test and push your image to Docker Hub. In Lesson 4.10, you deployed a Docker image to a cloud platform by hand. In this lesson, CircleCI will deploy to the cloud platform for you, so that it runs the **same image** your pipeline tested and published.

```
CircleCI: build → test → publish ──► Docker Hub ──► Cloud platform pulls the image and runs it
```

**Why deploy the image instead of the source code?**
- The image that passed your tests is **exactly** the image that runs in production
- The image is built once, by CI - the platform only pulls and runs it
- Your single-stage Dockerfile from Lesson 4.7 stays as it is

Use the same platform as in Lesson 4.10. The steps below explain **what** you do and **why**; the guide for your platform shows you **where to click** and gives you the exact `deploy` job:

| Platform | Step-by-step guide |
|---|---|
| **Render** | [render.md](./render.md) - Part A (this part) and Part B (Part 3) |
| **Railway** | [railway.md](./railway.md) - Part A (this part) and Part B (Part 3) |

<!-- INSTRUCTOR: tell the class which platform to use - the same one as Lesson 4.10 (Render is the current default). To support another platform, add a <platform>.md guide with the same Part A / Part B structure and list it here. -->

---

### Step 1: Check Your devops-demo Project and Docker Hub Image

Use your **`devops-demo`** repository from Lesson 4.7 - the one connected to CircleCI. You do not need a new repository or any changes to your Dockerfile.

**1.1: Verify the CircleCI pipeline works**

1. Go to [https://app.circleci.com](https://app.circleci.com)
2. Open your devops-demo project
3. Ensure the latest pipeline shows: build ✅ → test ✅ → publish ✅

**If the pipeline is broken, fix it before proceeding!**

**1.2: Check your image on Docker Hub**

1. Go to [https://hub.docker.com](https://hub.docker.com) and open your repositories
2. Find **`devops-demo`** with the tag **`latest`**
3. Make sure the repository is **Public** (Repository → Settings → Visibility)

**Why public?** The platform pulls your image from Docker Hub. Pulling from a private registry needs extra setup (and on some platforms, a paid plan).

Note your full image name - you will need it in Step 3:

```
YOUR_DOCKERHUB_USERNAME/devops-demo:latest
```

---

### Step 2: Log In to the Platform

Use the account you created in Lesson 4.10. If you don't have one yet, sign up - **GitHub** sign-up is the quickest.

➡️ **Render:** [render.md - Step A1](./render.md#step-a1-log-in-to-render) · **Railway:** [railway.md - Step A1](./railway.md#step-a1-log-in-to-railway)

---

### Step 3: Create a Service from Your Docker Image

This is the same as Lesson 4.10 Part 3 - but this time with the `latest` image your CI pipeline publishes:

```
YOUR_DOCKERHUB_USERNAME/devops-demo:latest
```

1. Create a new service from an **existing Docker image** and enter the image name above
2. Make sure the platform sends web traffic to port **8080** (Spring Boot's port)
3. Give the service a **public URL**

The platform pulls the image from Docker Hub and starts a container. There is **no build step** - the image is already built.

> **Do not deploy from your GitHub repository.** Both Render and Railway can build and deploy every push by themselves - but that happens before your tests have run, and outside your CI pipeline. In this lesson, **only CircleCI** decides when to deploy.

➡️ **Render:** [render.md - Step A2](./render.md#step-a2-create-a-web-service-from-your-docker-image) · **Railway:** [railway.md - Steps A2-A3](./railway.md#step-a2-create-a-service-from-your-docker-image)

---

### Step 4: Verify the Deployment

**Watch the logs:** you should see the image being **pulled** (not built), then Spring Boot starting:

```
Pulling image YOUR_DOCKERHUB_USERNAME/devops-demo:latest
...
Started DevopsDemoApplication in X.XXX seconds
```

**Test your application:**

```bash
curl https://YOUR-SERVICE-URL/hello
```

You should see: **"DevOps demo application is running!"**

➡️ **Render:** [render.md - Step A3](./render.md#step-a3-check-the-logs-and-test) · **Railway:** [railway.md - Step A4](./railway.md#step-a4-check-the-logs-and-test)

**Success!** The platform is running the image your CI pipeline published.

**But this is not continuous deployment yet.** When you push new code, CircleCI publishes a new image to Docker Hub - and the platform does nothing. Someone would have to redeploy by hand. In Part 3, you make deployment the **last step of the CI pipeline**.

---

## Part 3 - Automate Deployment with CircleCI

### Understanding the Automation Flow

**Current state:**
```
Code Push → GitHub → CircleCI → Build → Test → Publish to Docker Hub → STOPS
```

**What we want:**
```
Code Push → GitHub → CircleCI → Build → Test → Publish → Deploy → LIVE!
```

---

### How CircleCI Triggers the Platform

CircleCI cannot "push" an image into the platform. Instead, a new **deploy** job runs after **publish** and tells the platform: *redeploy this service*. The platform then pulls the latest image from Docker Hub and replaces the running container.

**How it works:**
1. CircleCI builds, tests and publishes the new image to Docker Hub
2. If everything passed, the **deploy** job calls the platform's deploy API
3. The call is authenticated with a **secret** that only allows deploying - stored in CircleCI, never in your code
4. The platform pulls `devops-demo:latest` from Docker Hub and restarts the service

If **build**, **test** or **publish** fails, the deploy job never runs - broken code never reaches production.

**Same idea, different tools:**

| | Render | Railway |
|---|---|---|
| Secret that allows the deploy | **Deploy hook URL** (the service's Settings) | **Project token** (project Settings → Tokens) |
| Which service to deploy | Part of the hook URL | **Service ID** |
| CircleCI environment variables | `RENDER_DEPLOY_HOOK_URL` | `RAILWAY_TOKEN`, `RAILWAY_SERVICE_ID` |
| What the deploy job runs | `curl -X POST` to the hook URL | `railway redeploy` (Railway CLI) |

---

### Step 1: Get a Deploy Credential from the Platform

Create (or copy) the secret CircleCI will use to trigger a deploy. **Copy it straight into CircleCI in the next step** - treat it like a password.

➡️ **Render:** [render.md - Step B1](./render.md#step-b1-copy-the-deploy-hook-url) · **Railway:** [railway.md - Steps B1-B2](./railway.md#step-b1-create-a-railway-project-token)

---

### Step 2: Add the Credentials to CircleCI

1. Open [https://app.circleci.com](https://app.circleci.com) → your devops-demo project
2. Click **"Project Settings"** → **"Environment Variables"**
3. Add the variable(s) for your platform (see the table above)

**Why environment variables?**
- Keep credentials secret
- Don't expose them in code or in Git
- Easy to change if needed (for example, if a secret leaks and you regenerate it)

➡️ **Render:** [render.md - Step B2](./render.md#step-b2-add-the-deploy-hook-to-circleci) · **Railway:** [railway.md - Step B3](./railway.md#step-b3-add-the-credentials-to-circleci)

---

### Step 3: Update CircleCI Configuration

**3.1: Open Your Project**

Open your devops-demo project in VS Code (or your editor), then open `.circleci/config.yml`.

**Your current config should look like this (from Lesson 4.7):**

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

workflows:
  build_test_publish:
    jobs:
      - build
      - test:
          requires:
            - build
      - publish:
          context: docker-hub
          requires:
            - test
```

**3.2: Add a Deploy Job**

Add a new `deploy` job after the `publish` job. Every platform's deploy job has the same shape - only the command is different:

```yml
  deploy:
    docker:
      - image: <a small image with the tool you need>
    steps:
      - run:
          name: Redeploy latest image on <platform>
          command: <ask the platform to redeploy the service>
```

Copy the exact job for your platform:

➡️ **Render:** [render.md - Step B3](./render.md#step-b3-add-the-deploy-job) · **Railway:** [railway.md - Step B4](./railway.md#step-b4-add-the-deploy-job)

**Notice what the deploy job does NOT need:**
- No `checkout` - nothing is built or uploaded, so the source code isn't needed
- No passwords in the file - the secret comes from the CircleCI environment variables

**3.3: Update the Workflow**

Add the deploy job to the end of your workflow, so it runs only after **publish** succeeds:

```yml
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

**Flow:** build → test → publish → deploy

Your platform guide has the complete `config.yml` if you want to compare.

---

### Step 4: Commit and Push Changes

```bash
git add .circleci/config.yml
git commit -m "Add CD: deploy automatically after publish"
git push origin main
```

**Watch the Magic! ✨**

1. Go to CircleCI: [https://app.circleci.com](https://app.circleci.com)
2. Your pipeline should trigger automatically
3. Watch all jobs run:
   - ⏳ build (compiling...)
   - ⏳ test (running tests...)
   - ⏳ publish (pushing to Docker Hub...)
   - ⏳ deploy (asking the platform to redeploy...)

**Timeline:**
- Build: ~2-3 minutes
- Test: ~2 minutes
- Publish: ~2-3 minutes
- Deploy job: < 1 minute (one API call)
- **Platform deployment:** ~1-2 minutes (pull image and start - no build)

**Total:** ~7-10 minutes from push to production!

---

### Step 5: Verify Deployment

**5.1: Check CircleCI**

All 4 jobs should be green ✅:
- build ✅
- test ✅
- publish ✅
- deploy ✅

**5.2: Check the platform**

A new deployment starts just after the CircleCI deploy job. Open its logs - the image is **pulled**, not built.

➡️ **Render:** [render.md - Step B4](./render.md#step-b4-check-the-deployment-on-render) · **Railway:** [railway.md - Step B5](./railway.md#step-b5-check-the-deployment-on-railway)

**5.3: Test Your Application**

Once the new deployment is live:

```bash
curl https://YOUR-SERVICE-URL/hello
```

**Expected response:**
```
DevOps demo application is running!
```

**Success! You have full CI/CD automation!** 🎉

---

## Part 4 - Testing the Full Pipeline

Let's make a code change to see the full automation in action!

### Step 1: Make a Code Change

**Edit your controller:**

Open `src/main/java/com/example/devopsdemo/controller/DemoController.java`

Change the message:

```java
@GetMapping("/hello")
public String hello() {
    return "DevOps demo - AUTOMATED DEPLOYMENT WORKS! 🚀";
}
```

---

### Step 2: Push the Change

```bash
git add .
git commit -m "Update hello message to test CD pipeline"
git push origin main
```

---

### Step 3: Watch the Automation

**In CircleCI:**
1. Pipeline triggers automatically
2. build → test → publish → deploy
3. All jobs complete successfully

**On your platform:**
1. Receives the redeploy request from the CircleCI deploy job
2. Pulls the new image from Docker Hub
3. Replaces the running container with the new version

**Timeline:** ~7-10 minutes total

---

### Step 4: Verify Change is Live

```bash
curl https://YOUR-SERVICE-URL/hello
```

**You should see the NEW message:**
```
DevOps demo - AUTOMATED DEPLOYMENT WORKS! 🚀
```

**🎉 Congratulations!** You just deployed to production by pushing code!

---

## Part 5 - Understanding What You Built

### The Complete CI/CD Pipeline

**1. Code Push (You)**
```
Developer: git push origin main
↓
GitHub: Code received, webhook sent to CircleCI
```

**2. Continuous Integration (CircleCI)**
```
Build Job:
  - Clone code from GitHub
  - Compile Java code with Maven
  - Create JAR file
  - Cache dependencies for faster future builds
  
Test Job:
  - Run all unit tests
  - Ensure code quality
  - If tests fail → STOP (don't deploy broken code!)
  
Publish Job:
  - Build Docker image
  - Push to Docker Hub
  - Image ready for deployment
```

**3. Continuous Deployment (CircleCI → Cloud platform)**
```
Deploy Job:
  - Read the deploy secret from CircleCI environment variables
  - Ask the platform to redeploy the devops-demo service

Cloud platform:
  - Pull the new image from Docker Hub (no build - CI already built it)
  - Stop old version
  - Start new version
  - Route traffic to new version
```

**4. Production (Cloud platform)**
```
Users access: https://YOUR-SERVICE-URL
They see your latest changes!
```

**Total time:** ~7-10 minutes from code push to production!

---

### Professional DevOps Practices You're Using

**1. Automated Testing** — Code is tested before deployment, broken code never reaches production

**2. Containerization** — Application runs the same everywhere, "works on my machine" problem solved

**3. Continuous Deployment** — Fast feedback loop, small frequent changes, easy to rollback

**4. Infrastructure as Code** — `.circleci/config.yml` defines your pipeline, version controlled and reproducible

**5. Environment Variables** — Credentials secure, different configs for different environments

---

## Troubleshooting Guide

| Problem | Likely cause | What to check |
|---|---|---|
| Deploy job fails with "Unauthorized" / `401` | The deploy secret in CircleCI is wrong, missing or was regenerated | Copy it again from the platform; the CircleCI variable name matches exactly; no extra spaces |
| Deploy job fails with "not found" / `404` | It points at a service that doesn't exist (e.g. deleted and recreated) | Copy the credential / ID of your **current** service |
| Platform can't pull the image | Wrong image name, or the repository is private | Image name matches Docker Hub exactly; repository is **Public**; `publish` pushed `latest` |
| Deploy succeeds but the old version is still running | `deploy` ran before `publish` finished, or `latest` wasn't updated | Workflow has `requires: - publish`; `latest` on Docker Hub was updated; a new deployment started on the platform |
| URL doesn't respond / `/hello` returns 404 | Wrong port, or the app didn't start | Logs show "Started DevopsDemoApplication"; traffic goes to port `8080` (Render: free service may be asleep - wait a minute) |
| Platform deploys on every push, even when tests fail | The service is connected to your **GitHub repository**, not the Docker image | Use a service created from the Docker image - only CircleCI should trigger deployments |
| CircleCI pipeline doesn't trigger | Webhook or branch problem | GitHub repo → Settings → Webhooks has CircleCI; you pushed to `main`; trigger the pipeline manually to test |

Your platform guide has the exact fixes: [render.md - Troubleshooting](./render.md#troubleshooting) · [railway.md - Troubleshooting](./railway.md#troubleshooting)

---

## Summary

### What You Accomplished Today

1. ✅ Deployed your Docker Hub image to a cloud platform
2. ✅ Created complete CI/CD pipeline (build → test → publish → deploy)
3. ✅ Automated deployment with a secret deploy credential stored in CircleCI
4. ✅ Tested full pipeline with code changes
5. ✅ Learned professional DevOps practices

---

## Additional Resources

- [What is Continuous Deployment? — Atlassian](https://www.atlassian.com/continuous-delivery/continuous-deployment)
- [CD Best Practices — Google Cloud](https://cloud.google.com/architecture/devops/devops-tech-continuous-delivery)
- [CI/CD Explained — Red Hat](https://www.redhat.com/en/topics/devops/what-is-ci-cd)
- [CircleCI Deployment Overview](https://circleci.com/docs/deployment-overview/)
- [CircleCI Environment Variables](https://circleci.com/docs/env-vars/)
- [CircleCI Status](https://status.circleci.com/)
- Platform docs: see the **Resources** section of [render.md](./render.md#resources) or [railway.md](./railway.md#resources)
