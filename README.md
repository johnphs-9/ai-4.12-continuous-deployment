# [4.12] Continuous Deployment - Automated Deployment to the Cloud

## Lesson Overview
![Title](./assets/images/infographic-4.12-continuous-deployment-railway.png)

## Dependencies
- [Self Studies](./studies.md)
- [Lesson](./lesson.md)
- [Assignment](./assignment.md)
- Deployment guides: [Render](./render.md) · [Railway](./railway.md)

## Lesson Objectives
* **Distinguish** between Continuous Delivery and Continuous Deployment
* **Deploy** a Docker image from Docker Hub to a cloud platform (such as Render or Railway)
* **Configure** CircleCI to automatically deploy applications after successful tests
* **Implement** a complete CI/CD pipeline from code push to production

## Lesson Plan

| Duration | What | How or Why |
|----------|------|------------|
| 10 min | Warm up | Intro and lesson overview, verify prerequisites and CircleCI pipeline status |
| 30 min | Part 1: CD concepts | Continuous Delivery vs Deployment, CD principles, benefits, pipeline stages, deployment environments, A/B testing |
| 30 min | Part 2: Run your Docker Hub image on a cloud platform | Check the 4.7 pipeline and public Docker Hub image, create a service from the image on the class platform (Render or Railway, following `render.md` / `railway.md`), get a public URL, verify endpoint |
| 10 min | Activity 1 — Manual deployment discussion | Learners discuss what's still manual in the current setup and what needs to be automated |
| 75 min | Part 3: Automate deployment with CircleCI | Get the platform's deploy credential (Render deploy hook URL, or Railway project token + service ID), add it to CircleCI, add a deploy job to config.yml (`curl` the hook, or `railway redeploy`), push and watch full pipeline run |
| 20 min | Part 4: Test the full pipeline | Make code change, push, watch automation, verify live endpoint |
| 10 min | Part 5: Understanding what you built | Full CI/CD pipeline recap, professional DevOps practices review |
| 10 min | Recap and wrap up | Key takeaways, troubleshooting tips, Q&A |
| **Total** | | **175 min — allows ~5 min buffer** |