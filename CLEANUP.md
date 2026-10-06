# Cleanup Guide — Three-Tier To-Do Application

## 1. Purpose

This document describes the cleanup steps for the Three-Tier To-Do Application after testing and deployment.

The cleanup process removes the AWS resources created for the project to avoid unnecessary resource usage and charges.

---

## 2. Cleanup Overview

The project uses the following AWS resources:

- Amazon ECS Cluster
- ECS Frontend Service
- ECS Backend Service
- ECS MongoDB Service
- ECS Tasks
- Amazon ECR Repositories
- AWS Cloud Map Namespace and Services
- Security Group
- VPC networking resources

Before deleting resources, make sure that the application is no longer required.

---

## 3. Stop ECS Services

First, scale the ECS services down to zero desired tasks.

### Frontend

    aws ecs update-service \
      --cluster three-tier-cluster \
      --service three-tier-frontend-service \
      --desired-count 0

### Backend

    aws ecs update-service \
      --cluster three-tier-cluster \
      --service three-tier-backend-service \
      --desired-count 0

### MongoDB

    aws ecs update-service \
      --cluster three-tier-cluster \
      --service three-tier-mongodb-service-2 \
      --desired-count 0

Verify that the tasks have stopped:

    aws ecs list-tasks \
      --cluster three-tier-cluster

---

## 4. Delete ECS Services

After the tasks have stopped, delete the ECS services.

### Frontend

    aws ecs delete-service \
      --cluster three-tier-cluster \
      --service three-tier-frontend-service

### Backend

    aws ecs delete-service \
      --cluster three-tier-cluster \
      --service three-tier-backend-service

### MongoDB

    aws ecs delete-service \
      --cluster three-tier-cluster \
      --service three-tier-mongodb-service-2

---

## 5. Delete ECS Cluster

After all ECS services have been deleted, delete the ECS cluster.

    aws ecs delete-cluster \
      --cluster three-tier-cluster

Verify the cluster:

    aws ecs describe-clusters \
      --clusters three-tier-cluster

---

## 6. Clean Up Amazon ECR

The project uses these ECR repositories:

    three-tier-frontend
    three-tier-backend
    three-tier-mongodb

To delete a repository and all images inside it:

    aws ecr delete-repository \
      --repository-name three-tier-frontend \
      --force

    aws ecr delete-repository \
      --repository-name three-tier-backend \
      --force

    aws ecr delete-repository \
      --repository-name three-tier-mongodb \
      --force

Verify the repositories:

    aws ecr describe-repositories

---

## 7. Clean Up AWS Cloud Map

The project uses the private namespace:

    three-tier.local

The Cloud Map services include:

    backend
    mongodb

Before deleting the namespace, remove the registered Cloud Map services.

The Cloud Map namespace and services can be removed from:

    AWS Console
        ↓
    AWS Cloud Map
        ↓
    Namespaces
        ↓
    three-tier.local

Delete the services associated with the namespace and then delete the namespace.

---

## 8. Clean Up Security Group

The application security group is:

    sg-012215fb155d043a5

Before deleting the security group:

- Ensure no ECS task or network interface is still using it.
- Ensure there are no dependent AWS resources.

After all dependent resources are removed, the security group can be deleted from the Amazon VPC console.

---

## 9. Clean Up Networking Resources

The project used the VPC:

    vpc-03237cb2bc44a1174

The project also used:

    subnet-08a1275921516ed82
    subnet-00bd0215a8ab1068c

Do not delete the VPC or subnets if they are shared with other applications or AWS resources.

If the VPC was created specifically for this project and is no longer required, its dependent resources should be removed before deleting the VPC.

---

## 10. Local Docker Cleanup

After AWS cleanup, local Docker resources can also be removed if they are no longer required.

Stop the local application:

    docker compose down

To also remove the locally created images:

    docker compose down --rmi local

To remove unused Docker resources:

    docker system prune

Review the Docker resources before confirming the prune operation.

---

## 11. Local Project Files

The GitHub repository and local source code do not need to be deleted as part of AWS cleanup.

The following project files can be retained for future use:

    backend/
    frontend/
    docker-compose.yml
    Jenkinsfile
    README.md
    ARCHITECTURE.md
    SETUP.md
    COMMANDS.md
    TESTING.md
    VERIFICATION.md
    TROUBLESHOOTING.md
    CLEANUP.md
    architecture.png
    screenshots/

Keeping the source code allows the application to be redeployed later.

---

## 12. GitHub Actions Cleanup

The GitHub Actions workflow uses the AWS IAM role:

    GitHubActionsThreeTierDeploy

If CI/CD is no longer required, the GitHub Actions workflow and its AWS OIDC configuration can be removed.

Before removing the IAM role, verify that no other GitHub repository or workflow is using it.

The workflow file is located at:

    .github/workflows/deploy.yml

---

## 13. Final Cleanup Checklist

| Resource | Cleanup Action | Status |
|---|---|---|
| ECS Frontend Service | Delete service | CHECK |
| ECS Backend Service | Delete service | CHECK |
| ECS MongoDB Service | Delete service | CHECK |
| ECS Tasks | Stop tasks | CHECK |
| ECS Cluster | Delete cluster | CHECK |
| ECR Frontend | Delete repository | CHECK |
| ECR Backend | Delete repository | CHECK |
| ECR MongoDB | Delete repository | CHECK |
| Cloud Map Backend Service | Delete service | CHECK |
| Cloud Map MongoDB Service | Delete service | CHECK |
| Cloud Map Namespace | Delete namespace | CHECK |
| Security Group | Delete if unused | CHECK |
| VPC/Subnets | Delete only if project-specific | CHECK |
| Local Docker Containers | Stop/remove | CHECK |
| GitHub OIDC Role | Remove if no longer required | OPTIONAL |
| GitHub Repository | Retain source code | RETAIN |

---

## 14. Important Notes

### MongoDB Data

The MongoDB container used for this project is part of the ECS deployment.

Before removing the MongoDB resources, make sure that any required application data has been backed up.

### AWS Billing

AWS resources may continue to generate charges while they are running or retaining billable resources.

After cleanup, verify the AWS Console to ensure that the project resources have been removed and that no unwanted resources remain.

### Shared Resources

Do not delete shared VPCs, subnets, IAM roles, security groups, or other AWS resources if they are being used by another project.

---

## 15. Final Cleanup Verification

After cleanup, verify the following:

    ECS Services → Deleted
    ECS Tasks → Stopped
    ECS Cluster → Deleted
    ECR Repositories → Deleted
    Cloud Map Services → Deleted
    Cloud Map Namespace → Deleted
    Unused Security Group → Deleted
    Project-Specific Networking → Removed if applicable
    Local Docker Resources → Cleaned if required

The GitHub repository and project documentation can be retained for future deployment and reference.

---

## 16. Cleanup Completion

Once the above resources have been removed and verified, the AWS deployment resources for the Three-Tier To-Do Application are cleaned up.

**Cleanup Status: COMPLETED AFTER RESOURCE VERIFICATION**
