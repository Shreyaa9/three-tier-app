# 🚀 Three-Tier To-Do Application on AWS

A containerized three-tier To-Do application deployed on **AWS ECS Fargate** using Docker, Amazon ECR, AWS Cloud Map, Nginx, Node.js, MongoDB, and GitHub Actions CI/CD with GitHub OIDC authentication.

---

## 📌 Project Overview

This project implements a **three-tier web application** consisting of:

- **Frontend:** HTML, CSS, JavaScript served using Nginx
- **Backend:** Node.js and Express REST API
- **Database:** MongoDB

The application is containerized using **Docker** and deployed on **Amazon ECS Fargate**.

AWS **Cloud Map** is used for service discovery between the application components, while **Nginx** acts as a reverse proxy between the frontend and backend.

A **GitHub Actions CI/CD pipeline** automatically builds Docker images, pushes them to Amazon ECR, and triggers ECS deployments whenever changes are pushed to the `main` branch.

---

## 🎯 Objectives

- Build a complete three-tier web application.
- Containerize all application components using Docker.
- Deploy the application using Amazon ECS Fargate.
- Store Docker images in Amazon ECR.
- Implement service discovery using AWS Cloud Map.
- Configure Nginx as a reverse proxy.
- Connect the backend with MongoDB.
- Implement CI/CD using GitHub Actions.
- Configure secure GitHub-to-AWS authentication using OIDC.
- Test the deployed application end-to-end.

---

## ✨ Features

- Add new To-Do tasks.
- View existing tasks.
- Mark tasks as completed.
- Store tasks persistently in MongoDB.
- Refresh the page without losing stored tasks.
- Frontend-to-backend communication through Nginx.
- Automated Docker image build and deployment through GitHub Actions.

---

## 🛠️ Technology Stack

| Layer | Technology |
|---|---|
| Frontend | HTML, CSS, JavaScript |
| Web Server | Nginx |
| Backend | Node.js, Express.js |
| Database | MongoDB |
| Containerization | Docker |
| Container Orchestration | Amazon ECS Fargate |
| Container Registry | Amazon ECR |
| Service Discovery | AWS Cloud Map |
| CI/CD | GitHub Actions |
| Authentication | GitHub OIDC + AWS IAM |
| Version Control | Git & GitHub |
| Cloud Platform | AWS |

---

