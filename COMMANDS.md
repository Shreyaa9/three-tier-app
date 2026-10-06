# Commands Reference — Three-Tier To-Do Application

## 1. Purpose

This document contains the main commands used during development, local testing, Docker setup, AWS deployment, ECS management, Cloud Map service discovery, ECR operations, GitHub integration, and cleanup of the Three-Tier To-Do Application.

---

# 2. Project Navigation

Navigate to the project directory:

    cd "C:\Users\Shreya Gupta\Desktop\three-tier-app"

Check the project files:

    dir

---

# 3. Git Commands

## Check Repository Status

    git status

## Check Commit History

    git log --oneline

## Pull Latest Changes

    git pull origin main --rebase

## Add Changes

    git add .

## Commit Changes

    git commit -m "Update project"

## Push Changes

    git push origin main

## Check Remote Repository

    git remote -v

---

# 4. Local Docker Commands

## Check Docker Version

    docker --version

## Check Docker Compose Version

    docker compose version

## Build the Application

    docker compose build

## Start the Application

    docker compose up -d

## Check Running Containers

    docker ps

## Check All Containers

    docker ps -a

## View Application Logs

    docker compose logs

## View Frontend Logs

    docker compose logs frontend

## View Backend Logs

    docker compose logs backend

## View MongoDB Logs

    docker compose logs mongodb

## Follow Logs

    docker compose logs -f

## Stop the Application

    docker compose down

## Stop and Remove Local Images

    docker compose down --rmi local

---

# 5. Local Application Testing Commands

## Test Frontend

Open in the browser:

    http://localhost

## Test Backend API

    curl http://localhost:5000/tasks

## Test Frontend API Through Nginx

    curl http://localhost/api/tasks

The expected successful response should contain task data.

---

# 6. Docker Container Inspection

## Inspect a Container

    docker inspect <container_name>

## Check Container Logs

    docker logs <container_name>

## Execute a Command Inside a Container

    docker exec -it <container_name> sh

## Check Docker Networks

    docker network ls

## Inspect Docker Network

    docker network inspect <network_name>

---

# 7. AWS CLI Configuration

## Check AWS CLI Version

    aws --version

## Check Current AWS Identity

    aws sts get-caller-identity

The project uses AWS Region:

    us-east-1

Set the default region if required:

    aws configure

---

# 8. Amazon ECR Commands

## List ECR Repositories

    aws ecr describe-repositories --region us-east-1

## Login to Amazon ECR

    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 714950921352.dkr.ecr.us-east-1.amazonaws.com

---

## Backend Image

Build backend image:

    docker build -t three-tier-backend ./backend

Tag backend image:

    docker tag three-tier-backend:latest 714950921352.dkr.ecr.us-east-1.amazonaws.com/three-tier-backend:latest

Push backend image:

    docker push 714950921352.dkr.ecr.us-east-1.amazonaws.com/three-tier-backend:latest

---

## Frontend Image

Build frontend image:

    docker build -t three-tier-frontend ./frontend

Tag frontend image:

    docker tag three-tier-frontend:latest 714950921352.dkr.ecr.us-east-1.amazonaws.com/three-tier-frontend:latest

Push frontend image:

    docker push 714950921352.dkr.ecr.us-east-1.amazonaws.com/three-tier-frontend:latest

---

## List Images in ECR

Backend:

    aws ecr list-images \
      --repository-name three-tier-backend \
      --region us-east-1

Frontend:

    aws ecr list-images \
      --repository-name three-tier-frontend \
      --region us-east-1

MongoDB:

    aws ecr list-images \
      --repository-name three-tier-mongodb \
      --region us-east-1

---

# 9. Amazon ECS Commands

## List ECS Clusters

    aws ecs list-clusters --region us-east-1

Cluster used by the project:

    three-tier-cluster

---

## List ECS Services

    aws ecs list-services \
      --cluster three-tier-cluster \
      --region us-east-1

---

## Check ECS Service

Frontend:

    aws ecs describe-services \
      --cluster three-tier-cluster \
      --services three-tier-frontend-service \
      --region us-east-1

Backend:

    aws ecs describe-services \
      --cluster three-tier-cluster \
      --services three-tier-backend-service \
      --region us-east-1

