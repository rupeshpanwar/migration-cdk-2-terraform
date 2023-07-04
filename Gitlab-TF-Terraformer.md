<details>
<summary>Prepare Docker Image</summary>
<br>

  ```
    # Start with Ubuntu 20.04
FROM ubuntu:20.04 AS deploy

# Set Environment Variables for Proxy
ENV http_proxy=http://1.1.1.1:80
ENV https_proxy=http://2.2.2.2:80 

# Install dependencies
RUN apt-get update && \
    apt-get install -y curl wget unzip jq && \
    mkdir -p ~/.terraform.d/plugins/linux_amd64

# Install AWS CLI
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install

# Install Terraformer
# Get Latest Terraformer release
RUN TERRAFORMER_VERSION=$(curl -sL https://api.github.com/repos/GoogleCloudPlatform/terraformer/releases/latest | jq -r ".tag_name") && \
    curl -LO https://github.com/GoogleCloudPlatform/terraformer/releases/download/0.8.24/terraformer-all-linux-amd64 && \
    chmod +x terraformer-all-linux-amd64 && \
    mv terraformer-all-linux-amd64 /usr/local/bin/terraformer

# Install Terraform
# Get Latest Terraform release
RUN TERRAFORM_VERSION=$(curl -s https://checkpoint-api.hashicorp.com/v1/check/terraform | jq -r -M '.current_version') && \
    curl https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip -o terraform.zip && \
    unzip terraform.zip && \
    mv terraform /usr/local/bin/
  ```
  
</details>

<details>
<summary>Gitlab-Pipeline-Terraformer-Import</summary>
<br>

  ```
    stages:
  - import

import_tf_resources:
  stage: import
  image: ecr.ap-southeast-1.amazonaws.com/terraform-runner:v1
  before_script:
    - export http_proxy=http://11.1.1.1:80
    - export https_proxy=http://2.2.2.2.2:80 
  script:
    - apt-get update -y
    - apt-get install -y curl jq
    - aws sts get-caller-identity  # Check if AWS credentials are working
    - mkdir -p ~/.aws
    # - echo -e "[profile developer]\nrole_arn = arn:aws:iam::123:role/cdk-role-01\ncredential_source = Ec2InstanceMetadata" > ~/.aws/config
    # # Set AWS_PROFILE environment variable
    # - export AWS_PROFILE=developer
    - mkdir -p /root/.terraform.d/plugins/linux_amd64
    - terraform init
    - terraformer import aws --resources=s3 --filter=s3=s3-bucket-name --regions=ap-southeast-1 --profile ""
  
  artifacts:
    paths:
      - generated/
    expire_in: 1 week

  ```
      
</details>

<details>
<summary>Remote s3 backend-dynamodb table-locks</summary>
<br>

  ```
    #main.tf
    terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
  required_version = ">= 0.13"
}

provider "aws" {
  region = "ap-southeast-1" # change as per your region
}

resource "aws_s3_bucket" "terraform_state" {
  bucket = "terraform-state-01" # replace with your unique bucket name
  versioning {
    enabled = true
  }

  # Enable server-side encryption by default
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

resource "aws_dynamodb_table" "dynamodb_terraform_state_lock" {
  name         = "dynamodb-terraform-01"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}

terraform {
  backend "s3" {
    bucket         = "terraform-state-01"
    key            = "state"
    region         = "ap-southeast-1" 
    dynamodb_table = "dynamodb-terraform-01"
    encrypt        = true
  }
}


  ```
</details>

<details>
<summary>Introduction</summary>
<br>
  
</details>

<details>
<summary>Introduction</summary>
<br>
  
</details>
