# 🎙️ Highly Available Text-to-Speech App on AWS

A .NET 9 ASP.NET Core web application deployed on AWS with a production-grade, highly available architecture. The app converts text to speech in multiple languages using **AWS Polly** and detects language using **AWS Comprehend**, served through an **Application Load Balancer** across two Availability Zones.

---

## 📺 Demo

<!-- Replace with your actual YouTube/Google Drive video link -->
[![Watch Demo](https://img.shields.io/badge/Watch-Demo%20Video-red?style=for-the-badge&logo=youtube)](https://your-video-link-here)

---

## 🏗️ Architecture

```
Internet
    ↓
Application Load Balancer
(public subnets — AZ-a & AZ-b)
    ↓
Auto Scaling Group
├── EC2 Instance (AZ-a) ← .NET 9 App on port 5000
└── EC2 Instance (AZ-b) ← .NET 9 App on port 5000
        ↓
IAM Role (least-privilege)
├── AWS Polly        → Text-to-Speech
├── AWS Comprehend   → Language Detection
└── S3               → App Artifact Storage
```

<!-- Replace with your architecture diagram image -->
![Architecture Diagram](screenshots/architecture-diagram.png)

---

## 🛠️ AWS Services Used

| Service | Purpose |
|---|---|
| VPC | Custom network with public subnets across 2 AZs |
| EC2 | Application servers running the .NET 9 app |
| Application Load Balancer | Distributes traffic, single entry point |
| Auto Scaling Group | Self-healing, maintains desired capacity |
| S3 | Stores app zip for automated deployment |
| IAM | Role-based access, zero hardcoded credentials |
| AWS Polly | Neural text-to-speech in multiple languages |
| AWS Comprehend | NLP and language detection |

---

## ✨ Key Features

- **High Availability** — Deployed across 2 AZs, no single point of failure
- **Self-Healing** — ASG automatically replaces unhealthy instances based on ELB health checks
- **Auto Scaling** — Scales out when CPU exceeds 50%, scales in when load drops
- **Secure** — IAM role-based authentication, no hardcoded AWS credentials, least-privilege access
- **Automated Deployment** — App pulled from S3 and started automatically via EC2 User Data on every instance launch

---

## 📸 Screenshots

### App Running via ALB DNS
<!-- Replace with your screenshot -->
![App via ALB](screenshots/app-running-alb.png)

### Both Instances Healthy in Target Group
<!-- Replace with your screenshot -->
![Healthy Instances](screenshots/target-group-healthy.png)

### Auto Scaling Group — 2 Instances InService
<!-- Replace with your screenshot -->
![ASG Instances](screenshots/asg-instances.png)

### Load Balancer Active
<!-- Replace with your screenshot -->
![Load Balancer](screenshots/alb-active.png)

### App Running on EC2 (systemctl status)
<!-- Replace with your screenshot -->
![Systemctl Status](screenshots/systemctl-running.png)

### S3 Bucket with App Artifact
<!-- Replace with your screenshot -->
![S3 Bucket](screenshots/s3-bucket.png)

### IAM Role with Least-Privilege Policies
<!-- Replace with your screenshot -->
![IAM Role](screenshots/iam-role.png)

---

## 🚀 Deployment

See the full step-by-step deployment guide here → [deployment-guide.md](docs/deployment-guide.md)

### Quick Summary
1. Upload app zip to S3
2. Create VPC with public subnets across 2 AZs
3. Create IAM role with Polly, Comprehend, S3 access
4. Create Security Groups for ALB and EC2
5. Create Launch Template with User Data script
6. Create Target Group on port 5000
7. Create Application Load Balancer
8. Create Auto Scaling Group — instances register automatically

---

## 🔑 Key Technical Decisions

**Why public subnets instead of private + NAT Gateway?**
For this learning project, public subnets with restricted security groups were used to avoid NAT Gateway costs. In production, EC2 instances would sit in private subnets behind a NAT Gateway.

**Why User Data instead of Elastic Beanstalk?**
Elastic Beanstalk automates what we built manually — ALB, ASG, EC2, health checks. Building it from scratch demonstrates understanding of each component rather than treating it as a black box.

**Why IAM Role instead of access keys?**
IAM roles follow AWS security best practices. Credentials are temporary, automatically rotated, and never stored on the instance. No risk of accidentally exposing keys in code or config files.

---

## 🐛 Issues Encountered & Solved

Real debugging experience from this deployment:

- **Wrong .NET runtime** — `dotnet-runtime-9.0` doesn't include ASP.NET Core. Fixed by installing `aspnetcore-runtime-9.0`
- **Dotnet binary path** — On Amazon Linux 2023, dotnet is at `/usr/lib64/dotnet/dotnet` not `/usr/bin/dotnet`
- **No internet on instances** — Forgot to enable auto-assign public IPv4 on public subnets
- **Browser HTTPS redirect** — ALB serves HTTP only, browser was auto-upgrading to HTTPS. Fixed by explicitly using `http://`
- **Wrong S3 file name** — File had spaces in name, renamed to `app.zip` for clean deployment

---

## 📁 Repository Structure

```
├── README.md
├── docs/
│   └── deployment-guide.md
├── userdata/
│   └── userdata.sh          ← EC2 User Data script (replace placeholders)
└── screenshots/
    ├── architecture-diagram.png
    ├── app-running-alb.png
    ├── target-group-healthy.png
    ├── asg-instances.png
    ├── alb-active.png
    ├── systemctl-running.png
    ├── s3-bucket.png
    └── iam-role.png
```

---

## 🧑‍💻 Author

**Uttkarsh**
- AWS Solutions Architect Associate (In Progress)
- CSE Student | Cloud & Infrastructure Enthusiast
- Japanese Language Learner (JLPT N5)

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
