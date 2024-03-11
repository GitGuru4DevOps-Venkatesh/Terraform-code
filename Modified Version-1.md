# Below is the modified version-1 of Terraform code with Kubernetes setup and Docker setup using Ubuntu commands.
## If Company or Users need to include Docker & Kubernetes Setup instead of using AWS_Cluster_EKS Set-up in the Terraform Code, then only can use this. 
### Or else you can use modified terraform code only..!

```
# Define AWS provider
provider "aws" {
  region = "us-east-1"  # Set the AWS region to us-east-1
}

# Variables
variable "ssh_key_name" {
  description = "Name of the SSH key pair"  # Description of the variable
  default     = "your_ssh_key_name"         # Default value for the SSH key name
}

variable "ami_id" {
  description = "AMI ID for EC2 instances"   # Description of the variable
  default     = "ami-12345678"               # Default AMI ID (should be replaced with appropriate value)
}

variable "instance_type" {
  description = "Instance type for EC2 instances"  # Description of the variable
  default     = "t2.micro"                         # Default instance type
}

# Create VPC
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"  # Define the CIDR block for the VPC
}

# Create public subnet
resource "aws_subnet" "public_subnet" {
  vpc_id            = aws_vpc.my_vpc.id        # Reference to the VPC ID
  cidr_block        = "10.0.1.0/24"             # Define the CIDR block for the subnet
  availability_zone = "us-east-1a"              # Specify the availability zone
}

# Create internet gateway
resource "aws_internet_gateway" "my_igw" {
  vpc_id = aws_vpc.my_vpc.id   # Reference to the VPC ID
}

# Attach internet gateway to VPC
resource "aws_vpc_attachment" "igw_attachment" {
  vpc_id             = aws_vpc.my_vpc.id             # Reference to the VPC ID
  internet_gateway_id = aws_internet_gateway.my_igw.id  # Reference to the internet gateway ID
}

# Create route table
resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.my_vpc.id   # Reference to the VPC ID

  route {
    cidr_block = "0.0.0.0/0"   # Route all traffic to the internet gateway
    gateway_id = aws_internet_gateway.my_igw.id   # Reference to the internet gateway ID
  }
}

# Associate public subnet with route table
resource "aws_route_table_association" "public_subnet_association" {
  subnet_id      = aws_subnet.public_subnet.id          # Reference to the public subnet ID
  route_table_id = aws_route_table.public_route_table.id  # Reference to the route table ID
}

# Jenkins EC2 instance
resource "aws_instance" "jenkins" {
  ami           = var.ami_id             # Reference to the AMI ID variable
  instance_type = var.instance_type      # Reference to the instance type variable
  subnet_id     = aws_subnet.public_subnet.id   # Reference to the public subnet ID
  key_name      = var.ssh_key_name       # Reference to the SSH key name variable

  tags = {
    Name = "Jenkins"   # Tag the instance with the name "Jenkins"
  }
}

# SonarQube EC2 instance
resource "aws_instance" "sonarqube" {
  ami           = var.ami_id             # Reference to the AMI ID variable
  instance_type = var.instance_type      # Reference to the instance type variable
  subnet_id     = aws_subnet.public_subnet.id   # Reference to the public subnet ID
  key_name      = var.ssh_key_name       # Reference to the SSH key name variable

  tags = {
    Name = "SonarQube"  # Tag the instance with the name "SonarQube"
  }
}

# Prometheus EC2 instance
resource "aws_instance" "prometheus" {
  ami           = var.ami_id             # Reference to the AMI ID variable
  instance_type = var.instance_type      # Reference to the instance type variable
  subnet_id     = aws_subnet.public_subnet.id   # Reference to the public subnet ID
  key_name      = var.ssh_key_name       # Reference to the SSH key name variable

  tags = {
    Name = "Prometheus"  # Tag the instance with the name "Prometheus"
  }
}

# Grafana EC2 instance
resource "aws_instance" "grafana" {
  ami           = var.ami_id             # Reference to the AMI ID variable
  instance_type = var.instance_type      # Reference to the instance type variable
  subnet_id     = aws_subnet.public_subnet.id   # Reference to the public subnet ID
  key_name      = var.ssh_key_name       # Reference to the SSH key name variable

  tags = {
    Name = "Grafana"   # Tag the instance with the name "Grafana"
  }
}

# RDS instance for Jira database
resource "aws_db_instance" "jira_db" {
  allocated_storage    = 20            # Allocate 20 GB of storage
  engine              = "mysql"       # Use MySQL as the database engine
  engine_version      = "5.7"         # MySQL version 5.7
  instance_class      = "db.t2.micro"  # Use t2.micro instance class
  name                = "jira-db"     # Name of the database instance
  username            = "admin"       # Username for database access
  password            = var.db_password  # Reference to the database password variable
  publicly_accessible = false          # Database is not publicly accessible
}

# Kubernetes setup
resource "aws_instance" "kubernetes_master" {
  ami           = var.ami_id             # Reference to the AMI ID variable
  instance_type = var.instance_type      # Reference to the instance type variable
  subnet_id     = aws_subnet.public_subnet.id   # Reference to the public subnet ID
  key_name      = var.ssh_key_name       # Reference to the SSH key name variable

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",   # Update package repositories
      "sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common",  # Install dependencies
      "curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -",   # Add Docker's official GPG key
      "sudo add-apt-repository \"deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable\"",  # Add Docker repository
      "sudo apt-get update",   # Update package repositories again
      "sudo apt-get install -y docker-ce",   # Install Docker
      "sudo usermod -aG docker ubuntu",      # Add the current user to the docker group
      "sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -",   # Add Google Cloud's official GPG key
      "sudo add-apt-repository \"deb http://apt.kubernetes.io/ kubernetes-xenial main\"",   # Add Kubernetes repository
      "sudo apt-get update",   # Update package repositories again
      "sudo apt-get install -y kubelet kubeadm kubectl",   # Install Kubernetes components
      "sudo apt-mark hold kubelet kubeadm kubectl",   # Mark Kubernetes packages as held to prevent automatic updates
    ]
  }

  tags = {
    Name = "Kubernetes-Master"   # Tag the instance with the name "Kubernetes-Master"
  }
}

resource "aws_instance" "kubernetes_worker" {
  ami           = var.ami_id             # Reference to the AMI ID variable
  instance_type = var.instance_type      # Reference to the instance type variable
  subnet_id     = aws_subnet.public_subnet.id   # Reference to the public subnet ID
  key_name      = var.ssh_key_name       # Reference to the SSH key name variable

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",   # Update package repositories
      "sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common",  # Install dependencies
      "curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -",   # Add Docker's official GPG key
      "sudo add-apt-repository \"deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable\"",  # Add Docker repository
      "sudo apt-get update",   # Update package repositories again
      "sudo apt-get install -y docker-ce",   # Install Docker
      "sudo usermod -aG docker ubuntu",      # Add the current user to the docker group
      "sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -",   # Add Google Cloud's official GPG key
      "sudo add-apt-repository \"deb http://apt.kubernetes.io/ kubernetes-xenial main\"",   # Add Kubernetes repository
      "sudo apt-get update",   # Update package repositories again
      "sudo apt-get install -y kubelet kubeadm kubectl",   # Install Kubernetes components
      "sudo apt-mark hold kubelet kubeadm kubectl",   # Mark Kubernetes packages as held to prevent automatic updates
    ]
  }

  tags = {
    Name = "Kubernetes-Worker"   # Tag the instance with the name "Kubernetes-Worker"
  }
}

# Outputs
output "jenkins_ip" {
  value = aws_instance.jenkins.private_ip   # Output the private IP of the Jenkins instance
}

output "sonarqube_ip" {
  value = aws_instance.sonarqube.private_ip   # Output the private IP of the SonarQube instance
}

output "prometheus_ip" {
  value = aws_instance.prometheus.private_ip   # Output the private IP of the Prometheus instance
}

output "grafana_ip" {
  value = aws_instance.grafana.private_ip   # Output the private IP of the Grafana instance
}

output "jira_db_endpoint" {
  value = aws_db_instance.jira_db.endpoint   # Output the endpoint of the Jira database instance
}

output "kubernetes_master_ip" {
  value = aws_instance.kubernetes_master.private_ip   # Output the private IP of the Kubernetes master instance
}

output "kubernetes_worker_ip" {
  value = aws_instance.kubernetes_worker.private_ip   # Output the private IP of the Kubernetes worker instance
}

output "vpc_id" {
  value = aws_vpc.my_vpc.id   # Output the ID of the VPC
}

output "public_subnet_id" {
  value = aws_subnet.public_subnet.id   # Output the ID of the public subnet
}
```

