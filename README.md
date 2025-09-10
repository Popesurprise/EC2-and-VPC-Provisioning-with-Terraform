Task: Provision an AWS VPC with an EC2 Instance Using Terraform

Prerequisites
1. **AWS Account**: You need an active AWS account with access keys (Access Key ID and Secret Access Key).
2. **Terraform Installed**: Download and install Terraform from [terraform.io](https://www.terraform.io/downloads.html).
3. **AWS CLI (Optional)**: For configuring AWS credentials locally.
4. **Basic Knowledge**: Familiarity with AWS and basic command-line usage.

---

### Steps to Accomplish the Task

#### Step 1: Set Up Your Environment
1. Install Terraform:
   - Download the appropriate Terraform binary for your operating system from the official website.
   - Unzip and add Terraform to your system’s PATH (e.g., `/usr/local/bin` on Linux/Mac).
   - Verify installation: `terraform --version`.

2. Configure AWS Credentials:
   - Run `aws configure` (if using AWS CLI) to set your Access Key ID, Secret Access Key, region (e.g., `us-east-1`), and output format (e.g., `json`).
   - Alternatively, set environment variables:
     ```bash
     export AWS_ACCESS_KEY_ID="your-access-key"
     export AWS_SECRET_ACCESS_KEY="your-secret-key"
     export AWS_DEFAULT_REGION="us-east-1"
     ```

Step 2: Create a Terraform Configuration
1. Create a Project Directory:
   ```bash
   mkdir terraform-aws-example
   cd terraform-aws-example
   ```

2. Write Terraform Configuration Files:
   Create a file named `main.tf` with the following code to define the AWS provider, VPC, subnet, security group, and EC2 instance.

   ```hcl
   # main.tf

   # Configure the AWS Provider
   provider "aws" {
     region = "us-east-1"
   }

   # Create a VPC
   resource "aws_vpc" "example_vpc" {
     cidr_block           = "10.0.0.0/16"
     enable_dns_hostnames = true
     enable_dns_support   = true
     tags = {
       Name = "example-vpc"
     }
   }

   # Create a Public Subnet
   resource "aws_subnet" "public_subnet" {
     vpc_id            = aws_vpc.example_vpc.id
     cidr_block        = "10.0.1.0/24"
     availability_zone = "us-east-1a"
     map_public_ip_on_launch = true
     tags = {
       Name = "public-subnet"
     }
   }

   # Create an Internet Gateway
   resource "aws_internet_gateway" "example_igw" {
     vpc_id = aws_vpc.example_vpc.id
     tags = {
       Name = "example-igw"
     }
   }

   # Create a Route Table
   resource "aws_route_table" "public_route_table" {
     vpc_id = aws_vpc.example_vpc.id
     route {
       cidr_block = "0.0.0.0/0"
       gateway_id = aws_internet_gateway.example_igw.id
     }
     tags = {
       Name = "public-route-table"
     }
   }

   # Associate Route Table with Subnet
   resource "aws_route_table_association" "public_subnet_association" {
     subnet_id      = aws_subnet.public_subnet.id
     route_table_id = aws_route_table.public_route_table.id
   }

   # Create a Security Group for SSH Access
   resource "aws_security_group" "example_sg" {
     vpc_id = aws_vpc.example_vpc.id
     name   = "example-sg"

     ingress {
       from_port   = 22
       to_port     = 22
       protocol    = "tcp"
       cidr_blocks = ["0.0.0.0/0"] # Allow SSH from anywhere (restrict in production)
     }

     egress {
       from_port   = 0
       to_port     = 0
       protocol    = "-1"
       cidr_blocks = ["0.0.0.0/0"]
     }

     tags = {
       Name = "example-sg"
     }
   }

   # Create an EC2 Instance
   resource "aws_instance" "example_instance" {
     ami                    = "ami-0c55b159cbfafe1f0" # Amazon Linux 2 AMI (update for your region)
     instance_type          = "t2.micro"
     subnet_id              = aws_subnet.public_subnet.id
     vpc_security_group_ids = [aws_security_group.example_sg.id]
     key_name               = "my-key-pair" # Replace with your SSH key pair name
     tags = {
       Name = "example-instance"
     }
   }

   # Output the Public IP of the EC2 Instance
   output "instance_public_ip" {
     value       = aws_instance.example_instance.public_ip
     description = "Public IP address of the EC2 instance"
   }
   ```

3. Create a Variables File (Optional):
   To make the configuration reusable, create a `variables.tf` file:
   ```hcl
   # variables.tf

   variable "region" {
     description = "AWS region for resources"
     default     = "us-east-1"
   }

   variable "ami_id" {
     description = "AMI ID for the EC2 instance"
     default     = "ami-0c55b159cbfafe1f0" # Update based on your region
   }

   variable "instance_type" {
     description = "EC2 instance type"
     default     = "t2.micro"
   }

   variable "key_name" {
     description = "Name of the SSH key pair"
     type        = string
   }
   ```

4. Create a Key Pair:
   - In the AWS Management Console, navigate to EC2 > Key Pairs.
   - Create a key pair named `my-key-pair` and download the `.pem` file.
   - Update the `key_name` in `main.tf` or pass it as a variable.

Step 3: Initialize and Apply the Terraform Configuration
1. Initialize Terraform:
   Run the following command to download the AWS provider and set up the working directory:
   ```bash
   terraform init
   ```

2. Preview the Plan:
   Check what resources Terraform will create:
   ```bash
   terraform plan
   ```

3. Apply the Configuration:
   Deploy the infrastructure:
   ```bash
   terraform apply
   ```
   - Type `yes` when prompted to confirm.
   - Terraform will provision the VPC, subnet, internet gateway, route table, security group, and EC2 instance.
   - After completion, it will output the public IP of the EC2 instance.

Step 4: Verify the Infrastructure
1. Check AWS Console:
   - Log in to the AWS Management Console.
   - Navigate to EC2 > Instances to confirm the instance is running.
   - Verify the VPC, subnet, and security group under VPC > Your VPCs and EC2 > Security Groups.

2. SSH into the EC2 Instance:
   Use the public IP output by Terraform to connect:
   ```bash
   ssh -i my-key-pair.pem ec2-user@<public-ip>
   ```

Step 5: Manage and Clean Up
1. **Update Infrastructure**:
   - Modify `main.tf` (e.g., change the instance type to `t3.micro`).
   - Run `terraform plan` to preview changes and `terraform apply` to update.

2. Destroy Infrastructure:
   When done, clean up to avoid AWS charges:
   ```bash
   terraform destroy
   ```
   - Type `yes` to confirm.
