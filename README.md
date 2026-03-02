Multi-App Django Deployment on AWS ECS Fargate

This project deploys three independent Django applications on a single domain using path-based routing via an Application Load Balancer (ALB). It is fully automated with Python scripts and CloudFormation (IaC).

Supports:

Amazon Web Services

Amazon ECS (Fargate launch type)

Elastic Load Balancing (ALB path-based routing)

Amazon RDS (Free Tier optimized)

Amazon ECR

Amazon Route 53

AWS Certificate Manager

🏗 Architecture Overview
🌐 Routing Strategy
Path	Application
/	app1 (first deployed app)
/app2	app2
/app3	app3

The Python deployment script tracks the “last deployed app” using AWS SSM Parameter Store to determine routing paths automatically.

📁 Project Structure
.
├── app1/                 # Django App 1 (root path "/")
├── app2/                 # Django App 2 ("/app2")
├── app3/                 # Django App 3 ("/app3")
│
├── cloudformation/
│   ├── core.yml          # VPC, Subnets, Security Groups, ALB
│   ├── app-root.yml      # ECS Service for first app at "/"
│   └── app-extra.json    # ECS Service + ALB rule for additional apps (app2, app3)
│
├── scripts/
│   └── run.py            # Python deployment automation
│
└── README.md
☁️ CloudFormation Stacks
core.yml

Creates:

VPC, Public & Private Subnets

Security Groups

Application Load Balancer

HTTPS Listener with ACM certificate

Default Target Group for root app

app-root.yml

Used for first deployed app:

ECS Cluster / Service

Registers app to default Target Group (root /)

app-extra.json

Used for subsequent apps (app2, app3):

ECS Cluster / Service

Registers app to a new Target Group

Adds ALB listener rule with automatic priority selection

Routes /appname → target service

🐳 Docker Container Deployment

For each Django app:

Dockerized with Python 3.11

Runs Gunicorn on port 8000

Minimal Fargate task size: 0.25 vCPU / 0.5GB RAM

1 task per service

Images stored in Amazon ECR

Build, tag, and push steps:

# 1️⃣ Build image
docker build -t app1 ./app1

# 2️⃣ Tag image for ECR
docker tag app1:latest <account>.dkr.ecr.<region>.amazonaws.com/app1:latest

# 3️⃣ Push to ECR
docker push <account>.dkr.ecr.<region>.amazonaws.com/app1:latest

Repeat for app2 and app3.

⚙ Deployment Steps
1️⃣ Configure AWS CLI

Ensure AWS CLI is configured with credentials and default region:

aws configure

You will be prompted for:

AWS Access Key ID

AWS Secret Access Key

Default region (e.g., us-east-1)

Output format (json)

This ensures the Python script can access CloudFormation, ECR, ECS, ELBv2, and SSM.

2️⃣ Deploy Apps Using Python Script

Usage:

# Deploy App 1 (root "/")
python scripts/run.py app1 

# Deploy App 2 ("/app2")
python scripts/run.py app2 

# Deploy App 3 ("/app3")
python scripts/run.py app3 

The script handles:

Ensuring ECR repo exists

Resolving Docker image URI

Creating or updating CloudFormation stacks

Automatic ALB rule creation and priority selection

SSM parameter update to track last deployed app

Post-check of ECS service and Target Group health

After deployment, it prints the application URL:

http://<ALB_DNS>/[app_path]/


Outbound: All

ECS Security Grou
