# Setup Guide — Three-Tier To-Do Application

This guide explains how to set up, run, and deploy the Three-Tier To-Do Application using Docker locally and AWS ECS Fargate.

---

## 1. Prerequisites

Make sure the following tools are installed and configured.

### Required Software

- Git
- Docker
- Docker Compose
- AWS CLI
- An AWS account
- GitHub account

### AWS Services Used

- Amazon ECR
- Amazon ECS
- AWS Cloud Map
- Amazon VPC
- AWS IAM
- Amazon EC2 networking components
- GitHub Actions

---

# 2. Clone the Repository

Clone the existing GitHub repository:

    git clone https://github.com/Shreyaa9/three-tier-app.git

Move into the project directory:

    cd three-tier-app

---

# 3. Project Structure

After cloning the repository, the main structure should look like:

    three-tier-app/
    │
    ├── backend/
    │   ├── Dockerfile
    │   ├── package.json
    │   └── server.js
    │
    ├── frontend/
    │   ├── Dockerfile
    │   ├── index.html
    │   └── nginx.conf
    │
    ├── docker-compose.yml
    ├── Jenkinsfile
    └── .github/
        └── workflows/
            └── deploy.yml

---

# 4. Local Setup Using Docker Compose

The application can be tested locally before deploying it to AWS.

The Docker Compose configuration contains three services:

    mongodb
    backend
    frontend

The communication flow is:

    Browser
       ↓
    Frontend :80
       ↓
    Backend :5000
       ↓
    MongoDB :27017

---

## 4.1 Build the Containers

From the project root directory, run:

    docker compose build

This builds the frontend and backend Docker images.

---

## 4.2 Start the Application

Run:

    docker compose up -d

The `-d` option starts the containers in detached mode.

Check running containers:

    docker ps

The three services should be running:

    frontend
    backend
    mongodb

---

## 4.3 Check Application Logs

To view all logs:

    docker compose logs

To view backend logs:

    docker compose logs backend

To view frontend logs:

    docker compose logs frontend

To view MongoDB logs:

    docker compose logs mongodb

To follow logs continuously:

    docker compose logs -f

---

# 5. Test the Local Application

Open a browser and visit:

    http://localhost

The To-Do application should be displayed.

Test the following operations:

- Add a task.
- View the task.
- Mark the task as completed.
- Refresh the page.
- Verify that the task is still present.

---

# 6. Test the Backend API Locally

The backend runs on port `5000`.

To retrieve tasks:

    curl http://localhost:5000/tasks

A successful request should return an HTTP response containing the stored tasks.

The main backend endpoints are:

    GET  /tasks
    POST /tasks
    PUT  /tasks/:id

---

# 7. Stop the Local Application

To stop the containers:

    docker compose down

To stop the containers and remove associated volumes:

    docker compose down -v

Use the second command only when the local MongoDB data is no longer required.

---

# 8. AWS Configuration

The deployed application uses AWS ECS Fargate.

The project was deployed in:

    AWS Region: us-east-1

The ECS cluster is:

    three-tier-cluster

The ECS services are:

    three-tier-frontend-service
    three-tier-backend-service
    three-tier-mongodb-service-2

---

# 9. Configure AWS CLI

Configure the AWS CLI using an AWS IAM user or another authorized AWS identity:

    aws configure

Provide the required AWS credentials and region.

Set the region to:

    us-east-1

Verify the configuration:

    aws sts get-caller-identity

This should return information about the AWS account and identity being used.

---

# 10. Amazon ECR Setup

Create or use the following ECR repositories:

    three-tier-frontend
    three-tier-backend
    three-tier-mongodb

Authenticate Docker with Amazon ECR:

    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 714950921352.dkr.ecr.us-east-1.amazonaws.com

---

# 11. Build Docker Images

Build the backend image:

    docker build -t three-tier-backend ./backend

Build the frontend image:

    docker build -t three-tier-frontend ./frontend

---

# 12. Tag Docker Images

Tag the backend image:

    docker tag three-tier-backend:latest 714950921352.dkr.ecr.us-east-1.amazonaws.com/three-tier-backend:latest

Tag the frontend image:

    docker tag three-tier-frontend:latest 714950921352.dkr.ecr.us-east-1.amazonaws.com/three-tier-frontend:latest

---

# 13. Push Images to Amazon ECR

Push the backend image:

    docker push 714950921352.dkr.ecr.us-east-1.amazonaws.com/three-tier-backend:latest

Push the frontend image:

    docker push 714950921352.dkr.ecr.us-east-1.amazonaws.com/three-tier-frontend:latest

MongoDB uses the MongoDB container image and does not require building the application image locally.

---

# 14. ECS Configuration

The ECS deployment uses:

    Launch Type: Fargate
    Network Mode: awsvpc
    Region: us-east-1
    Cluster: three-tier-cluster

The ECS services are deployed as separate services:

    Frontend
    Backend
    MongoDB

Each service runs as its own ECS task.

---

# 15. Network Configuration

The ECS tasks use the VPC:

    vpc-03237cb2bc44a1174

Subnets used by the application include:

    subnet-08a1275921516ed82
    subnet-00bd0215a8ab1068c

Security group:

    sg-012215fb155d043a5

Required application ports:

    80      → Frontend
    5000    → Backend
    27017   → MongoDB

MongoDB access is restricted to the application security group.

---

# 16. AWS Cloud Map Setup

The application uses a private Cloud Map namespace:

    three-tier.local

The services are registered using:

    backend.three-tier.local
    mongodb.three-tier.local

The backend uses the MongoDB service name for database connectivity.

