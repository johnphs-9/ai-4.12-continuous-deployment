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
* **Notify** the team on Discord whether each deployment succeeded or failed

## Lesson Plan

| Duration | What | How or Why |
|----------|------|------------|
| 10 min | Warm up | Intro and lesson overview, verify prerequisites (incl. a Discord account) and CircleCI pipeline status |
| 25 min | Part 1: CD concepts | Continuous Delivery vs Deployment, CD principles, benefits, pipeline stages, deployment environments, A/B testing |
| 25 min | Part 2: Run your Docker Hub image on a cloud platform | Check the 4.7 pipeline and public Docker Hub image, create a service from the image on the class platform (Render or Railway, following `render.md` / `railway.md`), get a public URL, verify endpoint |
| 10 min | Activity 1 — Manual deployment discussion | Learners discuss what's still manual in the current setup and what needs to be automated |
| 70 min | Part 3: Automate deployment with CircleCI | Get the platform's credentials (Render API key + service ID, or Railway project token + service ID), add them to CircleCI, add a deploy job that uses the platform CLI (`render deploys create --wait`, or `railway redeploy`), create a Discord webhook and add a notify job (`notify-success` / `notify-failure`), push and watch the full pipeline run |
| 20 min | Part 4: Test the full pipeline | Make a code change, push, watch automation, verify live endpoint and the ✅ Discord message; break the service ID to see `notify-failure` post ❌, then fix it |
| 10 min | Part 5: Understanding what you built | Full CI/CD pipeline recap, professional DevOps practices review |
| 5 min | Recap and wrap up | Key takeaways, troubleshooting tips, Q&A |
| **Total** | | **175 min — allows ~5 min buffer** |