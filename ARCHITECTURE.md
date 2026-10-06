# Architecture — Three-Tier To-Do Application

## 1. Overview

The Three-Tier To-Do Application is a containerized web application deployed on AWS ECS Fargate.

The application follows a three-tier architecture:

1. Presentation Tier — Frontend with Nginx
2. Application Tier — Node.js/Express Backend
3. Data Tier — MongoDB

The application is containerized using Docker and deployed using Amazon ECS Fargate. Amazon ECR is used to store Docker images, while AWS Cloud Map provides service discovery between ECS services.

---

## 2. High-Level Architecture

    ┌─────────────────────────────┐
    │           User              │
    │          Browser            │
    └──────────────┬──────────────┘
                   │
                   │ HTTP :80
                   ▼
    ┌─────────────────────────────┐
    │      Frontend Container      │
    │          Nginx :80           │
    │                             │
    │  Static HTML/CSS/JavaScript │
    │  Reverse Proxy              │
    └──────────────┬──────────────┘
                   │
                   │ /api/tasks
                   ▼
    ┌─────────────────────────────┐
    │        AWS Cloud Map         │
    │       three-tier.local       │
    │                             │
    │  backend.three-tier.local   │
    └──────────────┬──────────────┘
                   │
                   │ TCP :5000
                   ▼
    ┌─────────────────────────────┐
    │       Backend Container      │
    │     Node.js / Express :5000 │
    │                             │
    │        REST API             │
    └──────────────┬──────────────┘
                   │
                   │ MongoDB
                   ▼
    ┌─────────────────────────────┐
    │        AWS Cloud Map         │
    │       mongodb.three-tier     │
    │           .local             │
    └──────────────┬──────────────┘
                   │
                   │ TCP :27017
                   ▼
    ┌─────────────────────────────┐
    │       MongoDB Container      │
    │          :27017              │
    │                             │
    │         todoDB              │
    └─────────────────────────────┘

---

## 3. Three-Tier Architecture

### 3.1 Presentation Tier

The presentation tier is implemented using:

- HTML
- JavaScript
- Nginx
- Docker

The frontend is packaged as a Docker image and runs inside an ECS Fargate task.

Nginx performs two main functions:

- Serves the frontend application.
- Acts as a reverse proxy for backend API requests.

Frontend port:

    80

---

### 3.2 Application Tier

The application tier is implemented using:

- Node.js
- Express.js
- Mongoose
- Docker

The backend provides REST APIs for task management.

Backend port:

    5000

Available APIs:

    GET /tasks
    POST /tasks
    PUT /tasks/:id

The backend receives requests from the frontend through the Nginx reverse proxy.

---

### 3.3 Data Tier

The data tier uses MongoDB.

MongoDB stores the application task data in:

    todoDB

MongoDB port:

    27017

The backend connects to MongoDB using the MongoDB service name rather than relying on a fixed IP address.

---

## 4. Local Docker Architecture

The application was also tested locally using Docker Compose.

The local architecture is:

    Browser
       │
       ▼
    Frontend / Nginx
       │
       │ :5000
       ▼
    Backend / Node.js
       │
       │ :27017
       ▼
    MongoDB

Docker Compose defines the three services:

    mongodb
    backend
    frontend

The backend uses the MongoDB service name:

    mongodb

for database communication.

---

## 5. AWS Deployment Architecture

The production deployment uses Amazon ECS Fargate.

The ECS cluster is:

    three-tier-cluster

The application is divided into separate ECS services:

    three-tier-frontend-service
    three-tier-backend-service
    three-tier-mongodb-service-2

Each service runs its corresponding container.

---

## 6. AWS VPC Architecture

The ECS services are deployed inside the same VPC:

    vpc-03237cb2bc44a1174

The application uses the following subnets:

    subnet-08a1275921516ed82
    subnet-00bd0215a8ab1068c

The ECS tasks use `awsvpc` networking.

This gives each ECS task its own network interface and private IP address.

---

## 7. Security Group

The application uses the security group:

    sg-012215fb155d043a5

The required application ports are:

| Port | Purpose |
|---|---|
| 80 | Frontend / Nginx |
| 5000 | Backend API |
| 27017 | MongoDB |

MongoDB communication is restricted to the application security group.

---

## 8. AWS Cloud Map Service Discovery

AWS Cloud Map is used for service discovery between ECS services.

Private namespace:

    three-tier.local

Registered services:

    backend
    mongodb

The backend can therefore be reached using:

    backend.three-tier.local

MongoDB can be reached using:

    mongodb.three-tier.local

This allows the application to communicate using service names instead of hard-coded private IP addresses.

---

## 9. Nginx Reverse Proxy Architecture

The frontend Nginx server handles requests received from the browser.

Normal frontend requests are served directly by Nginx.

API requests using:

    /api/

