# Assessment / Quiz

## Questions

### Q1 (True/False)

Q1: Continuous Deployment automatically deploys every code change that passes tests to production without manual intervention?

A - True

B - False

---

Q2: What is the main difference between Continuous Delivery and Continuous Deployment?

A - Continuous Delivery deploys faster than Continuous Deployment

B - Continuous Delivery requires manual approval before production deployment, Continuous Deployment is fully automated

C - They are the same thing with different names

D - Continuous Delivery only works with Docker

---

Q3: In a professional CI/CD pipeline, deploying small, frequent changes is safer than deploying large, infrequent changes?

A - True

B - False

---

Q4: In this lesson's pipeline, what does the deploy job do?

A - Builds the Docker image again from the source code

B - Asks the cloud platform to redeploy the service, which then pulls the latest image from Docker Hub

C - Uploads the source code to the cloud platform

D - Runs the tests again on the cloud platform

---

Q5: Why do we create the service from the Docker image instead of connecting the platform to our GitHub repository?

A - Deploying from GitHub is not supported by Render or Railway

B - So that only CircleCI decides when to deploy - after the tests have passed

C - Docker images are always smaller than source code

D - So that we don't need a Dockerfile

---

Q6: Render and Railway automatically deploy a new `latest` image as soon as it is pushed to Docker Hub, so the deploy job is optional?

A - True

B - False

---

Q7: In the complete CI/CD pipeline, what happens if the test job fails?

A - The pipeline continues to publish and deploy anyway

B - Only a warning is shown but deployment proceeds

C - The pipeline stops and publish/deploy jobs do not run

D - Tests are automatically retried until they pass

---

Q8: The deploy job in CircleCI should run AFTER the publish job completes successfully?

A - True

B - False

---

Q9: What is the primary benefit of storing deploy credentials (such as a Railway token or a Render deploy hook URL) in CircleCI environment variables?

A - Faster deployment

B - Better performance

C - Security - credentials are encrypted and not exposed in code

D - Easier to remember

---

Q10: When you push code to GitHub, a complete CI/CD pipeline with deployment to a cloud platform typically takes how long from push to production?

A - 30-60 seconds

B - 5-10 minutes

C - 10-15 minutes

D - 30-45 minutes

## Notes
- Keep each question atomic and aligned to the lesson objectives.
- Put the answer immediately after each question.