MongoDB:

    aws ecs describe-services \
      --cluster three-tier-cluster \
      --services three-tier-mongodb-service-2 \
      --region us-east-1

---

## List Running Tasks

    aws ecs list-tasks \
      --cluster three-tier-cluster \
      --region us-east-1

---

## List Tasks for a Specific Service

Frontend:

    aws ecs list-tasks \
      --cluster three-tier-cluster \
      --service-name three-tier-frontend-service \
      --region us-east-1

Backend:

    aws ecs list-tasks \
      --cluster three-tier-cluster \
      --service-name three-tier-backend-service \
      --region us-east-1

MongoDB:

    aws ecs list-tasks \
      --cluster three-tier-cluster \
      --service-name three-tier-mongodb-service-2 \
      --region us-east-1

---

## Describe ECS Task

    aws ecs describe-tasks \
      --cluster three-tier-cluster \
      --tasks <TASK_ARN> \
      --region us-east-1

---

## Force New ECS Deployment

Backend:

    aws ecs update-service \
      --cluster three-tier-cluster \
      --service three-tier-backend-service \
      --force-new-deployment \
      --region us-east-1

Frontend:

    aws ecs update-service \
      --cluster three-tier-cluster \
      --service three-tier-frontend-service \
      --force-new-deployment \
      --region us-east-1

---

# 10. AWS Cloud Map Commands

## List Namespaces

    aws servicediscovery list-namespaces \
      --region us-east-1

Project namespace:

    three-tier.local

---

## Discover Backend Service

    aws servicediscovery discover-instances \
      --namespace-name three-tier.local \
      --service-name backend \
      --region us-east-1

---

## Discover MongoDB Service

    aws servicediscovery discover-instances \
      --namespace-name three-tier.local \
      --service-name mongodb \
      --region us-east-1

---

## List Cloud Map Services

    aws servicediscovery list-services \
      --region us-east-1

---

# 11. Backend API Testing on AWS

Test the backend directly:

    curl http://<BACKEND_PUBLIC_IP>:5000/tasks

Test the frontend Nginx API:

    curl http://<FRONTEND_PUBLIC_IP>/api/tasks

A successful request should return:

    HTTP 200 OK

with task data.

---

# 12. Nginx Configuration

The Nginx configuration file is located at:

    frontend/nginx.conf

The deployed configuration forwards:

    /api/

to:

    backend.three-tier.local:5000

The configuration can be viewed locally using:

    type frontend\nginx.conf

---

# 13. Network Testing Commands

## Test Backend Port from Windows PowerShell

    Test-NetConnection <BACKEND_PUBLIC_IP> -Port 5000

## Test Frontend Port

    Test-NetConnection <FRONTEND_PUBLIC_IP> -Port 80

---

# 14. AWS Reachability Analyzer

The frontend-to-backend network path was verified using AWS Reachability Analyzer.

The tested connection was:

    Frontend ECS ENI
          ↓
    Backend ECS ENI
          ↓
    TCP :5000

The final analysis result was successful.

The AWS CLI can be used to inspect network insights paths with:

    aws ec2 describe-network-insights-paths \
      --network-insights-path-ids <PATH_ID> \
      --region us-east-1

To inspect an analysis:

    aws ec2 describe-network-insights-analyses \
      --network-insights-analysis-ids <ANALYSIS_ID> \
      --region us-east-1

---

# 15. GitHub Actions

The GitHub Actions workflow is located at:

    .github/workflows/deploy.yml

The workflow is triggered by pushes to:

    main

The deployment workflow performs:

    Checkout
      ↓
    AWS OIDC Authentication
      ↓
    ECR Login
      ↓
    Backend Docker Build
      ↓
    Backend Image Push
      ↓
    Frontend Docker Build
      ↓
    Frontend Image Push
      ↓
    ECS Backend Deployment
      ↓
    ECS Frontend Deployment

---

# 16. AWS OIDC Verification

Check the current AWS identity:

    aws sts get-caller-identity

The GitHub Actions IAM role used by the project is:

    GitHubActionsThreeTierDeploy

