#  NCAA Game Highlights Processor

A full-stack DevOps project that collects and processes NCAA basketball game highlights using a Docker container, RapidAPI, AWS MediaConvert, and stores results in AWS S3. Infrastructure is provisioned using Terraform for scalability and repeatability.

---

##  Features

-  Fetches NCAA basketball video highlights from [RapidAPI](https://rapidapi.com/)
-  Stores highlight metadata and videos in AWS S3
-  Processes videos with AWS MediaConvert (resolution, codec, bitrate)
-  Runs inside a Docker container
-  Deployable using Infrastructure as Code (Terraform)
-  Uses AWS Secrets Manager and environment variables for secure API key storage
-  Supports local and cloud-based execution

---

##  Project Structure

src/
├── config.py
├── fetch.py
├── process_one_video.py
├── mediaconvert_process.py
├── run_all.py
├── requirements.txt
├── Dockerfile
├── .env
├── .gitignore
└── terraform/
├── main.tf
├── variables.tf
├── secrets.tf
├── iam.tf
├── ecr.tf
├── ecs.tf
├── s3.tf
├── outputs.tf
└── container_definitions.tpl


---

## Quick Start

### 1️⃣ Clone the repo

```bash
git clone https://github.com/NazBilgic/NCAAGameHighlights.git
cd src
2️⃣ Create a RapidAPI account and get an API key
We use NCAA Basketball highlights (free in the basic plan).

3️⃣ Store API Key in AWS Secrets Manager
bash
Copy
Edit
aws secretsmanager create-secret \
  --name my-api-key \
  --description "API key for Sport Highlights API" \
  --secret-string '{"api_key":"YOUR_ACTUAL_API_KEY"}' \
  --region us-east-1
4️⃣ Set Up IAM Role
Create an IAM role named HighlightProcessorRole with these policies:

AmazonS3FullAccess

MediaConvertFullAccess

AmazonEC2ContainerRegistryFullAccess

Trust Policy:

json
Copy
Edit
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
5️⃣ Prepare .env file
env

RAPIDAPI_KEY=your_api_key
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
S3_BUCKET_NAME=your_bucket_name
MEDIACONVERT_ENDPOINT=https://abc123.mediaconvert.us-east-1.amazonaws.com
MEDIACONVERT_ROLE_ARN=arn:aws:iam::123456789012:role/HighlightProcessorRole
bash

chmod 600 .env
6️⃣ Run Locally in Docker
bash

docker build -t highlight-processor .
docker run --env-file .env highlight-processor
✅ You should now see:

basketball_highlight.json in S3

Raw and processed videos uploaded to s3://<bucket>/videos/ and processed_videos/

🧰 Terraform Deployment (Bonus)
bash
Copy
Edit
cd terraform
terraform init
terraform validate
terraform plan
terraform apply -var-file="terraform.dev.tfvars"
Push Docker Image to ECR
bash
Copy
Edit
docker build -t highlight-pipeline .
docker tag highlight-pipeline:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/highlight-pipeline:latest

aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/highlight-pipeline:latest


 What I Learned
Working with Docker + AWS Services (S3, IAM, ECS, MediaConvert)

Using Terraform for reproducible infrastructure

Secure secret management with AWS Secrets Manager

Automating highlight collection pipelines




