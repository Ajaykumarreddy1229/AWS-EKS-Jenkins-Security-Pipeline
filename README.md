# AWS-EKS-Jenkins-Security-Pipeline
# Deploying the 2048 Game on Docker and Kubernetes (AWS EKS) with Jenkins CI/CD

## GitHub Repository

https://github.com/Ajaykumarreddy1229/devsecops-2048_game-.git

Clone the repository:

```bash
git clone https://github.com/Ajaykumarreddy1229/devsecops-2048_game-.git
cd devsecops-2048_game-
```

## Project Overview

This project demonstrates an end-to-end CI/CD and DevSecOps workflow for
the 2048 game application. Jenkins is used to automate source-code
checkout, code-quality analysis, dependency installation, security
scanning, Docker image creation, image publishing, and deployment.

Terraform is used to provision the AWS EKS infrastructure. The
application is packaged as a Docker image, pushed to Docker Hub, and
deployed to Kubernetes.

## Objectives

-   Automate AWS EKS infrastructure provisioning with Terraform and
    Jenkins.
-   Build a Jenkins pipeline for the 2048 application.
-   Integrate SonarQube for static code analysis and a quality-gate
    check.
-   Scan project files and container images with Trivy.
-   Configure OWASP Dependency-Check in Jenkins.
-   Build and publish a Docker image to Docker Hub.
-   Deploy the application to an AWS EKS cluster.

## Architecture / Workflow

The architecture below highlights the components covered by my project notes.
<img width="1536" height="1024" alt="2048-Project" src="https://github.com/user-attachments/assets/e66d0b79-70a3-42d3-a642-602cbba686a5" />

**Highlight guide:**

- **Green:** Covered in my project notes — GitHub, Jenkins, SonarQube code analysis and quality gate, Docker build and Docker Hub, Trivy filesystem/image scanning, Terraform, AWS, and EKS deployment.
- **Orange:** Partially covered — the notes include dependency installation, Docker build, and security scanning; however, the sample Jenkins pipeline does not show a separate unit-test stage.
- **Grey:** Not shown as implemented in my notes — OWASP Dependency-Check execution in the pipeline, Prometheus, Grafana, Loki, Alertmanager, Jaeger, and Slack/Email/Teams notifications. The notes mention installing the OWASP Dependency-Check plugin, but do not include its pipeline stage.

### Workflow summary

1. Developer pushes application or Terraform code to GitHub.
2. GitHub triggers Jenkins through a webhook.
3. Jenkins checks out the source and runs SonarQube code analysis and the quality-gate check.
4. Jenkins installs Node.js dependencies and runs a Trivy filesystem scan.
5. Jenkins builds the Docker image, tags it, and pushes it to Docker Hub.
6. Trivy scans the Docker image.
7. Terraform provisions the AWS EKS infrastructure, using an S3 backend for state.
8. Jenkins uses `kubectl` to apply the Kubernetes deployment manifest to EKS.

## Technologies Used

  Technology               Purpose
  ------------------------ -------------------------------------------------------
  AWS EC2                  Host the Jenkins and supporting tools
  AWS IAM                  Provide AWS permissions to the automation environment
  AWS S3                   Store Terraform remote state
  AWS EKS                  Managed Kubernetes cluster
  Terraform                Provision cloud infrastructure
  Jenkins                  Automate CI/CD pipelines
  GitHub                   Store application and infrastructure source code
  Java                     Run Jenkins
  Docker                   Build and run containers
  Docker Hub               Store and distribute the Docker image
  SonarQube                Static code analysis and quality gate
  Trivy                    Scan filesystems and container images
  OWASP Dependency-Check   Dependency vulnerability checking
  Node.js and npm          Install application dependencies
  kubectl                  Manage Kubernetes resources
  AWS CLI                  Interact with AWS services

## Implementation Steps

### 1. Prepare the AWS EC2 instance

-   Launch an Ubuntu 24 instance.
-   Configure the required storage.
-   Attach an IAM role with the permissions needed for the lab.
-   Connect to the instance and update the operating system.

**Note:** The source notes mention both `m7i-flex.large` and `t2.large`
in the instance description. Confirm the intended instance type before
deployment.

### 2. Install the required tools

The setup scripts in the notes are:

-   `script1.sh` --- installs Java 21, downloads and starts Jenkins as a
    WAR file, and installs Docker.
-   `script2.sh` --- installs Terraform, kubectl, and AWS CLI.
-   `trivy.sh` --- installs Trivy.

The notes also start a SonarQube LTS Community container on port `9000`.

### 3. Configure Jenkins

Install and configure the required Jenkins plugins and tools:

-   Eclipse Temurin installer
-   SonarQube Scanner
-   NodeJS
-   OWASP Dependency-Check
-   Docker-related plugins
-   Kubernetes-related plugins

