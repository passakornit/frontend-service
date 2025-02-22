# CI/CD Pipeline - Pipepline

This repository contains a GitHub Actions workflow for a comprehensive CI/CD pipeline designed to build, scan, and deploy Docker images to Amazon ECR. The pipeline is configured to run on pull requests targeting the `main` branch, as well as via manual triggers using workflow dispatch.

## Overview

The pipeline performs the following actions:
- **Code Checkout:** Retrieves the latest code from the repository.
- **Docker Setup:** Configures Docker Buildx for advanced image building.
- **AWS Configuration:** Sets up AWS credentials to interact with AWS services.
- **ECR Login:** Authenticates with Amazon ECR to allow image pushes.
- **Image Building:** Builds a Docker image tagged with the repository name and a unique run number.
- **Vulnerability Scanning:** Runs a security scan using Trivy, outputting results in SARIF format.
- **Security Reporting:** Uploads scan results to GitHub’s Security tab.
- **Image Deployment:** Pushes the Docker image to the designated ECR repository.
- **Notifications:** Sends notifications to Google Chat for both successful and failed pipeline runs.

## Trigger Events

- **Pull Request:** Automatically triggered when a pull request is made to the `main` branch.
- **Workflow Dispatch:** Can be manually triggered via the GitHub Actions interface.

## Detailed Workflow Steps

1. **Checkout Code**
   - Uses `actions/checkout@v3` to pull the repository code.

2. **Set Up Docker**
   - Uses `docker/setup-buildx-action@v2` to configure Docker Buildx, enabling multi-platform builds and advanced features.

3. **Configure AWS Credentials**
   - Uses `aws-actions/configure-aws-credentials@v1` to set up AWS credentials using secrets:
     - `AWS_ACCESS_KEY_ID`
     - `AWS_SECRET_ACCESS_KEY`
     - `AWS_REGION`

4. **Login to Amazon ECR**
   - Uses `aws-actions/amazon-ecr-login@v2` to authenticate with Amazon ECR, allowing subsequent image push operations.

5. **Set Image Version**
   - Sets an environment variable `IMAGE_VERSION` based on the GitHub run number to uniquely tag the Docker image.

6. **Build Docker Image**
   - Builds the Docker image with a tag composed of the ECR registry, repository, repository name, and the image version.
   - Example tag format: `ECR_REGISTRY/ECR_REPOSITORY:<IMAGE_VERSION>`

7. **Run Trivy Vulnerability Scanner**
   - Uses `aquasecurity/trivy-action@0.28.0` to scan the built Docker image for vulnerabilities.
   - Output is formatted in SARIF and saved as `trivy-results.sarif`.
   - The step is configured to fail the build if vulnerabilities 'CRITICAL' are detected (exit code `1`).

8. **Upload Trivy Scan Results**
   - Uses `github/codeql-action/upload-sarif@v3` to upload the SARIF file to GitHub’s Security tab.
   - This step is marked to continue on error to ensure that even if uploading fails, the workflow can still report the failure status.

9. **Push Docker Image to ECR**
   - If all prior steps succeed, the Docker image is pushed to the Amazon ECR repository.

10. **Google Chat Notifications**
    - **Success Notification:** When the pipeline succeeds, a message is sent to Google Chat containing details such as repository name, branch, tag, Docker image reference, and a link to the workflow run.
    - **Failure Notification:** If any step fails, a similar notification is sent to alert the team about the failure.

## Required Secrets

Ensure the following secrets are set in your GitHub repository:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`
- `ECR_REGISTRY`
- `ECR_REPOSITORY`
- `GOOGLE_CHAT_WEBHOOK`

## Customization Options

- **Image Tagging:** The Docker image tag uses the repository name and the GitHub run number. Modify this if a different tagging scheme is needed.
- **Notification Messages:** Edit the messages in the Google Chat notification steps to match your communication style or include additional details.
- **Security Scanning:** Adjust the Trivy scan configuration and exit code handling based on your organization’s security policies.

## Running the Pipeline

- **Automatic Execution:** Open a pull request targeting the `main` branch to trigger the pipeline automatically.
- **Manual Execution:** Use the "Run workflow" button in the Actions tab to start the pipeline manually (workflow dispatch).

## Conclusion

This CI/CD pipeline provides an automated workflow to build, scan, and deploy Docker images to Amazon ECR while integrating security scanning and real-time notifications. It is designed to ensure that only secure and validated images are deployed, improving the overall security posture of your deployment process.