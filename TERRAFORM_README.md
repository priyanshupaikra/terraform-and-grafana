# Terraform - Complete Setup & Guide

![Terraform](https://img.shields.io/badge/Terraform-v1.0+-blue) ![License](https://img.shields.io/badge/License-MPL--2.0-brightgreen) ![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen)

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Getting Started](#getting-started)
- [HCL Syntax](#hcl-syntax)
- [Providers](#providers)
- [Resources](#resources)
- [Variables & Outputs](#variables--outputs)
- [State Management](#state-management)
- [Modules](#modules)
- [Workspaces](#workspaces)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

---

## Introduction

### What is Terraform?

Terraform is an **Infrastructure as Code (IaC)** tool that allows you to define, preview, and deploy infrastructure using declarative configuration files.

### Key Features

- **Declarative IaC**: Describe desired state, not steps to achieve it
- **Multi-Cloud**: AWS, Azure, GCP, and 150+ providers
- **Plan & Apply**: Preview changes before applying
- **State Management**: Track infrastructure state
- **Modules**: Reusable infrastructure components
- **Workspaces**: Manage multiple environments
- **Remote State**: Collaborative infrastructure management
- **Version Control**: Track infrastructure changes like code

### Use Cases

- **Cloud Infrastructure**: VPCs, subnets, security groups
- **Compute Resources**: EC2 instances, Lambda functions
- **Databases**: RDS, DynamoDB, Elasticsearch
- **Networking**: Load balancers, CDN, firewalls
- **Containerization**: ECS, Kubernetes, Docker
- **Monitoring**: CloudWatch, Datadog
- **CI/CD**: GitHub Actions, Jenkins
- **Multi-Cloud**: Deploy across AWS, Azure, GCP

---

## Installation

### Prerequisites

- Linux, macOS, or Windows
- Administrative access to your system
- Appropriate cloud provider credentials

### Option 1: Official Binaries

#### macOS (Homebrew)

```bash
# Install
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Verify
terraform version
```

#### Linux (Direct Download)

```bash
# Download
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip

# Extract
unzip terraform_1.6.0_linux_amd64.zip

# Install
sudo mv terraform /usr/local/bin/

# Verify
terraform version
```

#### Windows (Chocolatey)

```powershell
# Install
choco install terraform

# Verify
terraform version
```

### Option 2: Package Managers

#### Ubuntu/Debian

```bash
# Add HashiCorp repository
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# Install
sudo apt update && sudo apt install terraform

# Verify
terraform version
```

#### CentOS/RHEL

```bash
# Add repository
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo

# Install
sudo yum -y install terraform

# Verify
terraform version
```

### Verify Installation

```bash
# Check Terraform version
terraform version

# Check if commands are available
terraform -help

# List available commands
terraform
```

---

## Getting Started

### Step 1: Create Project Directory

```bash
# Create directory
mkdir -p my-terraform-project
cd my-terraform-project

# Initialize git (optional but recommended)
git init
```

### Step 2: Create Configuration Files

#### main.tf

```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

#### variables.tf

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}
```

#### outputs.tf

```hcl
output "region" {
  description = "AWS region"
  value       = var.aws_region
}
```

### Step 3: Initialize Terraform

```bash
# Initialize working directory
terraform init

# Expected output:
# - Downloading provider plugins
# - Initializing backend
# - Terraform has been successfully initialized!
```

### Step 4: Validate Configuration

```bash
# Validate syntax
terraform validate

# Format code
terraform fmt -recursive

# Check what will be created
terraform plan
```

### Step 5: Apply Configuration

```bash
# Apply changes (creates resources)
terraform apply

# Review plan and type 'yes' to confirm

# Verify resources were created
terraform show
```

### Step 6: Destroy Resources

```bash
# Destroy all resources
terraform destroy

# Confirm by typing 'yes'

# Verify everything is destroyed
terraform show
```

---

## HCL Syntax

### Blocks

#### Provider Block

```hcl
provider "aws" {
  region = "us-east-1"
  
  assume_role {
    role_arn = "arn:aws:iam::123456789012:role/terraform-role"
  }
}
```

#### Resource Block

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "MyWebServer"
  }
}
```

#### Data Block

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }

  owners = ["099720109477"]  # Canonical
}
```

#### Variable Block

```hcl
variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
  
  validation {
    condition     = var.instance_count > 0 && var.instance_count < 10
    error_message = "Instance count must be between 1 and 9."
  }
}
```

#### Output Block

```hcl
output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.web.id
  sensitive   = false
}
```

#### Local Block

```hcl
locals {
  common_tags = {
    Project     = "MyProject"
    Environment = var.environment
    CreatedBy   = "Terraform"
  }
}
```

### Data Types

```hcl
# String
variable "name" {
  type = string
  default = "MyValue"
}

# Number
variable "port" {
  type = number
  default = 8080
}

# Boolean
variable "enable_logging" {
  type = bool
  default = true
}

# List
variable "availability_zones" {
  type = list(string)
  default = ["us-east-1a", "us-east-1b"]
}

# Map
variable "tags" {
  type = map(string)
  default = {
    Environment = "dev"
    Team = "ops"
  }
}

# Object
variable "database" {
  type = object({
    engine   = string
    version  = string
    instance_class = string
  })
}

# Tuple
variable "mixed_values" {
  type = tuple([string, number, bool])
  default = ["value", 123, true]
}
```

### String Interpolation

```hcl
# Simple interpolation
name = "my-instance-${var.environment}"

# Complex interpolation
greeting = "Hello, ${upper(var.name)}!"

# Multi-line strings
policy = jsonencode({
  Version = "2012-10-17"
  Statement = [...]
})
```

### Functions

```hcl
# String functions
upper("hello")              # "HELLO"
lower("HELLO")              # "hello"
length("abc")               # 3
substr("hello", 1, 3)       # "ell"
split(",", "a,b,c")        # ["a", "b", "c"]
join(",", ["a", "b", "c"]) # "a,b,c"

# Numeric functions
abs(-5)                     # 5
max(1, 2, 3)               # 3
min(1, 2, 3)               # 1
ceil(4.3)                  # 5
floor(4.7)                 # 4

# Collection functions
length(var.list)           # List length
keys(var.map)              # Map keys
values(var.map)            # Map values
contains(var.list, value)  # Check if contains
distinct(var.list)         # Remove duplicates
flatten(var.list)          # Flatten nested lists
merge(map1, map2)          # Merge maps

# Type functions
type(var.value)            # Get variable type
can(expression)            # Check if valid

# File functions
file("path/to/file")       # Read file content
jsondecode(file("file.json")) # Decode JSON

# Conditional
condition ? true_value : false_value
```

---

## Providers

### Configuring Providers

#### AWS Provider

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = {
      Project     = "MyProject"
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}
```

#### Multiple Regions

```hcl
provider "aws" {
  alias  = "us_east"
  region = "us-east-1"
}

provider "aws" {
  alias  = "us_west"
  region = "us-west-2"
}

resource "aws_instance" "east" {
  provider = aws.us_east
  ami      = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}

resource "aws_instance" "west" {
  provider = aws.us_west
  ami      = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}
```

#### Azure Provider

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

#### Google Cloud Provider

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  project = var.gcp_project
  region  = "us-central1"
}
```

#### Kubernetes Provider

```hcl
terraform {
  required_providers {
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
  }
}

provider "kubernetes" {
  host                   = aws_eks_cluster.main.endpoint
  cluster_ca_certificate = base64decode(aws_eks_cluster.main.certificate_authority[0].data)
  token                  = data.aws_eks_cluster_auth.main.token
}
```

---

## Resources

### EC2 Instances

```hcl
resource "aws_instance" "web" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = "t2.micro"
  key_name              = aws_key_pair.deployer.key_name
  subnet_id             = aws_subnet.main.id
  vpc_security_group_ids = [aws_security_group.web.id]
  
  root_block_device {
    volume_size           = 20
    volume_type          = "gp2"
    delete_on_termination = true
  }
  
  user_data = base64encode(<<-EOF
    #!/bin/bash
    sudo yum update -y
    sudo yum install -y httpd
    sudo systemctl start httpd
  EOF
  )
  
  tags = {
    Name = "WebServer"
  }
}
```

### VPC & Networking

```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "MainVPC"
  }
}

# Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  
  tags = {
    Name = "PublicSubnet"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "MainIGW"
  }
}

# Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block      = "0.0.0.0/0"
    gateway_id      = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "PublicRT"
  }
}

# Route Table Association
resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

### Security Groups

```hcl
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id
  
  # Ingress rules
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.allowed_ssh_cidr]
  }
  
  # Egress rules
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "WebSecurityGroup"
  }
}
```

### RDS Database

```hcl
resource "aws_db_instance" "main" {
  identifier     = "mydb"
  engine         = "postgres"
  engine_version = "15.3"
  instance_class = "db.t3.micro"
  
  allocated_storage     = 20
  storage_type         = "gp2"
  storage_encrypted    = true
  
  db_name  = "mydatabase"
  username = "admin"
  password = random_password.db_password.result
  
  skip_final_snapshot       = false
  final_snapshot_identifier = "mydb-final-snapshot"
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"
  
  multi_az = true
  
  tags = {
    Name = "PostgreSQL"
  }
}

resource "random_password" "db_password" {
  length  = 16
  special = true
}
```

### S3 Bucket

```hcl
resource "aws_s3_bucket" "main" {
  bucket = "my-bucket-${data.aws_caller_identity.current.account_id}"
  
  tags = {
    Name = "MyBucket"
  }
}

resource "aws_s3_bucket_versioning" "main" {
  bucket = aws_s3_bucket.main.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "main" {
  bucket = aws_s3_bucket.main.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

---

## Variables & Outputs

### Variable Definition

```hcl
# variables.tf

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
  
  validation {
    condition     = can(regex("^t2\\.", var.instance_type))
    error_message = "Instance type must be t2 family."
  }
}

variable "enable_monitoring" {
  description = "Enable CloudWatch monitoring"
  type        = bool
  default     = true
}

variable "tags" {
  description = "Common tags for resources"
  type        = map(string)
  default = {
    Environment = "dev"
    Project     = "MyProject"
  }
}
```

### Using Variables

#### In Terraform Code

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  tags = merge(var.tags, {
    Name = "WebServer"
  })
}
```

#### From Command Line

```bash
# Pass variable at apply time
terraform apply -var="instance_type=t2.small"

# Pass multiple variables
terraform apply \
  -var="instance_type=t2.small" \
  -var="enable_monitoring=false"

# Pass variable file
terraform apply -var-file="dev.tfvars"

# Set via environment variable
export TF_VAR_instance_type="t2.small"
terraform apply
```

#### Variable File

```hcl
# dev.tfvars
aws_region        = "us-east-1"
instance_type     = "t2.micro"
enable_monitoring = true

tags = {
  Environment = "dev"
  Project     = "MyProject"
  CostCenter  = "Engineering"
}
```

### Output Values

```hcl
# outputs.tf

output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.web.id
}

output "instance_ip" {
  description = "EC2 instance public IP"
  value       = aws_instance.web.public_ip
}

output "database_endpoint" {
  description = "RDS database endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true
}

output "s3_bucket_name" {
  description = "S3 bucket name"
  value       = aws_s3_bucket.main.bucket
}
```

### Accessing Outputs

```bash
# Show all outputs
terraform output

# Show specific output
terraform output instance_id

# Output as JSON
terraform output -json

# Output raw value (without quotes)
terraform output -raw instance_ip
```

---

## State Management

### Local State

```bash
# Terraform state stored locally
# File: terraform.tfstate
# Backup: terraform.tfstate.backup
```

### Remote State (S3 Backend)

#### Create Backend Configuration

```hcl
# backend.tf

terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

#### Initialize Backend

```bash
# First time setup
terraform init

# Migrate state to S3
terraform init -migrate-state

# Confirm migration
# Type 'yes' when prompted
```

#### State Locking (DynamoDB)

Create lock table:

```hcl
resource "aws_dynamodb_table" "terraform_locks" {
  name           = "terraform-locks"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

### State Commands

```bash
# Show current state
terraform show

# List resources in state
terraform state list

# Show specific resource
terraform state show aws_instance.web

# Remove resource from state (don't delete)
terraform state rm aws_instance.web

# Replace resource
terraform state replace-provider hashicorp/aws aws

# Pull remote state
terraform state pull > backup.tfstate

# Push local state to remote
terraform state push backup.tfstate
```

### State Best Practices

✅ **Always use remote state** for team environments  
✅ **Enable encryption** for sensitive data  
✅ **Use state locking** to prevent concurrent modifications  
✅ **Backup state regularly** before major changes  
✅ **Never commit state files** to version control  
✅ **Use .gitignore** for local state files  
✅ **Implement RBAC** for state access  

---

## Modules

### Module Structure

```
modules/
├── vpc/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
├── rds/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
└── ec2/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

### Creating a Module

#### VPC Module

```hcl
# modules/vpc/main.tf

resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = var.tags
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidr
  availability_zone       = var.availability_zone
  map_public_ip_on_launch = true
  
  tags = merge(var.tags, { Name = "PublicSubnet" })
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = var.tags
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block      = "0.0.0.0/0"
    gateway_id      = aws_internet_gateway.main.id
  }
  
  tags = merge(var.tags, { Name = "PublicRT" })
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

```hcl
# modules/vpc/variables.tf

variable "cidr_block" {
  description = "VPC CIDR block"
  type        = string
}

variable "public_subnet_cidr" {
  description = "Public subnet CIDR"
  type        = string
}

variable "availability_zone" {
  description = "Availability zone"
  type        = string
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default     = {}
}
```

```hcl
# modules/vpc/outputs.tf

output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_id" {
  description = "Public subnet ID"
  value       = aws_subnet.public.id
}

output "internet_gateway_id" {
  description = "Internet gateway ID"
  value       = aws_internet_gateway.main.id
}
```

### Using Modules

```hcl
# main.tf

module "vpc" {
  source = "./modules/vpc"
  
  cidr_block         = "10.0.0.0/16"
  public_subnet_cidr = "10.0.1.0/24"
  availability_zone  = "us-east-1a"
  
  tags = {
    Project     = "MyProject"
    Environment = var.environment
  }
}

module "ec2" {
  source = "./modules/ec2"
  
  instance_type      = "t2.micro"
  subnet_id          = module.vpc.public_subnet_id
  security_group_ids = [aws_security_group.web.id]
  
  tags = {
    Name = "WebServer"
  }
}
```

---

## Workspaces

### Managing Workspaces

```bash
# Create workspace
terraform workspace new dev
terraform workspace new prod

# List workspaces
terraform workspace list

# Switch workspace
terraform workspace select dev

# Show current workspace
terraform workspace show

# Delete workspace
terraform workspace delete dev
```

### Using Workspaces

```hcl
# main.tf

locals {
  env = terraform.workspace
  
  environment_config = {
    dev = {
      instance_type = "t2.micro"
      db_size       = "db.t2.micro"
    }
    prod = {
      instance_type = "t2.large"
      db_size       = "db.r5.xlarge"
    }
  }
}

resource "aws_instance" "web" {
  instance_type = local.environment_config[local.env].instance_type
  
  tags = {
    Name        = "WebServer"
    Environment = local.env
  }
}
```

---

## Essential Commands

### Workflow Commands

```bash
# Initialize working directory
terraform init

# Validate configuration
terraform validate

# Format configuration files
terraform fmt -recursive

# Plan changes (review before apply)
terraform plan -out=tfplan

# Apply changes
terraform apply tfplan

# Destroy infrastructure
terraform destroy
```

### Inspection Commands

```bash
# Show resources and outputs
terraform show

# List resources in configuration
terraform state list

# Show specific resource
terraform state show aws_instance.web

# Output values
terraform output

# Get specific output
terraform output instance_id
```

### Debugging Commands

```bash
# Enable debug logging
export TF_LOG=DEBUG
terraform apply

# Save debug logs to file
export TF_LOG_PATH=terraform.log
terraform apply

# Validate JSON configuration
terraform validate -json

# Check graph
terraform graph

# Check graph in visual format
terraform graph | dot -Tsvg > graph.svg
```

---

## Best Practices

### Code Organization

✅ **DO:**
- Organize by module (vpc, rds, ec2, etc.)
- Separate main.tf, variables.tf, outputs.tf
- Use consistent naming conventions
- Keep modules small and focused
- Document modules with README files

❌ **DON'T:**
- Put everything in one file
- Use unclear variable names
- Mix concerns in modules
- Create overly complex modules
- Ignore documentation

### State Management

✅ **DO:**
- Use remote state for team environments
- Enable state encryption
- Implement state locking
- Backup state regularly
- Use workspaces for environments
- Never commit state to Git

❌ **DON'T:**
- Use local state in production
- Share state files manually
- Edit state files directly
- Commit terraform.tfstate to Git
- Ignore state locking errors
- Delete state without backups

### Security

✅ **DO:**
- Use IAM roles instead of access keys
- Encrypt sensitive outputs
- Validate input variables
- Use secrets management (AWS Secrets Manager, HashiCorp Vault)
- Review plans before applying
- Implement least privilege

❌ **DON'T:**
- Hardcode credentials in code
- Use root credentials
- Store secrets in state
- Apply changes without review
- Skip validation
- Over-provision permissions

### Development Workflow

✅ **DO:**
- Use version control (Git)
- Create branches for features
- Use pull requests for reviews
- Test changes in dev workspace
- Plan before apply
- Document changes

❌ **DON'T:**
- Apply directly to production
- Skip code reviews
- Use master branch for development
- Make manual changes outside Terraform
- Ignore warnings
- Deploy untested code

---

## Troubleshooting

### Common Issues

#### Issue 1: Provider Authentication Failed

```bash
# Solution: Set AWS credentials
export AWS_ACCESS_KEY_ID="xxx"
export AWS_SECRET_ACCESS_KEY="xxx"
export AWS_REGION="us-east-1"

# Or use credentials file
# ~/.aws/credentials

# Or use IAM role (recommended for production)
```

#### Issue 2: State Lock Error

```bash
# View locks
terraform force-unlock <LOCK_ID>

# Or remove DynamoDB table entry manually
# This should only be done if lock is stuck
```

#### Issue 3: Resource Already Exists

```bash
# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0

# Or remove from state
terraform state rm aws_instance.web
# Then create with terraform apply
```

#### Issue 4: Plan Shows Unwanted Changes

```bash
# Review specific resource
terraform state show aws_instance.web

# Check for drift
terraform refresh

# Ignore certain attributes
lifecycle {
  ignore_changes = [tags["LastModified"]]
}
```

#### Issue 5: Backend Initialization Failed

```bash
# Reconfigure backend
terraform init -reconfigure

# Migrate state
terraform init -migrate-state

# Use local backend temporarily
terraform init -backend=false
```

---

## Continuous Integration/Deployment

### GitHub Actions Example

```yaml
# .github/workflows/terraform.yml

name: Terraform

on:
  push:
    branches: [ main ]
    paths: [ 'terraform/**' ]
  pull_request:
    branches: [ main ]
    paths: [ 'terraform/**' ]

jobs:
  terraform:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.6.0
    
    - name: Initialize Terraform
      run: terraform -chdir=terraform init
    
    - name: Validate
      run: terraform -chdir=terraform validate
    
    - name: Format Check
      run: terraform -chdir=terraform fmt -check -recursive
    
    - name: Plan
      run: terraform -chdir=terraform plan -out=tfplan
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    
    - name: Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform -chdir=terraform apply -auto-approve tfplan
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## Resources

### Official Documentation
- [Terraform Official Docs](https://www.terraform.io/docs/)
- [Terraform Registry](https://registry.terraform.io/)
- [AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform GitHub](https://github.com/hashicorp/terraform)

### Learning Resources
- [Terraform Tutorials](https://learn.hashicorp.com/terraform)
- [Terraform Associate Certification](https://www.hashicorp.com/certification/terraform-associate)
- [Infrastructure as Code Best Practices](https://www.hashicorp.com/blog)
- [Community Modules](https://registry.terraform.io/modules)

### Tools
- [Terraform Linting](https://github.com/terraform-linters/tflint)
- [Security Scanning](https://www.checkov.io/)
- [Cost Estimation](https://www.infracost.io/)
- [Documentation](https://terraform-docs.io/)

---

## Quick Start Template

```bash
# 1. Create directory
mkdir my-terraform && cd my-terraform

# 2. Create main.tf
cat > main.tf << 'EOF'
terraform {
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  tags = { Name = "WebServer" }
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}
EOF

# 3. Create variables.tf
cat > variables.tf << 'EOF'
variable "aws_region" {
  default = "us-east-1"
}

variable "instance_type" {
  default = "t2.micro"
}
EOF

# 4. Create outputs.tf
cat > outputs.tf << 'EOF'
output "instance_id" {
  value = aws_instance.web.id
}

output "instance_ip" {
  value = aws_instance.web.public_ip
}
EOF

# 5. Initialize and apply
terraform init
terraform plan
terraform apply
```

---

## Next Steps

1. **Learn HCL**: Master Terraform configuration language
2. **Create Modules**: Build reusable infrastructure components
3. **Set Up Remote State**: Configure team collaboration
4. **Implement CI/CD**: Automate infrastructure deployment
5. **Use Workspaces**: Manage multiple environments
6. **Monitor Costs**: Use Infracost for budget tracking
7. **Security Scan**: Use Checkov for compliance

---

## Support

- **Documentation**: https://www.terraform.io/docs/
- **Community Forum**: https://discuss.hashicorp.com/c/terraform/
- **GitHub Issues**: https://github.com/hashicorp/terraform/issues
- **HashiCorp Support**: https://www.hashicorp.com/support

---

**Happy Infrastructure Coding! 🚀**

---

*Last Updated: March 15, 2026*  
*Version: 1.0.0*  
*Status: Production Ready*
