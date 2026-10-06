# Testing Guide — Three-Tier To-Do Application

This document describes the testing and validation performed for the Three-Tier To-Do Application in the local Docker environment and after deployment on AWS ECS Fargate.

---

## 1. Testing Objectives

The main objectives of testing are:

- Verify that all application containers build and run correctly.
- Verify frontend availability.
- Verify backend API functionality.
- Verify MongoDB connectivity and data persistence.
- Verify Nginx reverse proxy functionality.
- Verify AWS Cloud Map service discovery.
- Verify ECS services and tasks.
- Verify frontend-to-backend network connectivity.
- Verify end-to-end application functionality.
- Verify GitHub Actions CI/CD deployment.
- Verify GitHub OIDC authentication.

---

## 2. Testing Environments

### Local Environment

The application was tested locally using Docker Compose.

The local architecture is:

    Browser
       ↓
    Frontend / Nginx :80
       ↓
    Backend / Node.js :5000
       ↓
    MongoDB :27017

### AWS Environment

The deployed architecture is:

    Browser
       ↓
    ECS Fargate Frontend / Nginx
       ↓
    AWS Cloud Map
       ↓
    ECS Fargate Backend / Node.js
       ↓
    AWS Cloud Map
       ↓
    ECS Fargate MongoDB

---

## 3. Docker Build Testing

The application containers were built using Docker Compose.

Command:

    docker compose build

### Expected Result

The frontend and backend Docker images should build successfully without errors.

### Result

**PASS**

The Docker images were successfully built.

---

## 4. Docker Container Testing

The application was started using:

    docker compose up -d

Running containers were checked using:

    docker ps

The required services are:

    frontend
    backend
    mongodb

### Expected Result

All required containers should be in the running state.

### Result

**PASS**

---

## 5. Docker Logs Testing

Application logs were checked using:

    docker compose logs

Individual services can be checked using:

    docker compose logs frontend

    docker compose logs backend

    docker compose logs mongodb

### Expected Result

The containers should start without application-level startup failures.

### Result

**PASS**

---

## 6. Frontend Testing

The local frontend was accessed through:

    http://localhost

### Test

The following were verified:

- Application page loads.
- To-Do interface is displayed.
- Task input is available.
- Existing tasks are displayed.
- Add Task functionality is available.
- Complete Task functionality is available.

### Result

**PASS**

---

## 7. Backend API Testing

The backend runs on port:

    5000

The task retrieval API was tested using:

    curl http://localhost:5000/tasks

### Expected Result

The backend should return the stored tasks successfully.

### Result

**PASS**

---

## 8. Backend API Endpoints Tested

The application provides the following endpoints:

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/tasks` | Retrieve tasks |
| POST | `/tasks` | Create a task |
| PUT | `/tasks/:id` | Update/complete a task |

All required API operations were tested through the application.

---

## 9. MongoDB Connectivity Testing

For the local Docker environment, the backend connects to MongoDB using:

    mongodb://mongodb:27017/todoDB

The database name is:

    todoDB

MongoDB connectivity was verified through application operations.

### Verification Process

    Create Task
        ↓
    Backend
        ↓
    MongoDB
        ↓
    Store Task
        ↓
    Retrieve Task

### Result

**PASS**

The backend successfully communicated with MongoDB.

---

## 10. Local Functional Testing

### Test Case 1 — Application Access

**Action:** Open `http://localhost`.

**Expected Result:** The To-Do application should load.

**Result:** PASS

---

### Test Case 2 — Add Task

**Action:** Enter a new task and click the Add Task button.

**Expected Result:** The new task should appear in the task list.

**Result:** PASS

---

### Test Case 3 — Complete Task

**Action:** Click the Complete Task button.

**Expected Result:** The selected task should be marked as completed.

**Result:** PASS

---

### Test Case 4 — Refresh Page

**Action:** Refresh the browser.

**Expected Result:** Previously stored tasks should still be available.

**Result:** PASS

---

## 11. Docker Network Testing

The Docker Compose services communicate through the Docker network.

The communication flow is:

    Frontend
       ↓
    Backend
       ↓
    MongoDB

The backend uses the Docker service name:

    mongodb

instead of `localhost`.

### Result

**PASS**

The application components successfully communicated through the Docker network.

---

## 12. AWS ECS Testing

The application was deployed using Amazon ECS Fargate.

### ECS Cluster

    three-tier-cluster

### ECS Services

    three-tier-frontend-service
    three-tier-backend-service
    three-tier-mongodb-service-2

---

## 13. ECS Service Status Testing

Each ECS service was checked to confirm that its task was running.

Expected state:

    RUNNING

The following services were verified:

