# Flask AWS CI/CD Local

This project is a simple Flask web application created as a hands-on learning project for Docker, AWS, CI/CD, and Kubernetes.

Instead of using real AWS resources during development, I use **FLoCI as a local AWS simulator**. This allows me to practice AWS workflows locally without creating or managing real AWS resources.

The project also uses **GitHub Actions** for CI/CD and **Minikube** for running the application on a local Kubernetes cluster.

## Technologies Used

- Python
- Flask
- Docker
- AWS ECR
- FLoCI
- Git & GitHub
- GitHub Actions
- Kubernetes
- Minikube

## Project Structure

```text
flask-aws-ci-cd-local/
│
├── app.py
├── Dockerfile
├── requirements.txt
├── README.md
│
├── k8s/
│   ├── deployment.yml
│   └── service.yml
│
└── .github/
    └── workflows/
        └── ci.yml
```

## Application

The application is a basic Flask web application running on port `5000`.

The current application displays:

```text
Hello, World
```

The application is intentionally simple because the primary goal of this project is to understand the complete workflow from application development to containerization, CI/CD, image storage, and Kubernetes deployment.

## FLoCI Local AWS Environment

I use **FLoCI** to simulate AWS services locally.

FLoCI runs through Docker and provides a local AWS-compatible endpoint:

```text
http://localhost:4566
```

The AWS CLI is configured to communicate with FLoCI instead of the real AWS environment.

```bash
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
export AWS_ENDPOINT_URL=http://localhost:4566
```

FLoCI is started using Docker Compose:

```bash
cd ~/floci
docker compose up -d
```

## Creating an ECR Repository

As part of the learning process, I created an ECR repository named `flask-app` in FLoCI.

```bash
aws ecr create-repository \
  --repository-name flask-app \
  --endpoint-url http://localhost:4566 \
  --region us-east-1
```

The repository is:

```text
flask-app
```

FLoCI provides a local Docker registry for the repository. I verified the registry using:

```bash
curl http://localhost:5100/v2/
```

The repository tags can be checked using:

```bash
curl http://localhost:5100/v2/flask-app/tags/list
```

## Building and Pushing the Docker Image

The Flask application is packaged as a Docker image using the project's `Dockerfile`.

Build the image:

```bash
docker build -t flask-app:latest .
```

Tag the image for the FLoCI ECR registry:

```bash
docker tag flask-app:latest \
000000000000.dkr.ecr.us-east-1.localhost:5100/flask-app:latest
```

Login to the local ECR registry:

```bash
aws ecr get-login-password \
  --endpoint-url http://localhost:4566 \
  --region us-east-1 \
| docker login \
  --username AWS \
  --password-stdin \
  000000000000.dkr.ecr.us-east-1.localhost:5100
```

Push the image:

```bash
docker push \
000000000000.dkr.ecr.us-east-1.localhost:5100/flask-app:latest
```

This workflow provides hands-on practice with the Docker image lifecycle and the basic concepts behind using AWS ECR.

## GitHub Actions CI/CD

The project uses **GitHub Actions** to automate the CI/CD process.

The workflow is triggered when changes are pushed to the `main` branch.

The workflow runs on a local **self-hosted GitHub Actions runner** and performs the following steps:

1. Checks out the source code.
2. Checks the Python environment.
3. Creates a Python virtual environment.
4. Installs the application dependencies.
5. Tests the Flask application.
6. Builds the Docker image.
7. Logs in to the FLoCI ECR registry.
8. Tags the Docker image.
9. Pushes the image to FLoCI ECR.
10. Starts and verifies the Minikube cluster.
11. Loads the image into Minikube.
12. Applies the Kubernetes configuration.
13. Updates the Kubernetes deployment with the new image.
14. Waits for the deployment rollout to complete.
15. Displays the final Kubernetes resources.

The overall workflow is:

```text
GitHub
   │
   │ Push to main
   ▼
GitHub Actions
   │
   ▼
Self-hosted Runner
   │
   ├── Test Flask Application
   │
   ├── Build Docker Image
   │
   ├── Push Image to FLoCI ECR
   │
   └── Deploy to Minikube
           │
           ▼
      Kubernetes
           │
           ▼
     Flask Application
```

## Kubernetes Deployment

The application is deployed to Kubernetes using **Minikube**.

Start Minikube:

```bash
minikube start --driver=docker
```

Check the Kubernetes cluster:

```bash
kubectl get nodes
```

The Kubernetes deployment runs two replicas of the Flask application.

```yaml
replicas: 2
```

The application is exposed using a Kubernetes `NodePort` service.

Check the Kubernetes resources:

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

Open the application using:

```bash
minikube service flask-app --url
```

The application can then be accessed in a browser and displays:

```text
Hello, World
```

## Why FLoCI?

I wanted to practice AWS concepts alongside the theory I was learning from AWS courses and tutorials.

Using FLoCI gives me a local environment where I can experiment with AWS-style workflows without relying on real AWS infrastructure for every learning exercise.

For this project, I mainly use FLoCI for practicing the **ECR and container image workflow**.

I plan to explore additional AWS and DevOps topics as I continue learning:

- IAM
- EC2
- VPC
- S3
- CloudWatch
- ECR
- Route 53
- Application Load Balancer (ALB)
- ECS
- EKS
- Terraform
- Infrastructure as Code (IaC)

## What This Project Demonstrates

This project brings several technologies together into one local CI/CD workflow:

- **GitHub** — source code management
- **GitHub Actions** — CI/CD automation
- **Self-hosted Runner** — executes the workflow locally
- **Docker** — containerizes the Flask application
- **FLoCI** — provides a local AWS-compatible environment
- **ECR** — stores the Docker image locally through FLoCI
- **Minikube** — provides a local Kubernetes cluster
- **Kubernetes** — deploys and manages the application

The main goal of this project is to gain practical experience with the tools and workflow commonly used in modern **AWS, DevOps, containerization, CI/CD, and Kubernetes environments**.

> **Note:** The AWS credentials used in this project (`test` / `test`) are dummy credentials for the local FLoCI environment and are not real AWS credentials.