Configure the JDK, Node.js, SonarQube Scanner, Dependency-Check, and
Docker installations under **Manage Jenkins → Tools**.

### 4. Provision the EKS cluster with Terraform

Create a Jenkins pipeline job for EKS provisioning.

The pipeline stages described in the notes are:

1.  Checkout the Terraform source from Git.
2.  Check the Terraform version.
3.  Run `terraform init`.
4.  Run `terraform validate`.
5.  Run `terraform plan`.
6.  Run the selected Terraform action (`apply` or `destroy`).

The Terraform backend is configured to use an S3 bucket. Update the
backend bucket setting to match the bucket created for your own
environment.

### 5. Configure SonarQube

-   Generate a SonarQube authentication token.
-   Store it in Jenkins Credentials as a Secret Text credential.
-   Configure the SonarQube server under Jenkins system settings.
-   Configure the SonarQube webhook to communicate with Jenkins.
-   Use the SonarQube Scanner in the application pipeline.
-   Run the quality-gate check.

### 6. Build and scan the application

The application pipeline described in the notes performs these
operations:

1.  Clean the Jenkins workspace.
2.  Checkout the 2048 application source code.
3.  Run SonarQube static code analysis.
4.  Wait for the quality-gate result.
5.  Install application dependencies using `npm install`.
6.  Run a Trivy filesystem scan.
7.  Build the Docker image.
8.  Tag and push the image to Docker Hub.
9.  Run a Trivy image scan.

The notes also describe installing the OWASP Dependency-Check plugin.
Add and configure an explicit Dependency-Check pipeline stage if you
want the pipeline to execute that scan; the sample pipeline shown in the
notes does not include that stage.

### 7. Deploy to AWS EKS

-   Update the local kubeconfig using `aws eks update-kubeconfig`.
-   Verify cluster access with `kubectl get nodes`.
-   Configure Kubernetes credentials in Jenkins.
-   Use the Jenkins Kubernetes integration to apply `deployment.yaml`.
-   Verify the deployment using `kubectl get pods` and
    `kubectl get service`.

The notes also include a Docker container run stage on port `3000`. This
is a separate container-based run on the Jenkins host; the Kubernetes
deployment is performed by applying the Kubernetes manifest to EKS.

## Important Configuration to Customize

Before using the sample scripts or pipelines, replace
environment-specific values with your own:

-   Git repository URL and branch.
-   Terraform S3 backend bucket name.
-   AWS region and EKS cluster name.
-   Docker Hub username `ajay1229` and image name.
-   Jenkins tool names and credential IDs.
-   Kubernetes deployment and service manifest settings.

Do not copy another person's account identifiers, public IP addresses,
credentials, or repository-specific values into your own project.

## Security Notes

-   **Never commit tokens, passwords, private keys, kubeconfig files, or
    other secrets to GitHub.** A token was present in the original
    working notes. If it was real or active, revoke it and generate a
    replacement immediately.
-   Use Jenkins Credentials for tokens and Docker Hub credentials.
-   Prefer a least-privilege IAM role instead of broad administrator
    access.
-   Avoid `chmod 777 /var/run/docker.sock`. Access to the Docker socket
    can provide root-level control over the host. Use a safer,
    restricted access model.
-   Restrict Jenkins, SonarQube, and application ports to trusted IP
    ranges.
-   Review Trivy and dependency-scan findings and resolve
    vulnerabilities before release.
-   Protect the Terraform state bucket and enable appropriate access
    controls.

## Validation Commands

``` bash
# Check installed tools
java -version
docker --version
terraform --version
kubectl version --client
aws --version
trivy --version

# Check AWS identity and EKS access
aws sts get-caller-identity
aws eks list-clusters --region <AWS_REGION>
kubectl get nodes

# Check Kubernetes resources
kubectl get pods
kubectl get services
kubectl get all
```

Replace `<AWS_REGION>` with the region used by your cluster.

## Expected Outcome

After successful execution:

-   Terraform provisions the EKS infrastructure.
-   Jenkins runs the application CI/CD pipeline.
-   SonarQube performs code analysis and reports the quality-gate
    result.
-   Trivy scans the project files and Docker image.
-   Docker publishes the application image to Docker Hub.
-   Jenkins deploys the application manifest to AWS EKS.
-   Kubernetes resources can be checked using `kubectl`.

## Learning Outcomes

Through this project, I practised:

-   Jenkins pipeline creation and configuration.
-   Terraform-based AWS infrastructure provisioning.
-   Docker image building and publishing.
-   SonarQube integration with Jenkins.
-   Filesystem and container image scanning with Trivy.
-   AWS EKS and Kubernetes deployment.
-   Managing CI/CD credentials and deployment configuration.

## Conclusion

This project brings together CI/CD automation, Infrastructure as Code,
containerization, security scanning, and Kubernetes deployment. It
provides hands-on practice with a DevSecOps workflow using Jenkins and
AWS EKS.
