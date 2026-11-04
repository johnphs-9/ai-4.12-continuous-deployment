# 4.12 Self Studies

**Estimated Preparation Time:** 65 minutes

---

> **📌 Note:** The videos in these self studies are selected to help you build familiarity with the tools and concepts before class. The approach, project structure, and commands used in the videos may differ from what we do in the lesson. Focus on understanding the **concepts and the why** — not on replicating the video step by step. The lesson content is what you should follow during class.

---

## Task 1 — CI, Continuous Delivery, and Continuous Deployment (25 minutes)

Watch the following video on the differences between CI, Continuous Delivery, and Continuous Deployment:

- 📹 [Difference between CI, Continuous Delivery and Continuous Deployment — https://www.youtube.com/watch?v=wbnOmXWyd9E]

While watching, refer to **lesson.md Part 1** and pay attention to:
- How Continuous Integration, Continuous Delivery, and Continuous Deployment relate to each other
- The key difference between Continuous Delivery and Continuous Deployment
- Why some teams choose manual approval before deploying to production
- What conditions need to be in place before a team can safely adopt Continuous Deployment

**Guiding Questions:**
1. In your own words, what is the difference between Continuous Delivery and Continuous Deployment?
2. Why would a banking application prefer Continuous Delivery over Continuous Deployment?
3. What role does automated testing play in making Continuous Deployment safe?

---

## Task 2 — CD Pipeline and Deployment Environments (20 minutes)

No video for this task. Refer to **lesson.md Part 1** (CD Pipeline and Deployment Environments sections) and answer the following:

1. List the four stages of a typical CD pipeline and explain what happens in each stage in your own words
2. Why do professional teams use multiple environments (development, staging, production, DR)? What problem does this solve?
3. What is the purpose of a Disaster Recovery (DR) environment and when would it be activated?
4. How does Continuous Deployment enable A/B testing more effectively than manual deployment?

**Guiding Questions:**
1. What is the difference between a staging environment and a production environment?
2. Why is it important to run health checks after deploying a new version?
3. What does "Done means Released" mean as a CD principle?

---

## Task 3 — Prepare for the Hands-On Lesson (20 minutes)

No video for this task. Complete the following before class to save time during the lesson:

1. Verify your CircleCI pipeline for devops-demo is working — go to [https://app.circleci.com](https://app.circleci.com) and confirm build ✅ → test ✅ → publish ✅
2. Confirm your Docker Hub has a `devops-demo:latest` image published
3. Make sure you can log in to the cloud platform you used in Lesson 4.10 — [Render](https://render.com) or [Railway](https://railway.app). If you don't have an account yet, sign up with GitHub
4. Have your Docker Hub username and password ready

Coming to class prepared will allow you to focus on the CD automation rather than setup.

**Guiding Questions:**
1. What is a deploy credential (for example, a Render deploy hook URL or a Railway project token) and why does CircleCI need one for automated deployment?
2. Why should credentials like `RENDER_DEPLOY_HOOK_URL`, `RAILWAY_TOKEN` and `DOCKER_TOKEN` never be hardcoded in config files?
3. In the CD pipeline we are building — build → test → publish → deploy — what happens if the test job fails?

---

## Active Engagement Strategies

- Pause the video when each concept (CI, Continuous Delivery, Continuous Deployment) is introduced and try to give your own example before the video does
- After watching, close the video and try to draw the flow from code push to production for both Continuous Delivery and Continuous Deployment
- For Task 3, complete the account and pipeline checks before class — do not leave them for the lesson

---

## Additional Reading Material

- [What is Continuous Deployment? — Atlassian](https://www.atlassian.com/continuous-delivery/continuous-deployment)
- [CI/CD Explained — Red Hat](https://www.redhat.com/en/topics/devops/what-is-ci-cd)
- [Render - Deploy Hooks](https://render.com/docs/deploy-hooks)
- [Railway Documentation](https://docs.railway.app/)
- [Railway CLI Reference](https://docs.railway.app/develop/cli)