The role ARN is:

    arn:aws:iam::714950921352:role/GitHubActionsThreeTierDeploy

GitHub Actions successfully authenticated using AWS OIDC.

---

# 17. Docker Cleanup Commands

Stop the local application:

    docker compose down

Remove locally created Compose images:

    docker compose down --rmi local

Check Docker containers:

    docker ps -a

Remove unused Docker resources:

    docker system prune

---

# 18. ECS Cleanup Commands

Scale services to zero before deleting them.

Frontend:

    aws ecs update-service \
      --cluster three-tier-cluster \
      --service three-tier-frontend-service \
      --desired-count 0 \
      --region us-east-1

Backend:

    aws ecs update-service \
      --cluster three-tier-cluster \
      --service three-tier-backend-service \
      --desired-count 0 \
      --region us-east-1

MongoDB:

    aws ecs update-service \
      --cluster three-tier-cluster \
      --service three-tier-mongodb-service-2 \
      --desired-count 0 \
      --region us-east-1

---

# 19. Useful Verification Commands

Check ECS service status:

    aws ecs describe-services \
      --cluster three-tier-cluster \
      --services three-tier-frontend-service three-tier-backend-service three-tier-mongodb-service-2 \
      --region us-east-1

Check running ECS tasks:

    aws ecs list-tasks \
      --cluster three-tier-cluster \
      --desired-status RUNNING \
      --region us-east-1

Check ECR repositories:

    aws ecr describe-repositories \
      --region us-east-1

Check Cloud Map namespaces:

    aws servicediscovery list-namespaces \
      --region us-east-1

Check AWS identity:

    aws sts get-caller-identity

---

# 20. Main Project Commands Summary

| Purpose | Command |
|---|---|
| Start local application | `docker compose up -d` |
| Stop local application | `docker compose down` |
| Build Docker images | `docker compose build` |
| Check containers | `docker ps` |
| View logs | `docker compose logs` |
| Test local backend | `curl http://localhost:5000/tasks` |
| Test deployed API | `curl http://<FRONTEND_PUBLIC_IP>/api/tasks` |
| Check AWS identity | `aws sts get-caller-identity` |
| List ECS clusters | `aws ecs list-clusters` |
| List ECS services | `aws ecs list-services --cluster three-tier-cluster` |
| List ECS tasks | `aws ecs list-tasks --cluster three-tier-cluster` |
| Discover backend | `aws servicediscovery discover-instances --namespace-name three-tier.local --service-name backend` |
| Discover MongoDB | `aws servicediscovery discover-instances --namespace-name three-tier.local --service-name mongodb` |
| Push Git changes | `git push origin main` |
| Pull Git changes | `git pull origin main --rebase` |

---

## 21. Project-Specific AWS Values

| Resource | Value |
|---|---|
| AWS Region | `us-east-1` |
| AWS Account ID | `714950921352` |
| ECS Cluster | `three-tier-cluster` |
| Frontend ECS Service | `three-tier-frontend-service` |
| Backend ECS Service | `three-tier-backend-service` |
| MongoDB ECS Service | `three-tier-mongodb-service-2` |
| Frontend ECR | `three-tier-frontend` |
| Backend ECR | `three-tier-backend` |
| MongoDB ECR | `three-tier-mongodb` |
| Cloud Map Namespace | `three-tier.local` |
| Backend Service | `backend` |
| MongoDB Service | `mongodb` |
| VPC | `vpc-03237cb2bc44a1174` |
| Security Group | `sg-012215fb155d043a5` |
| GitHub OIDC Role | `GitHubActionsThreeTierDeploy` |

---

## 22. Final Command Flow

The main workflow used during the project can be summarized as:

    cd "C:\Users\Shreya Gupta\Desktop\three-tier-app"

    docker compose build

    docker compose up -d

    docker ps

    curl http://localhost:5000/tasks

    git status

    git add .

    git commit -m "Update project"

    git pull origin main --rebase

    git push origin main

GitHub Actions then performs the automated AWS deployment.

The deployed application can be verified using:

    curl http://<FRONTEND_PUBLIC_IP>/api/tasks

A successful response confirms that the frontend, Nginx reverse proxy, Cloud Map service discovery, and backend API are working together.
