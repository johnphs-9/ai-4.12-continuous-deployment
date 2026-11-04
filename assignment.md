# Assignment (Optional)

## Brief

Implement a complete CI/CD pipeline with automated deployment to a cloud platform (Render or Railway) for your Spring Boot application.

Use the same platform as in class, and follow its guide for the clicks and the exact `deploy` job: [render.md](./render.md) or [railway.md](./railway.md).

1. **Run Your Docker Hub Image on the Platform**
   - Make sure your devops-demo CircleCI pipeline is green (build → test → publish) and `YOUR_DOCKERHUB_USERNAME/devops-demo:latest` is in a **public** Docker Hub repository
   - Create a service on the platform from the **Docker image** (not from your GitHub repository), with traffic going to port `8080`
   - Give the service a public URL
   - Test your application: `https://YOUR-SERVICE-URL/hello`
   - Take a screenshot showing your application running on the platform

2. **Configure Automated Deployment with CircleCI**
   - Get your platform's deploy credential:
     - **Render:** the service's deploy hook URL
     - **Railway:** a project token and the service ID
   - Add it to CircleCI as project environment variables (`RENDER_DEPLOY_HOOK_URL`, or `RAILWAY_TOKEN` and `RAILWAY_SERVICE_ID`) - never in `config.yml`
   - Add a `deploy` job to `.circleci/config.yml` that asks the platform to redeploy the service
   - Update your workflow so `deploy` runs only after `publish` succeeds
   - Commit and push your updated config.yml
   - Watch the complete pipeline run: build → test → publish → deploy
   - Verify all four jobs pass in CircleCI, and that a new deployment started on the platform

3. **Test the Full CI/CD Pipeline**
   - Make a small change to your DemoController (update the /hello message)
   - Update your test to match the new message
   - Commit and push the change
   - Watch the automated pipeline:
     - CircleCI builds, tests and publishes the Docker image
     - CircleCI triggers the deployment
     - The platform pulls the new image and redeploys
   - Verify the change appears on your live URL
   - Take screenshots showing:
     - CircleCI pipeline with all 4 jobs passing
     - The platform's deployment logs
     - Your updated message live on the platform
   - Write a brief report (4-5 sentences) explaining:
     - The complete flow from code push to production
     - How long the deployment took
     - What would happen if your tests failed
     - The benefits of automated deployment vs manual deployment

## Submission (Optional)

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.

## References
- Java: https://docs.oracle.com/javase/
- Spring Boot: https://docs.spring.io/spring-boot/docs/current/reference/html/
- Render deploy hooks: https://render.com/docs/deploy-hooks
- Railway CLI: https://docs.railway.com/cli/deploying