The frontend Nginx configuration uses:

    backend.three-tier.local

for forwarding API requests to the backend.

This avoids depending on fixed ECS task IP addresses.

---

# 17. Backend Environment Variable

The backend requires the MongoDB connection string.

For the ECS backend service, the MongoDB URI is configured to use the Cloud Map service:

    mongodb://mongodb.three-tier.local:27017/todoDB

For local Docker Compose, the backend uses:

    mongodb://mongodb:27017/todoDB

The application therefore uses Docker service discovery locally and AWS Cloud Map service discovery in the AWS environment.

---

# 18. Nginx Configuration

The frontend uses:

    frontend/nginx.conf

Nginx serves the frontend files and forwards API requests.

The API path is:

    /api/

The backend service is resolved using:

    backend.three-tier.local

The request flow is:

    Browser
       ↓
    Nginx :80
       ↓
    backend.three-tier.local:5000
       ↓
    Node.js Backend

---

# 19. ECS Task Definitions

Create task definitions for:

    Frontend
    Backend
    MongoDB

The frontend container exposes:

    80

The backend container exposes:

    5000

MongoDB exposes:

    27017

The ECS task definitions use:

    Fargate
    awsvpc

The backend task definition must contain the MongoDB environment variable.

---

# 20. ECS Services

Create the following ECS services inside:

    three-tier-cluster

Frontend:

    three-tier-frontend-service

Backend:

    three-tier-backend-service

MongoDB:

    three-tier-mongodb-service-2

For the frontend service, public IP assignment is enabled so the application can be accessed from the browser.

---

# 21. GitHub Actions Setup

The project contains the GitHub Actions workflow:

    .github/workflows/deploy.yml

The workflow is triggered by pushes to:

    main

The workflow performs:

    Checkout
       ↓
    AWS OIDC Authentication
       ↓
    ECR Login
       ↓
    Build Backend
       ↓
    Push Backend
       ↓
    Build Frontend
       ↓
    Push Frontend
       ↓
    Deploy Backend
       ↓
    Deploy Frontend

---

# 22. GitHub OIDC Configuration

GitHub Actions uses OpenID Connect to authenticate with AWS.

The IAM role used by the workflow is:

    GitHubActionsThreeTierDeploy

Role ARN:

    arn:aws:iam::714950921352:role/GitHubActionsThreeTierDeploy

The role is trusted by:

    token.actions.githubusercontent.com

The workflow uses:

    sts:AssumeRoleWithWebIdentity

This removes the need to store long-term AWS access keys in GitHub Actions secrets.

---

# 23. GitHub Repository

The project repository is:

    https://github.com/Shreyaa9/three-tier-app

The repository contains the source code, Docker configuration, documentation, architecture diagram, screenshots, and CI/CD workflow.

---

# 24. Deploy Using GitHub Actions

After making changes locally:

    git add .

Commit the changes:

    git commit -m "Update application"

Push to the main branch:

    git push origin main

GitHub Actions will automatically start the deployment workflow.

Check the workflow under:

    GitHub Repository
        ↓
    Actions
        ↓
    Deploy Three-Tier App

---

# 25. Verify ECS Deployment

After GitHub Actions completes successfully, verify the ECS services.

Check:

    ECS
      ↓
    Clusters
      ↓
    three-tier-cluster

Verify that these services are running:

    three-tier-frontend-service
    three-tier-backend-service
    three-tier-mongodb-service-2

Also verify that the ECS tasks are in the:

    RUNNING

state.

---

# 26. Access the Application

Find the public IP of the frontend ECS task.

Open the following in a browser:

    http://<FRONTEND_PUBLIC_IP>

The To-Do application should load.

Test:

    1. Add a task.
    2. Verify that the task appears.
    3. Complete the task.
    4. Refresh the page.
    5. Verify that the task remains stored.

---

# 27. Verify Backend Connectivity

The frontend communicates with the backend through:

    backend.three-tier.local:5000

The backend communicates with MongoDB through:

    mongodb.three-tier.local:27017

AWS Cloud Map provides the service discovery required for this communication.

AWS Reachability Analyzer can be used to verify the network path between ECS task ENIs.

---

# 28. Troubleshooting During Setup

If the frontend loads but API operations fail, check:

    1. Backend ECS task status.
    2. Backend Cloud Map registration.
    3. Nginx configuration.
    4. Security group rules.
    5. Backend port 5000.
    6. MongoDB service status.
    7. MongoDB service discovery.
    8. ECS task logs.

If the backend cannot connect to MongoDB, verify:

    mongodb://mongodb.three-tier.local:27017/todoDB

and confirm that the MongoDB service is registered in Cloud Map.

---

# 29. Successful Setup Verification

The setup is considered successful when all of the following work:

- Docker Compose starts successfully.
- Frontend opens locally.
- Backend API responds.
- MongoDB stores tasks.
- ECS services remain in the RUNNING state.
- Cloud Map services are registered.
- Nginx resolves the backend service.
- Frontend can call the backend.
- Backend can connect to MongoDB.
- Tasks can be added.
- Tasks can be completed.
- Tasks persist after refresh.
- GitHub Actions completes successfully.

---

# 30. Final Deployment Flow

The complete setup and deployment flow is:

    Clone Repository
          ↓
    Local Docker Build
          ↓
    Docker Compose Testing
          ↓
    Amazon ECR
          ↓
    ECS Fargate
          ↓
    AWS Cloud Map
          ↓
    Nginx Reverse Proxy
          ↓
    Node.js Backend
          ↓
    MongoDB
          ↓
    Application Verification

---

## Project Repository

    https://github.com/Shreyaa9/three-tier-app
