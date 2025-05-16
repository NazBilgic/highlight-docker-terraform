# NCAA Game Highlights Processor

This is a full-stack DevOps project that automates the collection, processing, and delivery of NCAA basketball game highlights. It uses Docker for containerization, Terraform for infrastructure provisioning, and various AWS services including S3, ECS, ECR, MediaConvert, IAM, and Secrets Manager.

## Description

The application fetches NCAA basketball highlight videos from RapidAPI, processes them using AWS MediaConvert, and stores both raw and processed files in an S3 bucket. It runs inside a Docker container and is deployed to the cloud with Terraform for scalability and consistency.

## Features

- Fetch NCAA highlight videos from RapidAPI  
- Process videos with AWS MediaConvert (codec, resolution, bitrate)  
- Store raw and processed video files in S3  
- Dockerized for local and cloud execution  
- Infrastructure managed via Terraform  
- Secure API credentials using AWS Secrets Manager  
- Optional deployment to AWS ECS and ECR

## Technologies Used

- Docker  
- Python  
- AWS (S3, ECS, ECR, IAM, MediaConvert, Secrets Manager)  
- Terraform  
- RapidAPI  

## Project Structure

highlight-docker-terraform/
│
├── src/
│   ├── config.py
│   ├── fetch.py
│   ├── process_one_video.py
│   ├── mediaconvert_process.py
│   ├── run_all.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .env
│
└── terraform/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── s3.tf
    ├── iam.tf
    ├── ecr.tf
    ├── ecs.tf
    ├── secrets.tf
    └── container_definitions.tpl

## Quick Start

1. Clone the repository

git clone https://github.com/NazBilgic/highlight-docker-terraform.git
cd highlight-docker-terraform/src

2. Set up RapidAPI and get your API key  
3. Store API key in AWS Secrets Manager

aws secretsmanager create-secret \
  --name my-api-key \
  --secret-string '{"api_key":"YOUR_API_KEY"}' \
  --region us-east-1

4. Create an IAM Role (HighlightProcessorRole) with these policies:
- AmazonS3FullAccess
- MediaConvertFullAccess
- AmazonEC2ContainerRegistryFullAccess

Trust policy:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "ec2.amazonaws.com",
          "ecs-tasks.amazonaws.com",
          "mediaconvert.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

5. Create a `.env` file

RAPIDAPI_KEY=your_api_key  
AWS_ACCESS_KEY_ID=your_access_key  
AWS_SECRET_ACCESS_KEY=your_secret_key  
S3_BUCKET_NAME=your_bucket_name  
MEDIACONVERT_ENDPOINT=https://xyz.mediaconvert.us-east-1.amazonaws.com  
MEDIACONVERT_ROLE_ARN=arn:aws:iam::123456789012:role/HighlightProcessorRole

chmod 600 .env

6. Run locally with Docker

docker build -t highlight-processor .
docker run --env-file .env highlight-processor

## Deploy with Terraform

cd terraform
terraform init
terraform validate
terraform plan
terraform apply -var-file="terraform.dev.tfvars"

## Push Docker Image to ECR

docker build -t highlight-pipeline .
docker tag highlight-pipeline:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/highlight-pipeline:latest

aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/highlight-pipeline:latest

## What I Learned

- Dockerized pipeline development  
- AWS video processing using MediaConvert  
- Infrastructure-as-Code with Terraform  
- Secure secret handling with Secrets Manager  
- Deploying containers to ECS via ECR  
- Using API integrations in real-world cloud environments

## Author

Naz Bilgiç  
linkedin.com/in/nazbilgic