## 🏗️ Architecture

    User / Browser
           |
           | HTTP :80
           v
    +----------------------+
    |   Frontend Container |
    |        Nginx         |
    |        Port 80       |
    +----------+-----------+
               |
               | /api/*
               | Reverse Proxy
               v
    +----------------------+
    |   Backend Container  |
    |   Node.js + Express  |
    |       Port 5000      |
    +----------+-----------+
               |
               | MongoDB Connection
               v
    +----------------------+
    |   MongoDB Container  |
    |       Port 27017     |
    +----------------------+


For the AWS deployment, these components run as separate **ECS Fargate services**.

---

## ☁️ AWS Architecture

    Internet
       |
       v
    User Browser
       |
       | HTTP :80
       v
    +--------------------------+
    |   ECS Fargate Frontend  |
    |       Nginx :80         |
    +------------+-------------+
                 |
                 | Cloud Map
                 | backend.three-tier.local
                 | :5000
                 v
    +--------------------------+
    |   ECS Fargate Backend   |
    |    Node.js / Express    |
    |        :5000            |
    +------------+-------------+
                 |
                 | Cloud Map
                 | mongodb.three-tier.local
                 | :27017
                 v
    +--------------------------+
    |   ECS Fargate MongoDB   |
    |        :27017            |
    +--------------------------+

    Private Namespace:
    three-tier.local

---

## 🔄 Application Flow

The request flow of the deployed application is:

    Browser
       ↓
    Frontend Nginx
       ↓
    /api/tasks
       ↓
    backend.three-tier.local
       ↓
    Node.js / Express Backend
       ↓
    mongodb.three-tier.local
       ↓
    MongoDB
       ↓
    Response
       ↓
    Browser

Nginx receives `/api/` requests and forwards them to the backend using AWS Cloud Map service discovery.

---

## 🐳 Docker

The project contains three Dockerized components.

### Frontend

- Base image: `nginx:alpine`
- Port: `80`
- Serves the frontend application.
- Acts as a reverse proxy for backend API requests.

### Backend

- Base image: `node:18-alpine`
- Port: `5000`
- Runs the Node.js/Express REST API.
- Connects to MongoDB using `MONGO_URI`.

### MongoDB

- Image: `mongo:latest`
- Port: `27017`
- Stores To-Do tasks.

---

## 📁 Project Structure

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
    │
    ├── README.md
    ├── ARCHITECTURE.md
    ├── SETUP.md
    ├── COMMANDS.md
    ├── TESTING.md
    ├── VERIFICATION.md
    ├── TROUBLESHOOTING.md
    ├── CLEANUP.md
    │
    ├── architecture.png
    │
    └── screenshots/

---

## 🔌 Backend API

The backend provides REST API endpoints for managing tasks.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/tasks` | Retrieve all tasks |
| POST | `/tasks` | Create a new task |
| PUT | `/tasks/:id` | Update a task |

Backend port:

    5000

MongoDB database:

    todoDB

---

## 🌐 Nginx Reverse Proxy

Nginx is used as the entry point for the frontend application.

API requests are sent through:

    /api/*

Nginx forwards these requests to:

    backend.three-tier.local:5000

The backend service is discovered using AWS Cloud Map rather than relying on a fixed ECS task IP.

---

## 🗺️ AWS Cloud Map

The project uses a private AWS Cloud Map namespace:

    three-tier.local

Services registered in the namespace include:

    backend.three-tier.local
    mongodb.three-tier.local

This allows ECS services to communicate using service names even when task IP addresses change.

---

## 🚀 AWS Deployment

### AWS Region

    us-east-1

### ECS Cluster

    three-tier-cluster

### ECS Services

    three-tier-frontend-service
    three-tier-backend-service
    three-tier-mongodb-service-2

### ECR Repositories

    three-tier-frontend
    three-tier-backend
    three-tier-mongodb

### VPC

    vpc-03237cb2bc44a1174

---

## 🔐 Security

The project uses AWS security mechanisms including:

- AWS IAM
- Security Groups
- GitHub OIDC
- Private AWS Cloud Map namespace
- ECS task networking

GitHub Actions authenticates with AWS using **OIDC** instead of storing long-term AWS access keys.

The IAM role used by GitHub Actions is:

    GitHubActionsThreeTierDeploy

The application security group allows the required application traffic on:

    80      → Frontend
    5000    → Backend
    27017   → MongoDB

MongoDB communication is restricted to the application security group.

---

## ⚙️ CI/CD Pipeline

GitHub Actions is used to automate deployment.

The workflow is triggered when changes are pushed to the `main` branch.

    Git Push
       ↓
    GitHub Actions
       ↓
    Authenticate using GitHub OIDC
       ↓
    Login to Amazon ECR
       ↓
    Build Backend Image
       ↓
    Push Backend Image
       ↓
    Build Frontend Image
       ↓
    Push Frontend Image
       ↓
    Force ECS Backend Deployment
       ↓
    Force ECS Frontend Deployment

Workflow file:

    .github/workflows/deploy.yml

---

## 🧪 Testing

The application was tested both locally and after AWS deployment.

### Local Testing

Docker Compose was used to verify:

- Frontend availability
- Backend API
- MongoDB connectivity
- Task creation
- Task completion
- Data persistence

### AWS Testing

The deployed application was verified for:

- Frontend accessibility
- Backend API response
- Nginx reverse proxy
- Cloud Map service discovery
- Task creation
- Task completion
- MongoDB persistence
- Page refresh persistence

The backend API returned:

    HTTP 200 OK

for successful task retrieval.

---

## 🛠️ Major Problems Solved

During development and deployment, several issues were encountered and resolved.

### MongoDB Container Issue

The MongoDB container initially exited unexpectedly.

The issue was resolved using:

    GLIBC_TUNABLES: "glibc.pthread.rseq=1"

### Nginx Proxy Issue

The frontend initially could not communicate correctly with the backend.

Nginx was configured to forward:

    /api/*

to the backend service.

### ECS Service Discovery

Directly using ECS task IP addresses was avoided because task IPs can change.

AWS Cloud Map was configured for service discovery.

### Network Connectivity

AWS Reachability Analyzer was used to verify the frontend-to-backend TCP connection on port `5000`.

### GitHub Actions Authentication

GitHub Actions was configured with AWS OIDC and the IAM role:

    GitHubActionsThreeTierDeploy

This enabled automated deployment without storing long-term AWS access keys.

---

## 📚 Key Learning Outcomes

This project provided practical experience with:

- Docker and Docker Compose
- Docker networking
- Amazon ECS Fargate
- Amazon ECR
- AWS Cloud Map
- Nginx reverse proxy
- Node.js and Express
- MongoDB
- AWS VPC and Security Groups
- AWS IAM
- GitHub Actions
- GitHub OIDC
- CI/CD automation
- AWS networking troubleshooting
- Application deployment and verification

---

## 📊 Final Architecture

                    +-----------------+
                    |      User       |
                    |    Browser      |
                    +--------+--------+
                             |
                             | HTTP :80
                             v
                    +-----------------+
                    |    Frontend     |
                    |      Nginx      |
                    |    ECS Fargate  |
                    +--------+--------+
                             |
                             | Cloud Map
                             | :5000
                             v
                    +-----------------+
                    |     Backend     |
                    | Node.js/Express |
                    |    ECS Fargate  |
                    +--------+--------+
                             |
                             | Cloud Map
                             | :27017
                             v
                    +-----------------+
                    |     MongoDB     |
                    |    ECS Fargate  |
                    +-----------------+

---

## ✅ Final Status

The application was successfully:

- Developed locally
- Containerized using Docker
- Tested using Docker Compose
- Pushed to Amazon ECR
- Deployed on ECS Fargate
- Connected using AWS Cloud Map
- Configured with Nginx reverse proxy
- Connected to MongoDB
- Tested through the deployed frontend
- Integrated with GitHub Actions
- Authenticated using GitHub OIDC

The final application successfully supports:

    Add Task
    Complete Task
    Refresh Page
    Persistent Task Storage

---

## 🔗 Repository

GitHub Repository:

    https://github.com/Shreyaa9/three-tier-app

---

## 👩‍💻 Project

**Three-Tier To-Do Application on AWS**

Built using:

**Docker + AWS ECS Fargate + ECR + Cloud Map + Nginx + Node.js + MongoDB + GitHub Actions + GitHub OIDC**
