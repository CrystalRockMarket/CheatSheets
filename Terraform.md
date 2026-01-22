# Terraform Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Basic Concepts](#3-basic-concepts)
4. [Configuration](#4-configuration)
5. [Resources](#5-resources)
6. [Variables](#6-variables)
7. [Modules](#7-modules)
8. [State](#8-state)
9. [Provisioners](#9-provisioners)
10. [Workspaces](#10-workspaces)
11. [Remote Backend](#11-remote-backend)
12. [Functions](#12-functions)
13. [CLI Commands](#13-cli-commands)
14. [Best Practices](#14-best-practices)
15. [Quick Reference](#15-quick-reference)

---

## 1. Introduction

### What is Terraform?
Terraform is an open-source infrastructure as code (IaC) tool that enables you to define and provision infrastructure using declarative configuration files.

### Key Features
| Feature | Description |
|---------|-------------|
| **Declarative** | Define desired state, not steps |
| **Multi-cloud** | AWS, Azure, GCP, and 100+ providers |
| **Immutable** | Infrastructure as code |
| **Plan/Apply** | Preview changes before applying |
| **State Management** | Tracks infrastructure |
| **Modular** | Reusable modules |
| **Version Control** | Infrastructure versioning |

### Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                     Terraform Workflow                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Write      │  │    Plan      │  │    Apply     │         │
│  │  .tf files   │──▶│  terraform   │──▶│  terraform   │         │
│  │              │  │   plan       │  │   apply      │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                    Terraform State                       │ │
│  │  terraform.tfstate (local or remote)                     │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│    AWS       │  │    Azure     │  │     GCP      │  │  Other      │
│  Provider    │  │   Provider   │  │   Provider   │  │  Providers  │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
```

---

## 2. Installation

### Linux/macOS
```bash
# Using tfenv (recommended)
git clone https://github.com/tfutils/tfenv.git ~/.tfenv
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bashrc
tfenv install 1.6.0
tfenv use 1.6.0

# Direct download
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Verify
terraform --version
```

### macOS with Homebrew
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

### Windows
```bash
# Using Chocolatey
choco install terraform

# Or download from terraform.io
```

### Verify Installation
```bash
terraform --version
# Terraform v1.6.0

terraform -help
```

---

## 3. Basic Concepts

### Provider Configuration
```hcl
# Configure AWS provider
provider "aws" {
  region = "us-east-1"
  profile = "default"
  
  # Alternative: Use environment variables
  # AWS_ACCESS_KEY_ID
  # AWS_SECRET_ACCESS_KEY
  # AWS_DEFAULT_REGION
}

# Configure Azure provider
provider "azurerm" {
  features {
    key_vault {
      purge_soft_delete_on_destroy = true
    }
  }
}

# Configure multiple providers
provider "aws" {
  alias  = "east"
  region = "us-east-1"
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"
}
```

### Resource Declaration
```hcl
# Create AWS EC2 instance
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name        = "web-server"
    Environment = "production"
  }
}

# Create Azure resource group
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
}
```

### Terraform Workflow
```bash
# Initialize Terraform
terraform init

# Create execution plan
terraform plan

# Apply changes
terraform apply

# Destroy infrastructure
terraform destroy

# Format configuration
terraform fmt

# Validate configuration
terraform validate

# Show infrastructure state
terraform show
```

---

## 4. Configuration

### Block Types
```hcl
# Provider block
provider "aws" {
  region = "us-east-1"
}

# Resource block
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

# Variable block
variable "instance_type" {
  type        = string
  default     = "t2.micro"
  description = "EC2 instance type"
}

# Output block
output "instance_id" {
  value       = aws_instance.example.id
  description = "EC2 instance ID"
}

# Module block
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
}

# Data source block
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-*-amd64-*"]
  }
}
```

### Resource Dependencies
```hcl
# Implicit dependency (automatic)
resource "aws_instance" "example" {
  ami               = data.aws_ami.ubuntu.id
  instance_type     = "t2.micro"
  # Depends on aws_ami.ubuntu automatically
}

# Explicit dependency
resource "aws_ebs_volume" "example" {
  availability_zone = aws_instance.example.availability_zone
  size              = 100
}

resource "aws_volume_attachment" "example" {
  device_name = "/dev/sdh"
  volume_id   = aws_ebs_volume.example.id
  instance_id = aws_instance.example.id
  
  # Explicit dependency (rarely needed)
  depends_on = [aws_ebs_volume.example]
}
```

---

## 5. Resources

### AWS EC2
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  # Networking
  subnet_id              = "subnet-12345678"
  vpc_security_group_ids = ["sg-12345678"]
  key_name               = "my-key-pair"
  
  # Storage
  root_block_device {
    volume_size = 20
    volume_type = "gp3"
  }
  
  # Tags
  tags = {
    Name        = "web-server"
    Environment = "production"
  }
  
  # User data (cloud-init)
  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World!" > /var/www/html/index.html
              EOF
}
```

### AWS S3 Bucket
```hcl
resource "aws_s3_bucket" "example" {
  bucket = "my-unique-bucket-name"
  
  versioning {
    enabled = true
  }
  
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
  
  lifecycle_rule {
    enabled = true
    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }
  }
}

resource "aws_s3_bucket_acl" "example" {
  bucket = aws_s3_bucket.example.id
  acl    = "private"
}
```

### AWS VPC
```hcl
resource "aws_vpc" "example" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "production"
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.example.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet"
  }
}

resource "aws_subnet" "private" {
  vpc_id     = aws_vpc.example.id
  cidr_block = "10.0.2.0/24"
  
  tags = {
    Name = "private-subnet"
  }
}

resource "aws_internet_gateway" "example" {
  vpc_id = aws_vpc.example.id
  
  tags = {
    Name = "igw"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.example.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.example.id
  }
  
  tags = {
    Name = "public-route-table"
  }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

### Azure Resources
```hcl
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
}

resource "azurerm_virtual_network" "example" {
  name                = "example-network"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  address_space       = ["10.0.0.0/16"]
}

resource "azurerm_subnet" "example" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_network_interface" "example" {
  name                = "example-nic"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.example.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_linux_virtual_machine" "example" {
  name                = "example-vm"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  size                = "Standard_F2"
  admin_username      = "adminuser"
  
  network_interface_ids = [
    azurerm_network_interface.example.id
  ]
  
  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }
  
  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
}
```

### Kubernetes
```hcl
resource "kubernetes_namespace" "example" {
  metadata {
    name = "production"
  }
}

resource "kubernetes_deployment" "example" {
  metadata {
    name = "nginx"
    namespace = kubernetes_namespace.example.metadata[0].name
  }
  
  spec {
    replicas = 3
    
    selector {
      match_labels = {
        app = "nginx"
      }
    }
    
    template {
      metadata {
        labels = {
          app = "nginx"
        }
      }
      
      spec {
        container {
          name  = "nginx"
          image = "nginx:1.21"
          
          port {
            container_port = 80
          }
          
          resources {
            requests = {
              cpu    = "100m"
              memory = "128Mi"
            }
            limits = {
              cpu    = "500m"
              memory = "256Mi"
            }
          }
        }
      }
    }
  }
}

resource "kubernetes_service" "example" {
  metadata {
    name = "nginx"
    namespace = kubernetes_namespace.example.metadata[0].name
  }
  
  spec {
    selector = {
      app = kubernetes_deployment.example.spec[0].selector[0].match_labels.app
    }
    
    port {
      port        = 80
      target_port = 80
    }
    
    type = "LoadBalancer"
  }
}
```

---

## 6. Variables

### Variable Types
```hcl
# String
variable "instance_type" {
  type        = string
  default     = "t2.micro"
  description = "EC2 instance type"
}

# Number
variable "count" {
  type        = number
  default     = 3
}

# Boolean
variable "enable_monitoring" {
  type        = bool
  default     = true
}

# List
variable "availability_zones" {
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

# Map
variable "instance_config" {
  type = map(string)
  default = {
    instance_type = "t2.micro"
    ami           = "ami-0c55b159cbfafe1f0"
  }
}

# Object
variable "server" {
  type = object({
    cpu    = number
    memory = number
    storage = number
  })
  default = {
    cpu    = 2
    memory = 4096
    storage = 100
  }
}

# Tuple
variable "ports" {
  type    = tuple([number, number, number])
  default = [80, 443, 8080]
}

# Set
variable "regions" {
  type    = set(string)
  default = ["us-east-1", "us-west-2"]
}
```

### Variable Files
```bash
# terraform.tfvars (auto-loaded)
instance_type = "t2.micro"
enable_monitoring = true

# custom.tfvars (manual)
terraform apply -var-file="custom.tfvars"

# .auto.tfvars (auto-loaded with .auto. prefix)
instance_type = "t2.micro"
```

### Sensitive Variables
```hcl
variable "api_key" {
  type        = string
  description = "API key for service"
  sensitive   = true
}

# Mark output as sensitive
output "api_endpoint" {
  value       = "https://api.example.com"
  description = "API endpoint"
  sensitive   = true
}
```

### Local Values
```hcl
locals {
  # Compute local values
  name_prefix = "${var.environment}-${var.project}"
  
  # Merge maps
  common_tags = merge(
    var.tags,
    {
      Project   = var.project
      ManagedBy = "Terraform"
    }
  )
  
  # Conditional
  instance_name = var.environment == "production" ? "prod-${var.name}" : "dev-${var.name}"
}
```

---

## 7. Modules

### Module Structure
```
modules/
└── networking/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

### Module Usage
```hcl
# Using module
module "vpc" {
  source  = "./modules/networking"
  version = "1.0.0"
  
  # Input variables
  name                 = "production-vpc"
  cidr_block           = "10.0.0.0/16"
  enable_nat_gateway   = true
  single_nat_gateway   = true
  
  # Tags
  tags = {
    Environment = "production"
  }
}

# Using module from registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway     = true
  single_nat_gateway     = true
  enable_dns_hostnames   = true
  enable_dns_support     = true
  
  tags = {
    Terraform = "true"
    Environment = "production"
  }
}
```

### Module Variables
```hcl
# modules/networking/variables.tf
variable "name" {
  type        = string
  description = "Name of the VPC"
}

variable "cidr_block" {
  type        = string
  description = "CIDR block for VPC"
}

variable "enable_nat_gateway" {
  type        = bool
  default     = false
  description = "Enable NAT gateway"
}

variable "tags" {
  type        = map(string)
  default     = {}
  description = "Tags to apply to resources"
}
```

### Module Outputs
```hcl
# modules/networking/outputs.tf
output "vpc_id" {
  value       = aws_vpc.main.id
  description = "ID of the VPC"
}

output "private_subnet_ids" {
  value       = [for subnet in aws_subnet.private : subnet.id]
  description = "IDs of private subnets"
}

output "public_subnet_ids" {
  value       = [for subnet in aws_subnet.public : subnet.id]
  description = "IDs of public subnets"
}
```

---

## 8. State

### Local State
```bash
# Default: terraform.tfstate
terraform apply
# Creates terraform.tfstate
```

### State Commands
```bash
# Show state
terraform show

# List resources
terraform state list

# Show specific resource
terraform state show aws_instance.web

# Move resource
terraform state mv aws_instance.web aws_instance.web_server

# Remove resource from state
terraform state rm aws_instance.web

# Pull state
terraform state pull > terraform.tfstate

# Push state
terraform state push terraform.tfstate

# Replace provider
terraform state replace-provider registry.terraform.io/-/aws aws

# View state
terraform state list
```

### State Locking
```hcl
# terraform.tfstate is automatically locked
# Use remote backend for locking
```

---

## 9. Provisioners

### Local-Exec
```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  provisioner "local-exec" {
    command     = "echo ${self.public_ip} > ip_address.txt"
    working_dir = "./scripts"
    interpreter = ["bash", "-c"]
    environment = {
      INSTANCE_ID = self.id
    }
  }
}
```

### Remote-Exec
```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
  
  provisioner "remote-exec" {
    inline = [
      "sudo yum install -y nginx",
      "sudo systemctl enable nginx",
      "sudo systemctl start nginx",
      "echo 'Hello from Terraform!' > /usr/share/nginx/html/index.html"
    ]
  }
}
```

### File Provisioner
```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
  
  provisioner "file" {
    source      = "./scripts/deploy.sh"
    destination = "/tmp/deploy.sh"
  }
  
  provisioner "remote-exec" {
    inline = ["chmod +x /tmp/deploy.sh", "/tmp/deploy.sh"]
  }
}
```

### Destroy-Time Provisioner
```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  provisioner "local-exec" {
    when        = destroy
    command     = "echo 'Instance ${self.instance_id} is being destroyed'"
    on_failure  = continue
  }
}
```

---

## 10. Workspaces

### Workspace Commands
```bash
# List workspaces
terraform workspace list

# Show current workspace
terraform workspace show

# Create workspace
terraform workspace new production

# Select workspace
terraform workspace select development

# Delete workspace
terraform workspace delete development
```

### Workspace Usage
```hcl
variable "environment" {
  type    = string
  default = "development"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = terraform.workspace == "production" ? "t2.large" : "t2.micro"
  
  tags = {
    Environment = terraform.workspace
    Workspace   = terraform.workspace
  }
}
```

---

## 11. Remote Backend

### S3 Backend
```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "production/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
    acl            = "bucket-owner-full-control"
  }
}
```

### Azure Storage Backend
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state"
    storage_account_name = "tfstate123"
    container_name       = "tfstate"
    key                  = "production.terraform.tfstate"
  }
}
```

### GCS Backend
```hcl
terraform {
  backend "gcs" {
    bucket = "my-terraform-state"
    prefix = "production"
  }
}
```

### State Locking with DynamoDB
```hcl
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

---

## 12. Functions

### Numeric Functions
```hcl
# abs
variable "num" { default = -5 }
output "abs" { value = abs(var.num) }  # 5

# min/max
output "min_val" { value = min(1, 5, 3) }  # 1
output "max_val" { value = max(1, 5, 3) }  # 5

# floor/ceil
output "floor" { value = floor(3.7) }  # 3
output "ceil" { value = ceil(3.2) }    # 4

# pow
output "pow" { value = pow(2, 3) }     # 8
```

### String Functions
```hcl
# lower/upper
output "lower" { value = lower("HELLO") }  # "hello"
output "upper" { value = upper("hello") }  # "HELLO"

# split/join
output "split" { value = split(",", "a,b,c") }  # ["a", "b", "c"]
output "join" { value = join(",", ["a", "b", "c"]) }  # "a,b,c"

# substr
output "substr" { value = substr("hello", 1, 3) }  # "ell"

# replace
output "replace" { value = replace("hello", "l", "x") }  # "hexxo"

# trimspace
output "trim" { value = trimspace("  hello  ") }  # "hello"

# format
output "format" { value = format("Instance: %s, ID: %d", "web", 123) }
```

### Collection Functions
```hcl
# length
output "len" { value = length(["a", "b", "c"]) }  # 3
output "len_str" { value = length("hello") }      # 5
output "len_map" { value = length({"a" = 1, "b" = 2}) }  # 2

# index
output "index" { value = index(["a", "b", "c"], "b") }  # 1

# slice
output "slice" { value = slice(["a", "b", "c", "d"], 1, 3) }  # ["b", "c"]

# concat
output "concat" { value = concat(["a", "b"], ["c", "d"]) }  # ["a", "b", "c", "d"]

# flatten
output "flatten" { value = flatten([["a", "b"], ["c"]]) }  # ["a", "b", "c"]

# merge
output "merge" { value = merge({"a" = 1}, {"b" = 2}) }  # {"a" = 1, "b" = 2}

# keys/values
output "keys" { value = keys({"a" = 1, "b" = 2}) }  # ["a", "b"]
output "values" { value = values({"a" = 1, "b" = 2}) }  # [1, 2]

# element
output "element" { value = element(["a", "b", "c"], 1) }  # "b"

# contains
output "contains" { value = contains(["a", "b", "c"], "b") }  # true

# sort
output "sort" { value = sort(["c", "a", "b"]) }  # ["a", "b", "c"]

# distinct
output "distinct" { value = distinct(["a", "b", "a", "c"]) }  # ["a", "b", "c"]

# setproduct
output "product" { value = setproduct(["a", "b"], [1, 2]) }
# [["a", 1], ["a", 2], ["b", 1], ["b", 2]]
```

### Type Conversion
```hcl
# tostring
output "to_string" { value = tostring(42) }  # "42"

# tonumber
output "to_number" { value = tonumber("42") }  # 42

# tobool
output "to_bool" { value = tobool("true") }  # true

# toset/tolist/tomap
output "tolist" { value = tolist(["a", "b"]) }
output "toset" { value = toset(["a", "b"]) }
output "tomap" { value = tomap({"a" = 1, "b" = 2}) }
```

---

## 13. CLI Commands

### Initialization
```bash
# Initialize Terraform
terraform init

# Initialize with specific backend
terraform init -backend-config="config.tfbackend"

# Upgrade providers
terraform init -upgrade

# Reinitialize (after config changes)
terraform init -reconfigure
```

### Planning
```bash
# Create execution plan
terraform plan

# Plan with variables
terraform plan -var="instance_type=t2.micro"

# Plan with var file
terraform plan -var-file="production.tfvars"

# Save plan
terraform plan -out=tfplan

# Plan destroy
terraform plan -destroy

# Show plan graphically
terraform plan -graph | dot -Tpng > plan.png
```

### Applying
```bash
# Apply changes
terraform apply

# Apply with auto-approve
terraform apply -auto-approve

# Apply saved plan
terraform apply tfplan

# Apply specific target
terraform apply -target=aws_instance.web

# Apply with variables
terraform apply -var="env=production"
```

### Destroying
```bash
# Destroy all infrastructure
terraform destroy

# Destroy with auto-approve
terraform destroy -auto-approve

# Destroy specific target
terraform destroy -target=aws_instance.web
```

### State Management
```bash
# List resources in state
terraform state list

# Show resource details
terraform state show aws_instance.web

# Move resource
terraform state mv aws_instance.web aws_instance.web_server

# Remove from state
terraform state rm aws_instance.web

# Pull remote state
terraform state pull

# Push state
terraform state push terraform.tfstate

# Replace provider
terraform state replace-provider registry.terraform.io/-/aws hashicorp/aws

# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0
```

### Workspaces
```bash
# List workspaces
terraform workspace list

# Show current workspace
terraform workspace show

# Create workspace
terraform workspace new production

# Select workspace
terraform workspace select development

# Delete workspace
terraform workspace delete development
```

### Other Commands
```bash
# Format Terraform files
terraform fmt

# Format with recursion
terraform fmt -recursive

# Validate configuration
terraform validate

# Show providers
terraform providers

# Show graph
terraform graph | dot -Tpng > graph.png

# Console (interactive)
terraform console

# Import existing infrastructure
terraform import aws_instance.web i-1234567890abcdef0

# Get modules
terraform get

# Get specific version
terraform get -update

# Taint resource (force recreation)
terraform taint aws_instance.web

# Untaint resource
terraform untaint aws_instance.web

# Show documentation
terraform show -help

# Version
terraform version

# Upgrade Terraform
terraform version -upgrade
```

---

## 14. Best Practices

### Project Structure
```
terraform/
├── environments/
│   ├── production/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── terraform.tfvars
│   └── staging/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── terraform.tfvars
├── modules/
│   ├── networking/
│   ├── compute/
│   ├── database/
│   └── storage/
├── shared/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
└── backend.tf
```

### Naming Conventions
```hcl
# Resources: type_name
resource "aws_instance" "web_server" {}
resource "aws_s3_bucket" "logs_bucket" {}

# Variables: descriptive_name
variable "instance_type" {}
variable "enable_monitoring" {}

# Outputs: descriptive_name
output "instance_id" {}
output "bucket_endpoint" {}

# Modules: provider_service
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
}

# Files: purpose.tf
main.tf      # Main configuration
variables.tf # Variable declarations
outputs.tf   # Output declarations
backend.tf   # Backend configuration
```

### Security Best Practices
```bash
# Use remote state with encryption
terraform {
  backend "s3" {
    encrypt        = true
    bucket         = "encrypted-terraform-state"
    dynamodb_table = "terraform-locks"
  }
}

# Don't commit secrets
# .gitignore
*.tfstate
*.tfvars
*.secret
*.pem
*.key

# Use variable sensitivity
variable "api_key" {
  type      = string
  sensitive = true
}

# Use least privilege IAM policies
data "aws_iam_policy_document" "minimal" {
  statement {
    actions   = ["ec2:Describe*"]
    resources = ["*"]
  }
}
```

### Performance Tips
```bash
# Use -parallelism flag
terraform apply -parallelism=20

# Split large configurations
# Use modules for reusable components

# Use data sources efficiently
# Avoid wildcards when possible

# Enable caching
# Use remote state
```

---

## 15. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Initialize | `terraform init` |
| Plan changes | `terraform plan` |
| Apply changes | `terraform apply` |
| Destroy | `terraform destroy` |
| Format code | `terraform fmt` |
| Validate | `terraform validate` |
| List state | `terraform state list` |
| Import resource | `terraform import` |
| Taint resource | `terraform taint` |
| Workspace list | `terraform workspace list` |

### Common Functions
| Function | Example | Result |
|----------|---------|--------|
| `file` | `file("config.txt")` | File contents |
| `templatefile` | `templatefile("conf.tmpl", {var=value})` | Rendered template |
| `uuid` | `uuid()` | Random UUID |
| `timestamp` | `timestamp()` | Current timestamp |
| `formatdate` | `formatdate("YYYY-MM-DD", timestamp())` | Formatted date |
| `base64encode` | `base64encode("data")` | Base64 encoded |
| `base64decode` | `base64decode("ZGF0YQ==")` | Base64 decoded |
| `sha256` | `sha256("data")` | SHA256 hash |
| `bcrypt` | `bcrypt("password")` | bcrypt hash |

### Block Types
| Block | Purpose |
|-------|---------|
| `terraform` | Configuration and settings |
| `provider` | Provider configuration |
| `resource` | Infrastructure resource |
| `data` | Data source |
| `variable` | Input variable |
| `output` | Output value |
| `module` | Module reference |
| `locals` | Local values |

### Import Syntax
```bash
# Import AWS EC2
terraform import aws_instance.web i-1234567890abcdef0

# Import AWS S3 bucket
terraform import aws_s3_bucket.my-bucket my-bucket-name

# Import Azure resource group
terraform import azurerm_resource_group.example /subscriptions/.../resourceGroups/example

# Import with config
terraform import -config=import.tfvars aws_instance.web i-1234567890abcdef0
```

### Exit Codes
| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
| 2 | Generic error |
| 3 | User interrupt |

---

*Last Updated: January 2026*
*Generated for Terraform 1.6.x*
