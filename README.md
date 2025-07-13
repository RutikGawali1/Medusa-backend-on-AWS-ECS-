
# 🚀 Medusa Backend Deployment on AWS ECS Fargate using Terraform & GitHub Actions

This project demonstrates how to deploy the [Medusa](https://medusajs.com) open-source headless commerce backend on **AWS ECS (Fargate)** using **Terraform** for infrastructure provisioning and **GitHub Actions** for CI/CD automation.

## 📁 Project Structure

```
medusa-aws-ecs/
├── medusa-backend/               # Medusa backend source code
│   ├── .env                      # Environment variables
│   ├── Dockerfile                # Container build
│   └── ...
├── terraform/                    # Terraform modules
│   ├── provider.tf               # AWS provider
│   ├── vpc.tf                    # VPC and subnets
│   ├── ecs.tf                    # ECS cluster, services
│   ├── ecr.tf                    # Elastic Container Registry
│   ├── alb.tf                    # Load Balancer
│   ├── iam.tf                    # IAM roles & policies
│   ├── rds.tf                    # PostgreSQL database
│   ├── outputs.tf                # Terraform outputs
│   └── variables.tf              # Variable definitions
├── .github/
│   └── workflows/
│       └── deploy.yml            # GitHub Actions workflow
└── README.md
```

## ✅ Prerequisites

- AWS account (with IAM admin access)
- [Terraform](https://developer.hashicorp.com/terraform/install) installed
- [Docker](https://www.docker.com/products/docker-desktop/) installed
- Node.js **v18** installed (not v20 or v22)
- GitHub repo to push code
- PostgreSQL setup (local or via RDS)

## 🧱 Step-by-Step Deployment

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/<your-username>/medusa-aws-ecs.git
cd medusa-aws-ecs
```

### 2️⃣ Create Medusa Backend Locally

```bash
npx @medusajs/medusa-cli@1.8.0 new medusa-backend --seed
cd medusa-backend
```

Create `.env` file:

```
DATABASE_URL=postgres://postgres:pass1234@localhost:5432/medusa
JWT_SECRET=supersecret
COOKIE_SECRET=supercookie
```

Run locally:

```bash
npm install
npm run dev
```

### 3️⃣ Add Dockerfile for Medusa

In `medusa-backend/`, create `Dockerfile`:

```Dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
EXPOSE 9000
CMD ["npm", "start"]
```

Build and test:

```bash
docker build -t medusa-app .
docker run -p 9000:9000 medusa-app
```

### 4️⃣ Terraform: Provision AWS Infrastructure

```bash
cd terraform
terraform init
terraform apply -auto-approve
```

### 5️⃣ Push Docker Image to ECR

```bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <ecr-url>

docker tag medusa-app <ecr-url>:latest
docker push <ecr-url>:latest
```

### 6️⃣ Configure CI/CD (GitHub Actions)

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy Medusa to ECS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repo
      uses: actions/checkout@v3

    - name: Set up Docker
      uses: docker/setup-buildx-action@v3

    - name: Login to ECR
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build and Push Docker image
      run: |
        docker build -t medusa-app .
        docker tag medusa-app:latest <your-ecr-url>:latest
        docker push <your-ecr-url>:latest

    - name: Deploy using Terraform
      run: |
        cd terraform
        terraform init
        terraform apply -auto-approve
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## 🔍 Expected Output

- ECS Fargate service with Medusa backend running
- Accessible at: `http://<alb-dns>:9000`

## 📹 Video Demo (Add your own)

📺 **Video Link**: [https://your-video-demo-link.com](https://your-video-demo-link.com)

## ✍️ Author

**Rutik Gawali**  
💼 LinkedIn: [linkedin.com/in/rutikgawali](https://linkedin.com/in/rutikgawali)

## 💬 Questions?

Raise an issue or contact me directly!
