# AWS VPC Deployment with CloudFormation and TaskCat Testing

This repository is for the NT548.P11 course - Fall 2024 semester at University of Information Technology - VNUHCM.

It provides a CloudFormation-based solution to create a Virtual Private Cloud (VPC) on AWS, complete with EC2 instances, NAT Gateway, and routing configurations. The deployment is tested using TaskCat, ensuring the infrastructure's correctness. We will also use an S3 bucket to store and deploy the CloudFormation templates.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Prerequisites](#prerequisites)
3. [Folder Structure](#folder-structure)
4. [Deployment Steps](#deployment-steps)
5. [Testing with TaskCat](#testing-with-taskcat)
6. [Cleaning Up](#cleaning-up)
7. [Advantages of Using S3 for Deployment](#advantages-of-using-s3-for-deployment)
8. [Additional Notes](#additional-notes)

## Architecture Overview

The infrastructure deployed consists of:
- A custom VPC (`vpc.yaml`).
- EC2 instances (`EC2.yaml`).
- A NAT Gateway for outgoing internet access (`NAT.yaml`).
- Routing configuration to manage traffic within the VPC (`RouteTable.yaml`).

## Prerequisites

- AWS CLI installed and configured with proper credentials
- Python 3 installed
- Taskcat installed
- An AWS account with necessary IAM permissions

## Folder Structure

The repository contains the following files:

```plaintext
├── main.yaml          # The main CloudFormation template that includes all nested stacks
├── vpc.yaml           # VPC CloudFormation template
├── EC2.yaml           # EC2 CloudFormation template
├── NAT.yaml           # NAT Gateway CloudFormation template
├── RouteTable.yaml    # Routing CloudFormation template
├── .taskcat.yml       # TaskCat configuration file for testing
└── README.md          # This README file
```

## Deployment Steps

### 1. Create a Key Pair

Create a key pair to enable SSH access to your EC2 instance:

```bash
aws ec2 create-key-pair --key-name group12 --query 'KeyMaterial' --output text > group12.pem
```

### 2. Upload CloudFormation Templates to S3

Create an S3 bucket to store the CloudFormation templates:

```bash
aws s3 mb s3://group12-testing-bucket --region us-east-1
```

Upload the CloudFormation templates to the S3 bucket:

```bash
aws s3 cp NAT.yaml s3://group12-testing-bucket/
aws s3 cp EC2.yaml s3://group12-testing-bucket/
aws s3 cp vpc.yaml s3://group12-testing-bucket/
aws s3 cp RouteTable.yaml s3://group12-testing-bucket/
```

### 3. Deploy the VPC Using CloudFormation

Deploy the VPC stack through CloudFormation using the main template:

NOTE: Should run the test with taskcat before do this step!

```bash
aws cloudformation deploy --stack-name group12 --template-file main.yaml
```

This will deploy the resources as specified in the `main.yaml` template.

## Testing with TaskCat

TaskCat can be used to validate the CloudFormation templates before deployment.

### 1. Install Python and TaskCat

Ensure Python 3 is installed. If not, install it first. Then proceed to install TaskCat:

```bash
sudo yum -y update
python3 --version
curl -O https://bootstrap.pypa.io/get-pip.py
python3 get-pip.py --user
sudo pip install --upgrade pip
pip3 install taskcat --user
```

### 2. Verify the TaskCat Installation

Confirm TaskCat is installed by checking the version:

```bash
taskcat --version
```

### 3. Run TaskCat Tests

To validate the templates using TaskCat, run:

```bash
taskcat test run
```

TaskCat will execute the tests specified in the `.taskcat.yml` configuration file, and the test results will be stored in the `taskcat_outputs` directory.

## Cleaning Up

After deployment, you may want to clean up the created resources to avoid additional charges:

1. Delete the CloudFormation stack:

    ```bash
    aws cloudformation delete-stack --stack-name group12
    ```

2. Remove the key pair:

    ```bash
    aws ec2 delete-key-pair --key-name group12
    ```

3. Delete the S3 bucket and its contents:

    ```bash
    aws s3 rb s3://group12-testing-bucket --force
    ```

## Advantages of Using S3 for Deployment

- **Centralized Storage:** Keeps all templates in a central place, making it easier to manage and version them.
- **Secure Access:** Leverage S3 bucket policies and IAM roles for fine-grained access control.
- **Template Sharing:** Easily share templates across multiple accounts or regions.
- **Reduces Local Dependencies:** CloudFormation can fetch templates directly from S3, avoiding local path issues.

## Additional Notes

- Make sure your AWS CLI is properly configured (`aws configure`).
- Ensure that the region specified (`us-east-1` in this example) matches your AWS environment.
- IAM roles and permissions should be set up correctly for deploying CloudFormation templates and using TaskCat for testing.
- When using TaskCat, ensure that your configuration file (`.taskcat.yml`) is correctly set up for your testing needs.

