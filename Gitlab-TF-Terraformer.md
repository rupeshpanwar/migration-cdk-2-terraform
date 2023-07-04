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
<summary>Introduction</summary>
<br>
  
</details>

<details>
<summary>Introduction</summary>
<br>
  
</details>
