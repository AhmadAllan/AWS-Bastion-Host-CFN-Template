# AWS CloudFormation Bastion Host Example Neasted Template

This project deploys an architecture with a Virtual Private Cloud (VPC), a public EC2 instance (bastion host), and a private EC2 instance. The setup includes SSH access configuration options, an Internet Gateway, a NAT Gateway, and a layered security model with separate public and private subnets.

## Architecture Overview

The deployment uses nested CloudFormation stacks to achieve modularity. Here’s a high-level overview of the architecture:

1. **VPC Stack**: Creates a VPC with public and private subnets, along with routing tables, an Internet Gateway for the public subnet, and a NAT Gateway for the private subnet.
2. **Public EC2 Stack**: Deploys a public EC2 instance (Apache web server), which can be used as a bastion host to manage the private instance. SSH and HTTP access is configured, and password or key-based authentication can be selected.
3. **Private EC2 Stack**: Deploys a private EC2 instance within the private subnet, accessible via SSH only through the bastion host. Password authentication can be set up with an SSM Parameter.

## Setup Instructions

### Prerequisites

- AWS CLI and IAM permissions to create CloudFormation stacks, EC2 instances, IAM roles, and VPC components.
- **SSH Key Pair** (optional): If you choose to use an SSH key pair, ensure that it’s created in the target AWS region.

### Steps for Deployment

1. **Upload Templates to S3**:
  - Save the following nested templates (`vpc-template.yaml`, `public-ec2.yaml`, `private-ec2.yaml`) in an S3 bucket in the region where you plan to deploy the stack.
  - Obtain the URLs for each uploaded file (click on each file in S3 and coby “Object URL”).

2. **Deploy the Master Template**:
  - Open the AWS CloudFormation Console.
  - Choose **Create stack** → **With new resources (standard)**.
  - Upload `master.yaml` directly in the console, as it’s the master file that links the nested stacks.

3. **Specify Template Parameters**:
  - **UseKeyPair**: Choose `true` to use an SSH key pair for the EC2 instances or `false` to disable key-based authentication.
  - **KeyPairName**: Enter the name of an existing key pair (required if `UseKeyPair` is `true`).
  - **PublicInstancePassword** and **PrivateInstancePassword**: Set passwords for the public and private instances. The passwords will be stored in AWS Systems Manager (SSM) Parameter Store.
  - **VpcTemplateURL**: Enter the S3 URL of `vpc-template.yaml`.
  - **PublicEc2TemplateURL**: Enter the S3 URL of `public-ec2.yaml`.
  - **PrivateEc2TemplateURL**: Enter the S3 URL of `private-ec2.yaml`.

4. **Review and Create**:
  - Review the parameters and options. Acknowledge that CloudFormation will create IAM resources if prompted.
  - Click **Create stack**.

5. **Monitor Stack Creation**:
  - The stack creation may take several minutes. Monitor progress in the **Events** tab of the master stack.
  - Once completed, check the **Outputs** section for information such as the public IP of the bastion host and the private instance.

## Accessing Instances

- **Public EC2 Instance (Bastion Host)**:
  - Access the public instance using its public IP and the configured SSH key (if `UseKeyPair` is true) or with the password via SSH.
  - **HTTP**: The public instance also hosts an Apache server, accessible via its public IP at port 80.

- **Private EC2 Instance**:
  - Access the private instance by first SSHing into the public instance (bastion host) and then initiating an SSH session from the bastion to the private instance.
  - SSH and password configurations are set up in the `UserData` scripts.

## Cleanup

To delete the stack and associated resources, delete the master stack from the CloudFormation Console. This action will automatically delete all nested stacks and resources associated with them.

## Stack Structure and Modular Templates

The setup consists of four YAML templates:
- **master.yaml**: Orchestrates the entire deployment by calling nested stacks.
- **vpc-template.yaml**: Creates VPC, subnets, and related networking resources.
- **public-ec2.yaml**: Deploys the public-facing EC2 instance (bastion host).
- **private-ec2.yaml**: Deploys the private EC2 instance in the private subnet.

By uploading the nested templates (`vpc-template.yaml`, `public-ec2.yaml`, `private-ec2.yaml`) to S3, you can easily reference them in the `master.yaml` stack, simplifying stack management and maintaining modularity in your architecture.