are forwarded to the backend.

The request flow is:

    Browser
       ↓
    /api/tasks
       ↓
    Nginx
       ↓
    backend.three-tier.local:5000
       ↓
    Node.js Backend

The Nginx configuration uses the AWS VPC DNS resolver to resolve the Cloud Map backend service.

---

## 10. Backend-to-MongoDB Architecture

The backend communicates with MongoDB through AWS Cloud Map.

The communication flow is:

    Node.js Backend
          ↓
    mongodb.three-tier.local
          ↓
    MongoDB :27017
          ↓
    todoDB

This provides service-based communication between the application and database tiers.

---

## 11. Container Architecture

### Frontend Container

Base image:

    nginx:alpine

Responsibilities:

- Serve frontend files.
- Handle HTTP requests.
- Reverse proxy API requests.

Port:

    80

---

### Backend Container

Base image:

    node:18-alpine

Responsibilities:

- Run Node.js application.
- Provide REST APIs.
- Communicate with MongoDB.
- Handle task operations.

Port:

    5000

---

### MongoDB Container

Image:

    mongo:latest

Responsibilities:

- Store task data.
- Provide database services to the backend.

Port:

    27017

---

## 12. Amazon ECR Architecture

Amazon ECR is used to store the Docker images used by ECS.

The project uses the following repositories:

    three-tier-frontend
    three-tier-backend
    three-tier-mongodb

The deployment flow is:

    Source Code
        ↓
    Docker Build
        ↓
    Amazon ECR
        ↓
    ECS Fargate
        ↓
    Running Containers

---

## 13. CI/CD Architecture

GitHub Actions is used to automate the deployment process.

The workflow is triggered when changes are pushed to the `main` branch.

The deployment flow is:

    Developer
        ↓
    GitHub Repository
        ↓
    GitHub Actions
        ↓
    AWS OIDC Authentication
        ↓
    Amazon ECR
        ↓
    ECS Deployment
        ↓
    Updated Frontend & Backend

The workflow builds and pushes the frontend and backend Docker images and then forces new ECS deployments.

---

## 14. GitHub OIDC Architecture

GitHub Actions authenticates with AWS using OpenID Connect (OIDC).

IAM role:

    GitHubActionsThreeTierDeploy

The workflow assumes this role without storing long-term AWS access keys inside GitHub.

The authentication flow is:

    GitHub Actions
          ↓
    GitHub OIDC Provider
          ↓
    AWS STS
          ↓
    GitHubActionsThreeTierDeploy
          ↓
    AWS ECR / ECS

---

## 15. Complete Application Request Flow

The complete request flow is:

    User Browser
          │
          ▼
    ECS Frontend
          │
          ▼
    Nginx
          │
          │ /api/tasks
          ▼
    backend.three-tier.local
          │
          ▼
    ECS Backend
          │
          ▼
    mongodb.three-tier.local
          │
          ▼
    ECS MongoDB
          │
          ▼
        todoDB

The response then travels back through:

    MongoDB
       ↓
    Backend
       ↓
    Nginx
       ↓
    Frontend
       ↓
    Browser

---

## 16. Final Architecture Components

| Layer | Technology | Port | AWS Component |
|---|---|---:|---|
| Presentation | HTML, JavaScript, Nginx | 80 | ECS Fargate |
| Application | Node.js, Express, Mongoose | 5000 | ECS Fargate |
| Data | MongoDB | 27017 | ECS Fargate |
| Container Registry | Docker Images | — | Amazon ECR |
| Service Discovery | AWS Cloud Map | — | Private Namespace |
| CI/CD | GitHub Actions | — | GitHub + AWS OIDC |
| Networking | VPC / awsvpc | — | Amazon VPC |

---

## 17. Architecture Benefits

The implemented architecture provides:

- Separation of frontend, backend, and database layers.
- Independent containerization of application components.
- ECS Fargate-based deployment without managing servers.
- Service discovery using AWS Cloud Map.
- Nginx-based reverse proxying.
- Docker-based local development and testing.
- Amazon ECR-based container image management.
- Automated deployment using GitHub Actions.
- Secure AWS authentication through GitHub OIDC.
- Scalable service-oriented application structure.

---

## 18. Final Architecture

The final deployed architecture can be summarized as:

    ┌──────────────────────┐
    │        User          │
    │      Browser         │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ ECS Fargate Frontend │
    │      Nginx :80       │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │     AWS Cloud Map    │
    │   backend service    │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ ECS Fargate Backend  │
    │ Node.js / Express    │
    │       :5000          │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │     AWS Cloud Map    │
    │   mongodb service    │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │  ECS Fargate MongoDB │
    │       :27017         │
    │       todoDB         │
    └──────────────────────┘

This architecture represents the final working deployment of the Three-Tier To-Do Application.