##This is the line-by-line explanation of the Terraform code:

1. **Provider Definition:**
   - Defines the AWS provider with the specified region as `us-east-1`.

2. **Variables:**
   - Declares three variables: `ssh_key_name`, `ami_id`, and `instance_type` with default values. These variables are customizable.

3. **Create VPC:**
   - Creates a Virtual Private Cloud (VPC) with the CIDR block `10.0.0.0/16`.

4. **Create Public Subnet:**
   - Creates a public subnet within the VPC with CIDR block `10.0.1.0/24` and located in availability zone `us-east-1a`.

5. **Create Internet Gateway:**
   - Creates an internet gateway for the VPC.

6. **Attach Internet Gateway to VPC:**
   - Attaches the internet gateway created in the previous step to the VPC.

7. **Create Route Table:**
   - Creates a route table for the VPC and adds a route to send all traffic (`0.0.0.0/0`) to the internet gateway.

8. **Associate Public Subnet with Route Table:**
   - Associates the public subnet created earlier with the route table, enabling internet access for instances in the subnet.

9. **Create Jenkins EC2 Instance:**
   - Launches an EC2 instance using the specified AMI, instance type, and SSH key in the public subnet. Tags the instance with the name "Jenkins".

10. **Create SonarQube EC2 Instance:**
    - Launches another EC2 instance for SonarQube with similar configurations and tags it appropriately.

11. **Create Prometheus EC2 Instance:**
    - Launches an EC2 instance for Prometheus monitoring with similar configurations and tags.

12. **Create Grafana EC2 Instance:**
    - Launches an EC2 instance for Grafana dashboard with similar configurations and tags.

13. **Create RDS Instance for Jira Database:**
    - Sets up an RDS MySQL instance named "jira-db" with specified configurations, including allocated storage, engine version, and access settings.

14. **Kubernetes Setup:**
    - Launches EC2 instances for Kubernetes master and worker nodes with provisioning steps for installing Docker and Kubernetes components using remote-exec provisioner.

15. **Outputs:**
    - Defines outputs to display important information like private IPs of the instances, RDS endpoint, VPC ID, and public subnet ID.

This Terraform script automates the setup of a basic infrastructure on AWS, including networking components, EC2 instances for various services, an RDS database, and Kubernetes nodes.

This script sets up an AWS infrastructure including a VPC, public subnet, internet gateway, route table, various EC2 instances for Jenkins, SonarQube, Prometheus, Grafana, and Kubernetes, an RDS MySQL database instance for Jira, and provides outputs for essential information such as instance IPs, RDS endpoint, VPC ID, and subnet ID.
