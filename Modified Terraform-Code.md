# If you don't have a VPC and subnets set up in your AWS account. User or Company can use this Terraform Code.

## Below is the modified Terraform code that includes the VPC, subnet, and route table setup along with the existing resources:

```
# Define AWS provider
provider "aws" {
  region = "us-east-1"
}

# Variables
variable "ssh_key_name" {
  description = "Name of the SSH key pair"
  default     = "your_ssh_key_name"
}

variable "ami_id" {
  description = "AMI ID for EC2 instances"
  default     = "ami-12345678" # Enter the appropriate AMI ID
}

variable "instance_type" {
  description = "Instance type for EC2 instances"
  default     = "t2.micro"
}

# Create VPC
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
}

# Create public subnet
resource "aws_subnet" "public_subnet" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}

# Create internet gateway
resource "aws_internet_gateway" "my_igw" {
  vpc_id = aws_vpc.my_vpc.id
}

# Attach internet gateway to VPC
resource "aws_vpc_attachment" "igw_attachment" {
  vpc_id             = aws_vpc.my_vpc.id
  internet_gateway_id = aws_internet_gateway.my_igw.id
}

# Create route table
resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.my_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.my_igw.id
  }
}

# Associate public subnet with route table
resource "aws_route_table_association" "public_subnet_association" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_route_table.id
}

# Jenkins EC2 instance
resource "aws_instance" "jenkins" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public_subnet.id
  key_name      = var.ssh_key_name

  tags = {
    Name = "Jenkins"
  }
}

# SonarQube EC2 instance
resource "aws_instance" "sonarqube" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public_subnet.id
  key_name      = var.ssh_key_name

  tags = {
    Name = "SonarQube"
  }
}

# Prometheus EC2 instance
resource "aws_instance" "prometheus" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public_subnet.id
  key_name      = var.ssh_key_name

  tags = {
    Name = "Prometheus"
  }
}

# Grafana EC2 instance
resource "aws_instance" "grafana" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public_subnet.id
  key_name      = var.ssh_key_name

  tags = {
    Name = "Grafana"
  }
}

# RDS instance for Jira database
resource "aws_db_instance" "jira_db" {
  allocated_storage    = 20
  engine              = "mysql"
  engine_version      = "5.7"
  instance_class      = "db.t2.micro"
  name                = "jira-db"
  username            = "admin"
  password            = var.db_password
  publicly_accessible = false
}

# EKS cluster
resource "aws_eks_cluster" "my_cluster" {
  name     = "my-cluster"
  role_arn = "arn:aws:iam::123456789012:role/eks-service-role-AWSServiceRoleForAmazonEKS-EXAMPLE"
  version  = "1.20"
}

# Outputs
output "jenkins_ip" {
  value = aws_instance.jenkins.private_ip
}

output "sonarqube_ip" {
  value = aws_instance.sonarqube.private_ip
}

output "prometheus_ip" {
  value = aws_instance.prometheus.private_ip
}

output "grafana_ip" {
  value = aws_instance.grafana.private_ip
}

output "jira_db_endpoint" {
  value = aws_db_instance.jira_db.endpoint
}

output "eks_cluster_endpoint" {
  value = aws_eks_cluster.my_cluster.endpoint
}

output "vpc_id" {
  value = aws_vpc.my_vpc.id
}

output "public_subnet_id" {
  value = aws_subnet.public_subnet.id
}
```

### In this code:

We've added resources to create a VPC, a public subnet, an internet gateway, a route table, and associated them appropriately.

The Jenkins, SonarQube, Prometheus, and Grafana instances are now placed in the public subnet created within the VPC.
Outputs are provided for the Jenkins instance IP, SonarQube instance IP, Prometheus instance IP, Grafana instance IP, RDS endpoint, EKS cluster endpoint, VPC ID, and public subnet ID for reference.

This code provides a complete setup including networking infrastructure and the provisioned resources using Terraform.