- `three-tier-frontend-service`
- `three-tier-backend-service`
- `three-tier-mongodb-service-2`

### Result

**PASS**

---

## 14. ECS Task Configuration Testing

The ECS tasks were checked for:

- Correct task definition.
- Correct container image.
- Correct port configuration.
- Correct network configuration.
- Correct security group.
- Correct service configuration.
- Running task status.

Required application ports are:

    80      → Frontend
    5000    → Backend
    27017   → MongoDB

### Result

**PASS**

---

## 15. Amazon ECR Testing

The project uses the following ECR repositories:

    three-tier-frontend
    three-tier-backend
    three-tier-mongodb

The frontend and backend Docker images were successfully pushed to Amazon ECR.

### Result

**PASS**

---

## 16. AWS Cloud Map Testing

The private Cloud Map namespace is:

    three-tier.local

The application services include:

    backend
    mongodb

The service names used by the application are:

    backend.three-tier.local
    mongodb.three-tier.local

Cloud Map was tested to verify that ECS services could be discovered using service names instead of fixed task IP addresses.

### Result

**PASS**

---

## 17. Backend Service Discovery Testing

The backend ECS service was registered with AWS Cloud Map.

The backend service was discovered using:

    backend.three-tier.local

The Cloud Map discovery process returned the private IP address of the running backend task.

### Result

**PASS**

This confirmed that the backend service was successfully registered and discoverable.

---

## 18. MongoDB Service Discovery Testing

MongoDB was registered in the same Cloud Map namespace.

The expected service name is:

    mongodb.three-tier.local

The backend uses this service name to communicate with MongoDB in the AWS environment.

Successful task creation and retrieval confirmed that the backend could communicate with MongoDB.

### Result

**PASS**

---

## 19. Nginx Reverse Proxy Testing

Nginx is responsible for forwarding frontend API requests to the backend.

The browser sends:

    /api/tasks

Nginx forwards the request to:

    backend.three-tier.local:5000

The request flow is:

    Browser
       ↓
    Nginx :80
       ↓
    /api/tasks
       ↓
    backend.three-tier.local:5000
       ↓
    Node.js Backend

### Result

**PASS**

The frontend successfully communicated with the backend through the Nginx reverse proxy.

---

## 20. Frontend-to-Backend Network Testing

AWS Reachability Analyzer was used to verify the network path between the frontend and backend ECS task network interfaces.

The tested protocol was:

    TCP

The tested port was:

    5000

The Reachability Analyzer result was successful.

### Result

**PASS**

This confirmed that the frontend task could reach the backend task over TCP port `5000`.

---

## 21. Backend API Testing on AWS

The deployed backend API was tested using:

    GET /tasks

The API successfully returned:

    HTTP 200 OK

with task data.

This confirmed that:

- The backend ECS task was running.
- The Node.js application was responding.
- The API endpoint was working.
- The backend could communicate with MongoDB.

### Result

**PASS**

---

## 22. Frontend API Testing on AWS

The deployed frontend API endpoint was tested using:

    /api/tasks

The request successfully passed through Nginx and reached the backend.

The complete flow was:

    Browser
       ↓
    Frontend ECS Task
       ↓
    Nginx
       ↓
    backend.three-tier.local
       ↓
    Backend ECS Task
       ↓
    MongoDB

### Result

**PASS**

---

## 23. End-to-End Testing

The complete application was tested through the browser.

The complete request flow is:

    User
      ↓
    Frontend
      ↓
    Nginx
      ↓
    Backend
      ↓
    MongoDB
      ↓
    Backend
      ↓
    Nginx
      ↓
    Frontend
      ↓
    User

This confirms that all three application tiers communicate successfully.

### Result

**PASS**

---

## 24. AWS Functional Test Cases

### Test Case 1 — Access Deployed Application

**Action:** Open the frontend public IP in a browser.

    http://<FRONTEND_PUBLIC_IP>

**Expected Result:** The To-Do application should load.

**Result:** PASS

---

### Test Case 2 — Add Task

**Action:** Enter a task and submit it through the deployed application.

**Expected Result:** The task should appear in the task list.

**Result:** PASS

---

### Test Case 3 — Complete Task

**Action:** Click the Complete Task button.

**Expected Result:** The task should be marked as completed.

**Result:** PASS

---

### Test Case 4 — Refresh Application

**Action:** Refresh the browser after creating a task.

**Expected Result:** The task should remain available.

**Result:** PASS

---

## 25. Data Persistence Testing

Data persistence was tested by creating a task and refreshing the application.

The verification flow was:

    Create Task
        ↓
    Backend
        ↓
    MongoDB
        ↓
    Refresh Browser
        ↓
    GET /tasks
        ↓
    MongoDB
        ↓
    Task Returned

