# Pipeline for OWASP Dependency-Check and Docker Deployment

This Jenkins pipeline script automates OWASP Dependency-Check vulnerability scanning and Docker image building and deployment, with notifications to Discord on success or failure.

## Prerequisites

Ensure the following are set up before running this pipeline:
- Jenkins with necessary plugins (OWASP Dependency-Check, Docker, SSH)
- Docker Hub credentials configured in Jenkins
- Discord webhook URL for notifications

## Pipeline Overview

The pipeline consists of three main stages:
1. **Checkout**: Fetches the source code from the repository.
2. **OWASP Dependency-Check Vulnerabilities**: Runs OWASP Dependency-Check to scan for vulnerabilities.
3. **Build and Push Docker Image**: Builds a Docker image and pushes it to Docker Hub.
4. **Trigger Deployment**: Deploys the Docker image to a remote server via SSH.

## Environment Variables

Make sure these environment variables are configured in your Jenkins job:
- `DOCKER_CREDENTIALS_ID`: Jenkins credentials ID for Docker Hub.
- `IMAGE`: Docker image name and tag.
- `NVD_API_KEY`: API key for NVD (National Vulnerability Database).
- `DISCORD_WEBHOOK_URL`: Discord webhook URL for notifications.

## Usage

1. **Configure Jenkins**:
   - Create a new pipeline job in Jenkins.
   - Copy the pipeline script (`Jenkinsfile`) into the job configuration.
   
2. **Run the Pipeline**:
   - Trigger the pipeline manually or set up automatic triggers based on your repository changes.

3. **Monitor and Notifications**:
   - Jenkins will notify Discord with build and deployment status updates.

## Discord Notifications

Notifications are sent to Discord using a webhook. Customize the Discord webhook URL and icon URL in the `sendDiscordNotification` function within the pipeline script.

## Contributing

Feel free to fork this repository and enhance the pipeline script based on your needs. Pull requests are welcome!