# 🖥️ Realtime Code Editor — AWS Cloud Deployment

A collaborative, real-time code editor built with Monaco Editor and deployed on **AWS using ECS Fargate, ECR, and ALB** — fully containerized and production-ready on the cloud.

---

## ☁️ AWS Architecture

```
                        ┌─────────────────────────────────────────┐
                        │              AWS Cloud                   │
                        │           ap-northeast-1 (Tokyo)         │
                        │                                          │
  User Browser  ──────► │  Application Load Balancer (ALB)        │
                        │           │                              │
                        │           ▼                              │
                        │  ┌─────────────────────┐                │
                        │  │   ECS Cluster        │                │
                        │  │   (Fargate)          │                │
                        │  │                      │                │
                        │  │  ┌────────────────┐  │                │
                        │  │  │ Task Definition │  │                │
                        │  │  │ docker-aws:1    │  │                │
                        │  │  │ 1 vCPU / 3GB   │  │                │
                        │  │  └────────────────┘  │                │
                        │  └─────────────────────┘                │
                        │           │                              │
                        │           ▼                              │
                        │  ┌─────────────────────┐                │
                        │  │   Amazon ECR         │                │
                        │  │   Docker Registry    │                │
                        │  │   (62MB image)       │                │
                        │  └─────────────────────┘                │
                        │                                          │
                        │  VPC → Subnets → Security Groups        │
                        └─────────────────────────────────────────┘
```

---

## 🚀 AWS Services Used

| Service | Purpose | Details |
|---------|---------|---------|
| **Amazon ECR** | Docker image registry | Stored server image (62MB) |
| **Amazon ECS** | Container orchestration | Fargate serverless launch type |
| **AWS Fargate** | Serverless compute | No EC2 instances to manage |
| **Application Load Balancer** | Traffic distribution | HTTP routing to ECS tasks |
| **Amazon VPC** | Network isolation | Custom subnets & route tables |
| **Security Groups** | Firewall rules | Inbound/outbound traffic control |
| **IAM** | Access management | ECR & ECS permissions |

---

## 🐳 Docker + ECR Setup

### 1. Build the Docker Image
```bash
docker build -t docker-aws/server .
```

### 2. Authenticate with Amazon ECR
```bash
aws ecr get-login-password --region ap-northeast-1 | docker login \
  --username AWS \
  --password-stdin 908027414459.dkr.ecr.ap-northeast-1.amazonaws.com
```

### 3. Tag the Image
```bash
docker tag docker-aws/server:latest \
  908027414459.dkr.ecr.ap-northeast-1.amazonaws.com/docker-aws/server:latest
```

### 4. Push to ECR
```bash
docker push \
  908027414459.dkr.ecr.ap-northeast-1.amazonaws.com/docker-aws/server:latest
```

---

## ⚙️ ECS Fargate Deployment

### Task Definition Configuration

| Parameter | Value |
|-----------|-------|
| Launch Type | AWS Fargate |
| Task CPU | 1,024 units (1 vCPU) |
| Task Memory | 3,072 MiB (3 GB) |
| Network Mode | awsvpc |
| OS / Architecture | Linux / X86_64 |
| App Environment | Fargate |

### ECS Cluster
- **Cluster Name:** `smart-rhinoceros-ct8wvz`
- **Region:** `ap-northeast-1` (Tokyo)
- **Status:** Active

---

## 🔐 IAM Permissions Required

The IAM user needs the following AWS managed policies:

```
✅ AmazonEC2ContainerRegistryFullAccess
✅ AmazonECS_FullAccess
```

Minimum ECR permissions needed:
```json
{
  "Effect": "Allow",
  "Action": [
    "ecr:GetAuthorizationToken",
    "ecr:BatchCheckLayerAvailability",
    "ecr:PutImage",
    "ecr:InitiateLayerUpload",
    "ecr:UploadLayerPart",
    "ecr:CompleteLayerUpload"
  ],
  "Resource": "*"
}
```

---

## 🌐 Network Configuration

```
VPC
 ├── Public Subnet (ALB)
 │    └── Application Load Balancer
 │         └── Target Group → ECS Tasks
 └── Private Subnet (ECS)
      └── ECS Fargate Tasks
           └── Security Group (port 3000 inbound)
```

- **ALB DNS:** `docker-aws-new-alb-1529385460.ap-northeast-1.elb.amazonaws.com`
- **Security Groups:** Configured for HTTP traffic on application port

---

## 📸 AWS Console Screenshots

| Resource | Screenshot |
|----------|-----------|
| ECS Cluster (Active) | <img width="1920" height="1080" alt="Screenshot from 2026-04-13 10-23-20" src="https://github.com/user-attachments/assets/59ff8b84-5433-4606-bb8b-c05bd9f901bf" /> |
| Task Definition (docker-aws:1) | <img width="1920" height="1080" alt="Screenshot from 2026-04-13 10-25-33" src="https://github.com/user-attachments/assets/5220d5ca-31af-4985-ba42-9324084900cd" /> |

---

## ✨ App Features

- 🔴 **Real-time Collaboration** — Multiple users edit code simultaneously
- 📝 **Monaco Editor** — Same editor that powers VS Code
- 🔄 **Yjs CRDT** — Conflict-free document sync across all clients
- 🚪 **Room-based Sessions** — Isolated collaboration via unique room IDs
- 🌐 **Socket.io** — WebSocket communication between clients and server

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React, Monaco Editor |
| Backend | Node.js, Express |
| Real-time Sync | Socket.io, Yjs |
| Containerization | Docker |
| Registry | AWS ECR |
| Orchestration | AWS ECS (Fargate) |
| Load Balancing | AWS ALB |
| Networking | AWS VPC, Subnets, Security Groups |
| Auth & Access | AWS IAM |

---

### Run with Docker locally
```bash
docker build -t realtime-code-editor .
docker run -p 3000:3000 realtime-code-editor
```
