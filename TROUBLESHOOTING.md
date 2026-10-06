# Troubleshooting Guide — Three-Tier To-Do Application

## 1. Purpose

This document records the main issues encountered during the development and AWS deployment of the Three-Tier To-Do Application, along with the solutions that were used to resolve them.

The troubleshooting steps are based on the actual issues encountered during the project.

---

# 2. SSH Permission Denied While Connecting to EC2

## Problem

While initially working with AWS EC2, SSH access returned a permission denied error.

## Cause

The SSH private key permissions were not configured correctly.

## Solution

The key file permissions were corrected and the SSH connection was retried.

After correcting the permissions, SSH access worked successfully.

## Result

**RESOLVED**

---

# 3. Docker Not Installed

## Problem

Docker commands were not available on the Ubuntu EC2 instance.

## Error

    docker: command not found

## Solution

Docker was installed using:

    sudo apt update

    sudo apt install docker.io -y

Docker installation was then verified using:

    docker --version

## Result

**RESOLVED**

---

# 4. Docker Compose Plugin Not Available

## Problem

The Docker Compose plugin was not available directly through the default Ubuntu package repository.

## Solution

The official Docker repository was added and the Docker Compose plugin was installed.

The installation was verified using:

    docker compose version

## Result

**RESOLVED**

---

# 5. MongoDB Container Exited

## Problem

The MongoDB container exited unexpectedly during deployment.

## Cause

A compatibility issue related to glibc pthread rseq behavior was encountered.

## Solution

The MongoDB service was configured with:

    GLIBC_TUNABLES: "glibc.pthread.rseq=1"

After rebuilding and restarting the containers, MongoDB remained running.

## Result

**RESOLVED**

---

# 6. Backend Could Not Connect to MongoDB

## Problem

The backend was unable to communicate correctly with MongoDB during deployment.

## Cause

The application needed to use the MongoDB service name instead of relying on `localhost` or a fixed IP address.

## Local Solution

The local Docker Compose configuration uses:

    MONGO_URI=mongodb://mongodb:27017/todoDB

Here, `mongodb` is the Docker Compose service name.

## AWS Solution

AWS Cloud Map was configured so that MongoDB could be discovered using:

    mongodb.three-tier.local

## Result

**RESOLVED**

---

# 7. Frontend Could Not Communicate with Backend

## Problem

The frontend loaded successfully, but API operations such as adding or completing tasks did not work after AWS deployment.

## Cause

The frontend needed to communicate with the backend through the deployed Nginx reverse proxy.

The frontend could not simply use the backend container's localhost address because the frontend and backend were running in separate ECS tasks.

## Solution

Nginx was configured to forward:

    /api/

to:

    backend.three-tier.local:5000

AWS Cloud Map was used to resolve the backend service name.

## Result

**RESOLVED**

---

# 8. Nginx Reverse Proxy Configuration Issue

## Problem

The frontend was accessible, but API requests were not correctly reaching the backend.

## Cause

The Nginx reverse proxy configuration needed to correctly remove the `/api/` prefix before forwarding the request to the backend.

For example:

    /api/tasks

needed to become:

    /tasks

at the backend.

## Solution

The Nginx configuration was updated to use:

    rewrite ^/api/(.*)$ /$1 break;

and then forward the request to:

    backend.three-tier.local:5000

## Result

The API request:

    /api/tasks

successfully reached:

    /tasks

on the backend.

**RESOLVED**

---

# 9. Cloud Map Backend Service Discovery Issue

## Problem

The frontend Nginx container needed to resolve the backend ECS service dynamically.

## Cause

The backend task IP address can change when ECS replaces or restarts a task.

Using a fixed private IP would therefore not be reliable.

## Solution

AWS Cloud Map was configured with the private namespace:

    three-tier.local

The backend service was registered as:

    backend

The frontend Nginx configuration uses:

    backend.three-tier.local

for backend communication.

## Verification

