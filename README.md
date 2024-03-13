# Terraform-code
### If you have already a VPC and subnets set up in your AWS account. Then only, User or Company can use this Terraform Code. If not, you can use modified Terraform code file.

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

Here, can see clear explanation:

AWS Provider Configuration: This block defines the provider for AWS and specifies the region to deploy resources in.
```
provider "aws" {
  region = "us-east-1"
}
```

Variables: These are parameters that can be customized when running Terraform. They provide flexibility and can be used to abstract away details that may change across deployments.
```
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
```
AWS EC2 Instances: These are resources representing virtual machines in the AWS cloud.

```
resource "aws_instance" "jenkins" {
  // Configuration for Jenkins EC2 instance
}

resource "aws_instance" "sonarqube" {
  // Configuration for SonarQube EC2 instance
}

resource "aws_instance" "prometheus" {
  // Configuration for Prometheus EC2 instance
}

resource "aws_instance" "grafana" {
  // Configuration for Grafana EC2 instance
}
```

AWS RDS Instance: This is a resource representing a relational database service instance.
```
resource "aws_db_instance" "jira_db" {
  // Configuration for Jira database RDS instance
}
```
AWS EKS Cluster: This resource defines an Elastic Kubernetes Service cluster.
```
resource "aws_eks_cluster" "my_cluster" {
  // Configuration for EKS cluster
}
```
Outputs: These provide information about the deployed resources once Terraform has applied the configuration.
```
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
```
## Explanation:

### This file describes the infrastructure to be provisioned on AWS using 'Terraform'. 
### It includes EC2 instances for Jenkins, SonarQube, Prometheus, and Grafana, an RDS instance for the Jira database, and an EKS cluster for Kubernetes. 
### The output blocks are used to display important information such as IP addresses and endpoints after Terraform has applied the configuration. 
### Ensure that the values provided for variables like ami_id, subnet_id, and role_arn are appropriate for your AWS environment. 
### Additionally, you may want to customize other parameters like instance types, database settings, and security configurations according to your requirements.
------------------------------------------------------------------------------------------
## Step-by-step guide on how to use Terraform to provision the infrastructure described in the code provided:

### Step 1: Install Terraform
If you haven't already, you need to install Terraform on your local machine. You can download Terraform from the [official website](https://www.terraform.io/downloads.html) and follow the installation instructions for your operating system.

### Step 2: Set Up AWS Credentials
Ensure that you have AWS credentials configured on your machine. You can set up credentials using the AWS CLI or by setting environment variables (`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`).

### Step 3: Create a Terraform Configuration File
Create a file with a `.tf` extension (e.g., `main.tf`) and paste the Terraform configuration provided in your question into this file.

### Step 4: Customize Variables
Review the variables declared in the configuration file (`variables.tf`) and customize them according to your requirements. For example, you may need to specify your SSH key name, appropriate AMI IDs, subnet IDs, etc.

### Step 5: Initialize Terraform
Open a terminal or command prompt, navigate to the directory where your Terraform configuration file is located, and run the following command to initialize Terraform:

```
terraform init
```

This command initializes Terraform and downloads any necessary plugins.

### Step 6: Plan the Infrastructure
Run the following command to see what Terraform plans to do without actually executing any actions:

```
terraform plan
```

This command will show you a summary of the actions Terraform will take based on your configuration.

### Step 7: Apply the Configuration
If the plan looks good, you can apply the Terraform configuration to provision the infrastructure on AWS. Run the following command:

```
terraform apply
```

Terraform will prompt you to confirm the plan. Type `yes` and press Enter to proceed.

### Step 8: Monitor Progress
Terraform will start provisioning the infrastructure. You'll see output indicating the progress of each resource being created. This process may take some time depending on the complexity of your configuration.

### Step 9: Verify Resources
Once Terraform has finished applying the configuration, you can verify that the resources have been created successfully by logging in to the AWS Management Console or by using the AWS CLI.

### Step 10: Destroy Resources (Optional)
If you want to tear down the infrastructure provisioned by Terraform, you can run the following command:

```
terraform destroy
```

This will destroy all the resources defined in your Terraform configuration. Be cautious when using this command, as it will permanently delete your infrastructure.

### Additional Tips:
- Remember to save your Terraform configuration files in a version control system (e.g., Git) for tracking changes.
- Make sure to keep sensitive information like AWS access keys and passwords secure and avoid committing them to version control.

By following these steps, you'll be able to use Terraform to provision infrastructure on AWS as described in your configuration file.