The created task remained available after refreshing the page.

### Result

**PASS**

This confirmed successful data persistence in MongoDB.

---

## 26. Security Group Testing

The application security group was checked for the required ports.

    80      → Frontend
    5000    → Backend
    27017   → MongoDB

MongoDB communication is restricted to the application security group.

### Result

**PASS**

The required application networking was available.

---

## 27. GitHub Actions Testing

The CI/CD workflow was tested by pushing changes to the `main` branch.

The workflow performs:

    Git Push
       ↓
    GitHub Actions
       ↓
    AWS OIDC Authentication
       ↓
    ECR Login
       ↓
    Build Backend
       ↓
    Push Backend Image
       ↓
    Build Frontend
       ↓
    Push Frontend Image
       ↓
    Force ECS Backend Deployment
       ↓
    Force ECS Frontend Deployment

### Result

**PASS**

The GitHub Actions workflow completed successfully.

---

## 28. GitHub OIDC Testing

GitHub Actions uses the IAM role:

    GitHubActionsThreeTierDeploy

The workflow authenticates using:

    sts:AssumeRoleWithWebIdentity

Successful authentication verified:

- GitHub OIDC provider configuration.
- IAM trust policy.
- GitHub Actions AWS authentication.
- Required deployment permissions.

### Result

**PASS**

---

## 29. CI/CD Deployment Verification

After a successful GitHub Actions workflow, the following were verified:

1. GitHub Actions workflow completed successfully.
2. Docker images were pushed to ECR.
3. ECS backend deployment was triggered.
4. ECS frontend deployment was triggered.
5. ECS tasks entered the running state.
6. Frontend application remained accessible.
7. API requests worked.
8. Task creation worked.
9. Task completion worked.
10. Data persisted after refresh.

### Result

**PASS**

---

## 30. Troubleshooting Validation

Previously encountered deployment issues were retested after their fixes.

### MongoDB Container Issue

The MongoDB container compatibility issue was resolved using:

    GLIBC_TUNABLES: "glibc.pthread.rseq=1"

MongoDB then remained available for the application.

### Nginx Reverse Proxy Issue

Nginx was configured to forward:

    /api/*

to:

    backend.three-tier.local:5000

The deployed frontend was then able to communicate with the backend.

### Cloud Map Service Discovery

Backend service discovery was verified using:

    backend.three-tier.local

MongoDB service discovery was configured using:

    mongodb.three-tier.local

### Network Connectivity

AWS Reachability Analyzer confirmed successful frontend-to-backend connectivity over:

    TCP :5000

### Result

**PASS**

---

## 31. Test Summary

| Test Area | Expected Result | Status |
|---|---|---|
| Docker Build | Images build successfully | PASS |
| Docker Compose | Services start successfully | PASS |
| Frontend | Application loads | PASS |
| Backend API | API responds successfully | PASS |
| MongoDB | Database communication works | PASS |
| Add Task | Task is created | PASS |
| Complete Task | Task is updated | PASS |
| Refresh | Data remains available | PASS |
| ECR | Images are available | PASS |
| ECS | Tasks are running | PASS |
| Cloud Map | Services are discoverable | PASS |
| Nginx | API requests are forwarded | PASS |
| Network Connectivity | TCP port 5000 is reachable | PASS |
| AWS Deployment | Application is accessible | PASS |
| GitHub Actions | Workflow completes successfully | PASS |
| GitHub OIDC | AWS authentication succeeds | PASS |
| End-to-End Flow | All three tiers communicate | PASS |
| Data Persistence | Tasks remain after refresh | PASS |

---

## 32. Final Verification

The final working application architecture was verified as:

    User
      ↓
    ECS Fargate Frontend
      ↓
    Nginx Reverse Proxy
      ↓
    AWS Cloud Map
      ↓
    ECS Fargate Backend
      ↓
    AWS Cloud Map
      ↓
    ECS Fargate MongoDB

The application successfully supports:

- Frontend access.
- Backend API requests.
- Task creation.
- Task completion.
- MongoDB storage.
- Data persistence.
- Cloud Map service discovery.
- Nginx reverse proxy.
- ECS Fargate deployment.
- GitHub Actions CI/CD.
- GitHub OIDC authentication.

---

## 33. Final Result

Testing confirmed that the **Three-Tier To-Do Application** works successfully in both the local Docker environment and the AWS ECS Fargate environment.

All major functional, networking, service discovery, persistence, deployment, and CI/CD tests were successfully completed.

The final verified flow is:

    Browser
       ↓
    Frontend / Nginx
       ↓
    Backend / Node.js
       ↓
    MongoDB

The application successfully performs the complete flow from user interaction to database storage and back to the user interface.
