---

# 📄 **ECS Deployment Guide – `iamtejas23/space` on AWS ECS (Fargate)**

## 📌 Objective

This document outlines the end-to-end process to deploy a Docker-based application on **AWS Elastic Container Service (ECS)** using **Fargate** launch type, with publicly accessible endpoints, monitoring through CloudWatch, and proper security configuration.

---

## 📦 Application Overview

* **Docker Image:** `iamtejas23/space`
* **Port:** 3000 (Application listens on this port internally)
* **Hosting:** AWS ECS Fargate (Serverless)
* **Logging:** AWS CloudWatch Logs

---

## 🛠️ Prerequisites

* AWS account with necessary permissions (IAM, ECS, EC2, VPC)
* AWS CLI installed and configured (optional for advanced operations)

---

## 1️⃣ Create ECS Cluster (Networking only – Fargate)

1. **Open ECS Console:**
   [https://console.aws.amazon.com/ecs](https://console.aws.amazon.com/ecs)

2. **Navigation:**

   * Go to **Clusters** → Click **Create Cluster**
   * Select **"Networking only" (Powered by AWS Fargate)** → Click **Next**

3. **Configure Cluster:**

   * Cluster name: `space-cluster`
   * Leave other settings as default
   * Click **Create**

> 🔁 If creation fails due to existing stack:
> Go to [CloudFormation](https://console.aws.amazon.com/cloudformation) → Delete stack named `Infra-ECS-Cluster-space-cluster-*`

---

## 2️⃣ Create IAM Role: `ecsTaskExecutionRole`

### a. Create IAM Role using AWS CLI:

```bash
aws iam create-role \
  --role-name ecsTaskExecutionRole \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "Service": "ecs-tasks.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }'
```

### b. Attach ECS Task Execution Policy:

```bash
aws iam attach-role-policy \
  --role-name ecsTaskExecutionRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```

---

## 3️⃣ Create Task Definition

1. Navigate to: **Task Definitions → Create new Task Definition**
2. Select **Fargate** → Click **Next**
3. Configure:

   * **Task name**: `space-task`
   * **Execution Role**: `ecsTaskExecutionRole`
   * **Operating System**: Linux
   * **Task memory/CPU**: `0.5GB / 0.25vCPU`
4. **Add Container**:

   * Container name: `space-app`
   * Image URI: `iamtejas23/space`
   * Port mapping: **3000**
   * Click **Add**
5. Click **Next → Create**

---

## 4️⃣ Create ECS Service

1. Go to **Clusters** → `space-cluster` → **Create Service**

2. Settings:

   * Launch Type: `Fargate`
   * Task Definition: `space-task:1`
   * Service name: `space-service`
   * Desired tasks: `1`
   * Click **Next**

3. **Configure Networking:**

   * Select **default VPC**
   * Choose **2 public subnets**
   * **Auto-assign public IP: ENABLED**
   * **Create new security group:**

     * Name: `space-sg`
     * Inbound rule:

       * Type: **Custom TCP**
       * Port: `3000`
       * Source: `0.0.0.0/0`

4. Click **Next → Create Service**

---

## 5️⃣ Access the Application

### Find Public IP:

1. Go to **Clusters → space-cluster → Tasks**
2. Click on the running **Task ID**
3. Scroll to **Network** section → Copy the **Public IP**

---

### Confirm Application is Running:

1. Go to **CloudWatch Logs** → Find log group for ECS → `space-app` container

2. You should see:

   ```
   INFO Accepting connections at http://localhost:3000
   ```

3. Visit the application:

   ```
   http://<public-ip>:3000
   ```

Example:

```
http://3.87.219.28:3000
```

---

## 6️⃣ (Optional) Troubleshooting

| Issue                  | Solution                                                         |
| ---------------------- | ---------------------------------------------------------------- |
| **No Public IP?**      | Ensure "Auto-assign public IP" is enabled and subnets are public |
| **App not loading?**   | Check Security Group allows inbound traffic on port 3000         |
| **Container crashes?** | View logs in **CloudWatch** for error messages                   |
| **Port mismatch?**     | Ensure ECS Task Definition maps **container port 3000**          |

---

## 🔒 Security Best Practices

* Restrict security group to specific IP ranges if possible
* Consider moving to **ALB** for production-grade access
* Use **Secrets Manager or SSM** for environment variables and credentials

---

## ✅ Summary

| Component           | Value                     |
| ------------------- | ------------------------- |
| **Image**           | `iamtejas23/space`        |
| **Port**            | 3000                      |
| **Cluster**         | `space-cluster`           |
| **Task Definition** | `space-task`              |
| **Service**         | `space-service`           |
| **Public IP**       | `http://3.87.219.28:3000` |
| **Logs**            | AWS CloudWatch            |

---