Backend service discovery was tested using:

    aws servicediscovery discover-instances \
      --namespace-name three-tier.local \
      --service-name backend

The command successfully returned the running backend task information.

## Result

**RESOLVED**

---

# 10. Frontend-to-Backend Network Connectivity Issue

## Problem

It was necessary to verify whether the frontend ECS task could actually reach the backend ECS task on port `5000`.

## Solution

AWS Reachability Analyzer was used.

The tested path was:

    Frontend ECS ENI
        ↓
    Backend ECS ENI

Protocol:

    TCP

Port:

    5000

The Reachability Analyzer analysis returned:

    succeeded: true

This confirmed that the network path was reachable.

## Result

**RESOLVED**

---

# 11. ECS Exec Could Not Be Enabled

## Problem

ECS Exec was attempted for troubleshooting, but ECS returned an error indicating that a valid task role was required.

## Error

    The service couldn't be updated because a valid taskRoleArn is not being used.

## Cause

The frontend ECS task definition did not have a task role configured.

The task definition had an execution role, but no `taskRoleArn`.

## Resolution

ECS Exec was not required for the final application because network connectivity and application behavior could be verified using other methods such as:

- AWS Reachability Analyzer
- Cloud Map discovery
- API testing
- Browser testing

## Result

**Not required for final deployment**

---

# 12. ECS Tasks Started and Then Stopped

## Problem

During the ECS deployment process, some tasks started and then stopped.

## Troubleshooting Approach

The ECS service and task status were checked through the ECS console and logs.

The application configuration was checked for:

- Container configuration
- Environment variables
- Port mappings
- Network configuration
- MongoDB connectivity
- Service discovery

After correcting the required configuration, the services were able to remain running.

## Result

**RESOLVED**

---

# 13. Mongoose Buffering Timeout

## Problem

The backend produced a Mongoose error similar to:

    MongooseError: Operation `tasks.find()` buffering timed out after 10000ms

## Cause

The backend could not successfully communicate with MongoDB.

This was related to the MongoDB connection configuration and ECS networking/service discovery.

## Solution

The MongoDB connection was configured to use the correct service discovery address rather than an unavailable hostname or fixed IP.

The backend was then able to communicate with the MongoDB service.

## Result

**RESOLVED**

---

# 14. Frontend Browser Timeout

## Problem

The deployed frontend initially returned:

    ERR_CONNECTION_TIMED_OUT

## Troubleshooting

The following were checked:

- ECS service status
- ECS task status
- Public IP address
- Security group
- Port `80`
- VPC networking
- Task network interface

The security group and ECS networking were corrected/verified.

## Result

The frontend became accessible through its public IP.

**RESOLVED**

---

# 15. Git Push Rejected Because Remote Was Ahead

## Problem

A Git push was rejected because the remote `main` branch contained changes that were not present locally.

## Cause

The local branch was behind the GitHub repository.

## Solution

The remote changes were first pulled using rebase:

    git pull origin main --rebase

After the local branch was updated, the changes were pushed again:

    git push origin main

The push then completed successfully.

## Result

**RESOLVED**

---

# 16. GitHub Actions OIDC Authentication Issue

## Problem

GitHub Actions initially had an issue assuming the AWS IAM role using OIDC.

## Cause

The IAM trust policy needed to correctly restrict and match the GitHub repository and branch identity.

## Solution

The GitHub OIDC provider was configured with:

    https://token.actions.githubusercontent.com

The IAM role used was:

    GitHubActionsThreeTierDeploy

The trust policy was updated to use the correct GitHub OIDC audience and subject.

After the correction, GitHub Actions successfully authenticated with AWS.

The successful workflow log showed:

    Assuming role with OIDC
    Authenticated as assumedRoleId

## Result

**RESOLVED**

---

# 17. GitHub Actions Deployment Issue

## Problem

The application needed to automatically deploy updated frontend and backend images after changes were pushed to GitHub.

## Solution

The GitHub Actions workflow was configured to:

1. Checkout the repository.
2. Authenticate with AWS using OIDC.
3. Login to Amazon ECR.
4. Build the backend Docker image.
5. Push the backend image to ECR.
6. Build the frontend Docker image.
7. Push the frontend image to ECR.
8. Force a new ECS backend deployment.
9. Force a new ECS frontend deployment.

The workflow was then successfully executed.

## Result

**RESOLVED**

---

# 18. Final Nginx / Cloud Map Fix

## Problem

The application frontend was loading, but the deployed API request was not initially reaching the backend correctly.

## Solution

The final Nginx configuration was updated to:

- Use the AWS VPC DNS resolver.
- Resolve `backend.three-tier.local`.
- Remove the `/api/` prefix before forwarding.
- Proxy requests to backend port `5000`.

The final configuration was committed to GitHub.

Final commit:

    7193448 — Update nginx.conf

The GitHub Actions deployment then completed successfully.

## Result

**RESOLVED**

---

# 19. Final API Verification

After the Nginx and Cloud Map fixes, the deployed frontend API was tested using:

    curl http://<FRONTEND_PUBLIC_IP>/api/tasks

The request returned:

    HTTP 200 OK

Task data was successfully returned.

This confirmed that the following components were working together:

    Frontend
       ↓
    Nginx
       ↓
    Cloud Map
       ↓
    Backend
       ↓
    MongoDB

## Result

**VERIFIED**

---

# 20. Task Add / Complete Button Issue

## Problem

During an earlier deployment, the application page loaded but the Add Task / Complete Task functionality did not work correctly.

## Cause

The frontend API request was not correctly reaching the backend through the Nginx reverse proxy.

## Solution

The Nginx `/api/` proxy configuration and Cloud Map backend service discovery were corrected.

After the fix:

- Add Task worked.
- Complete Task worked.
- Refresh preserved the task data.

## Result

**RESOLVED**

---

# 21. Troubleshooting Summary

| Issue | Solution | Status |
|---|---|---|
| SSH permission denied | Corrected SSH key permissions | RESOLVED |
| Docker not installed | Installed Docker | RESOLVED |
| Docker Compose unavailable | Added Docker repository and installed plugin | RESOLVED |
| MongoDB container exited | Added glibc rseq configuration | RESOLVED |
| MongoDB connection issue | Corrected service-based connection | RESOLVED |
| Frontend API not working | Configured Nginx reverse proxy | RESOLVED |
| Nginx API path issue | Added `/api/` rewrite | RESOLVED |
| Cloud Map discovery | Registered backend service | RESOLVED |
| Frontend → Backend connectivity | Verified using Reachability Analyzer | VERIFIED |
| ECS Exec unavailable | Not required for final deployment | NOT REQUIRED |
| ECS tasks stopping | Checked configuration and logs | RESOLVED |
| Mongoose timeout | Corrected MongoDB connectivity | RESOLVED |
| Browser timeout | Verified ECS, IP, SG and networking | RESOLVED |
| Git push rejected | Used `git pull --rebase` | RESOLVED |
| GitHub OIDC issue | Corrected IAM trust policy | RESOLVED |
| CI/CD deployment | Configured GitHub Actions workflow | RESOLVED |
| Add/Complete Task issue | Corrected Nginx and Cloud Map routing | RESOLVED |

---

# 22. Final Troubleshooting Result

After resolving the above issues, the final application successfully achieved:

- Local Docker Compose deployment.
- Working frontend.
- Working backend API.
- Working MongoDB connection.
- ECS Fargate deployment.
- ECR image deployment.
- AWS Cloud Map service discovery.
- Nginx reverse proxy.
- Frontend-to-backend connectivity.
- Backend-to-MongoDB connectivity.
- Successful API response.
- Add Task functionality.
- Complete Task functionality.
- Data persistence after refresh.
- Successful GitHub Actions deployment.
- Successful GitHub OIDC authentication.

**Final Status: ALL MAJOR ENCOUNTERED ISSUES RESOLVED**
