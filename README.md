# &#x20;Highly Available Text-to-Speech App on AWS

A .NET 9 ASP.NET Core web application deployed on AWS with a production-grade, highly available architecture. The app converts text to speech in multiple languages using **AWS Polly** and detects language using **AWS Comprehend**, served through an **Application Load Balancer** across two Availability Zones.

\---

## &#x20;App in Action

<!-- 
  HOW TO ADD YOUR GIF:
  1. Record screen with OBS
  2. Go to ezgif.com → Video to GIF → upload your recording → convert → download
  3. Save the file as app-demo.gif inside the videos/ folder
  4. Push to GitHub — it will play automatically in the README

!\[App Demo](videos/app-demo.gif)

\---

## &#x20;Architecture

<!-- Save your draw.io diagram as architecture-diagram.png inside screenshots/ folder -->

!\[Architecture Diagram](screenshots/architecture-diagram.png)

```
Internet
    ↓
Application Load Balancer
(public subnets — AZ-a \& AZ-b)
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

\---

## &#x20;AWS Services Used

|Service|Purpose|
|-|-|
|VPC|Custom network with public subnets across 2 AZs|
|EC2|Application servers running the .NET 9 app|
|Application Load Balancer|Distributes traffic, single entry point|
|Auto Scaling Group|Self-healing, maintains desired capacity|
|S3|Stores app zip for automated deployment|
|IAM|Role-based access, zero hardcoded credentials|
|AWS Polly|Neural text-to-speech in multiple languages|
|AWS Comprehend|NLP and language detection|

\---

## &#x20;Features

* **High Availability** — Deployed across 2 AZs, no single point of failure
* **Self-Healing** — ASG automatically replaces unhealthy instances based on ELB health checks
* **Auto Scaling** — Scales out when CPU exceeds 50%, scales in when load drops
* **Secure** — IAM role-based authentication, no hardcoded credentials, least-privilege access
* **Automated Deployment** — App pulled from S3 and started via EC2 User Data on every instance launch

\---

## &#x20;Infrastructure Screenshots

### Load Balancer — Active

<!-- Save as screenshots/alb-active.png -->

!\[ALB Active](screenshots/alb-active.png)

### Target Group — Both Instances Healthy

<!-- Save as screenshots/target-group-healthy.png -->

!\[Healthy Instances](screenshots/target-group-healthy.png)

### Auto Scaling Group — 2 Instances InService

<!-- Save as screenshots/asg-instances.png -->

!\[ASG Instances](screenshots/asg-instances.png)

### IAM Role — Least Privilege Policies

<!-- Save as screenshots/iam-role.png -->

!\[IAM Role](screenshots/iam-role.png)

### S3 Bucket — App Artifact

<!-- Save as screenshots/s3-bucket.png -->

!\[S3 Bucket](screenshots/s3-bucket.png)

### App Running on EC2 — systemctl status

<!-- Save as screenshots/systemctl-running.png -->

!\[Systemctl Running](screenshots/systemctl-running.png)

\---

## &#x20;Clip Walkthroughs

### 1 — Self Healing in Action

*Terminating an instance and watching ASG automatically replace it*

<!-- 
  HOW TO ADD:
  1. Screen record yourself terminating an instance and ASG replacing it
  2. Convert to GIF at ezgif.com
  3. Save as videos/self-healing.gif

!\[Self Healing](videos/self-healing.gif)

\---

### 2 — Architecture Walkthrough

*ALB → Target Group → ASG → IAM Role clicked through visually*

<!-- 
  HOW TO ADD:
  1. Screen record clicking through each AWS service briefly
  2. Convert to GIF at ezgif.com
  3. Save as videos/architecture-walkthrough.gif

!\[Architecture Walkthrough](videos/architecture-walkthrough.gif)

\---

### 3 — User Data Script Explained

*How EC2 automatically installs the runtime and deploys the app on launch*

<!-- 
  HOW TO ADD:
  1. Screen record scrolling through the userdata.sh file with zoom-ins
  2. Convert to GIF at ezgif.com
  3. Save as videos/userdata-explained.gif

!\[User Data Explained](videos/userdata-explained.gif)

\---

## &#x20;Deployment

See the full step-by-step guide → [deployment-guide.md](docs/deployment-guide.md)

### Quick Summary

1. Upload `app.zip` to S3
2. Create VPC with 2 public subnets across 2 AZs — enable auto-assign public IPv4
3. Create IAM role with Polly, Comprehend, S3 access
4. Create Security Groups for ALB and EC2
5. Create Launch Template with User Data script
6. Create Target Group on port 5000
7. Create Application Load Balancer on port 80
8. Create Auto Scaling Group — instances register to TG automatically

\---

## &#x20;Real Issues Encountered \& Fixed

These are actual problems hit during deployment — not a tutorial follow-along.

|Problem|Root Cause|Fix|
|-|-|-|
|User Data script timing out|Public subnets had no auto-assign public IP|Enabled auto-assign public IPv4 on subnets|
|S3 download 404 error|File name had spaces|Renamed to `app.zip`|
|App failing with exit-code 150|Wrong .NET package installed|Used `aspnetcore-runtime-9.0` not `dotnet-runtime-9.0`|
|Dotnet binary not found|Different path on Amazon Linux 2023|Used `/usr/lib64/dotnet/dotnet` instead of `/usr/bin/dotnet`|
|Browser timeout on ALB URL|Browser auto-upgrading to HTTPS|Explicitly used `http://` in URL|
|EC2 Instance Connect failing|No public IP assigned to instance|Enabled auto-assign public IPv4 on subnet|

\---

## &#x20;Key Technical Decisions

**Why public subnets instead of private + NAT Gateway?**
For this learning project, public subnets with restricted security groups were used to avoid NAT Gateway costs. In production, EC2 instances would sit in private subnets behind a NAT Gateway — that is the correct production architecture.

**Why User Data instead of Elastic Beanstalk?**
Elastic Beanstalk automates exactly what was built here manually — ALB, ASG, EC2, health checks, deployment. Building it from scratch demonstrates understanding of each component individually rather than treating Beanstalk as a black box.

**Why IAM Role instead of access keys?**
IAM roles follow AWS security best practices. Credentials are temporary, automatically rotated, and never stored on the instance — no risk of accidentally exposing keys in code or config files.

\---

## &#x20;Repository Structure

```
├── README.md
├── docs/
│   └── deployment-guide.md
├── userdata/
│   └── userdata.sh
├── screenshots/
│   ├── architecture-diagram.png
│   ├── alb-active.png
│   ├── target-group-healthy.png
│   ├── asg-instances.png
│   ├── iam-role.png
│   ├── s3-bucket.png
│   └── systemctl-running.png
└── videos/
    ├── app-demo.gif
    ├── self-healing.gif
    ├── architecture-walkthrough.gif
    └── userdata-explained.gif
```

\---

## &#x20;About

**Uttkarsh Tiwari**
AWS Solutions Architect Associate (In Progress) · CSE Student · Cloud \& Infrastructure Enthusiast

\---

## &#x20;License

This project is open source and available under the [MIT License](LICENSE).

