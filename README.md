# Terraform-code
**To make the Terraform code more reusable, we can parameterize the resource creation and provide flexibility for users to customize the setup according to their needs. Here's the modified version of the code**
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

variable "subnet_id" {
  description = "Subnet ID for EC2 instances"
  default     = "subnet-12345678" # Enter the appropriate subnet ID
}

variable "db_password" {
  description = "Password for the RDS instance"
  default     = "password"
}

# Jenkins EC2 instance
resource "aws_instance" "jenkins" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  key_name      = var.ssh_key_name

  tags = {
    Name = "Jenkins"
  }
}

# SonarQube EC2 instance
resource "aws_instance" "sonarqube" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  key_name      = var.ssh_key_name

  tags = {
    Name = "SonarQube"
  }
}

# Prometheus EC2 instance
resource "aws_instance" "prometheus" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  key_name      = var.ssh_key_name

  tags = {
    Name = "Prometheus"
  }
}

# Grafana EC2 instance
resource "aws_instance" "grafana" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
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

output "jira_db_endpoint" {
  value = aws_db_instance.jira_db.endpoint
}

output "eks_cluster_endpoint" {
  value = aws_eks_cluster.my_cluster.endpoint
}
```
In this modified code:

I've introduced variables for the SSH key pair name, AMI ID, instance type, subnet ID, and RDS database password. Users can customize these values when deploying the infrastructure.
The resource definitions now use these variables, making the code more reusable and adaptable to different environments.
Users can provide their own values for these variables according to their specific requirements when deploying the Terraform configuration.
This version of the code allows for greater flexibility and reusability, as it can be easily customized for different scenarios without needing to modify the resource definitions directly.
