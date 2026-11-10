# Assignment (Optional)

## Brief

Implement a complete CI/CD pipeline with automated deployment to a cloud platform (Render or Railway) and Discord notifications for your Spring Boot application.

Use the same platform as in class, and follow its guide for the clicks and the exact `deploy` job: [render.md](./render.md) or [railway.md](./railway.md).

1. **Run Your Docker Hub Image on the Platform**
   - Make sure your devops-demo CircleCI pipeline is green (build → test → publish) and `YOUR_DOCKERHUB_USERNAME/devops-demo:latest` is in a **public** Docker Hub repository
   - Create a service on the platform from the **Docker image** (not from your GitHub repository), with traffic going to port `8080`
   - Give the service a public URL
   - Test your application: `https://YOUR-SERVICE-URL/hello`
   - Take a screenshot showing your application running on the platform

2. **Configure Automated Deployment with CircleCI**
   - Get your platform's deploy credentials:
     - **Render:** an API key and the service ID (`srv-...`)
     - **Railway:** a project token and the service ID
   - Add them to CircleCI as project environment variables (`RENDER_API_KEY` and `RENDER_SERVICE_ID`, or `RAILWAY_TOKEN` and `RAILWAY_SERVICE_ID`) - never in `config.yml`
   - Add a `deploy` job to `.circleci/config.yml` that uses the platform's CLI to redeploy the service (Render: `render deploys create --image <your image> --wait`)
   - Update your workflow so `deploy` runs only after `publish` succeeds

3. **Notify Discord**
   - Create a Discord server and a webhook, and add the webhook URL to CircleCI as `DISCORD_WEBHOOK_URL`
   - Add a `notify` job that posts the deployment result to Discord
   - Add it to the workflow twice: `notify-success` (requires `deploy: success`) and `notify-failure` (requires `deploy: failed`)
   - Commit and push your updated config.yml
   - Watch the complete pipeline run: build → test → publish → deploy → notify-success
   - Verify all five jobs pass in CircleCI (`notify-failure` is not run), that a new deployment started on the platform, and that a ✅ message appeared in Discord

4. **Test the Full CI/CD Pipeline**
   - Make a small change to your DemoController (update the /hello message)
   - Update your test to match the new message
   - Commit and push the change
   - Watch the automated pipeline:
     - CircleCI builds, tests and publishes the Docker image
     - CircleCI triggers the deployment
     - The platform pulls the new image and redeploys
   - Verify the change appears on your live URL
   - Test the failure path: set a wrong service ID in CircleCI, rerun the workflow, and check that `notify-failure` posts ❌ to Discord; then put the correct ID back
   - Take screenshots showing:
     - CircleCI pipeline with all 5 jobs passing
     - The platform's deployment logs
     - Your updated message live on the platform
     - The ✅ and ❌ messages in Discord
   - Write a brief report (4-5 sentences) explaining:
     - The complete flow from code push to production
     - How long the deployment took
     - What would happen if your tests failed, and what would happen if the deployment failed
     - The benefits of automated deployment vs manual deployment

## Submission (Optional)

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.

## References
- Java: https://docs.oracle.com/javase/
- Spring Boot: https://docs.spring.io/spring-boot/docs/current/reference/html/
- Render CLI: https://render.com/docs/cli
- Discord webhooks: https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks
- Railway CLI: https://docs.railway.com/cli/deploying
