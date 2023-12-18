## Introduction

In today’s rapidly evolving cloud infrastructure and DevOps landscape, the demand for scalable and efficient solutions is critical. Terraform, as a part of the Infrastructure as Code (IaC) movement, stands out as a robust and flexible tool for managing and deploying cloud resources. It empowers users to define their infrastructure through code, thereby introducing a new level of speed and automation in the setup process.

A key highlight of Terraform is its modular approach, embodied in Terraform modules. These modules act as reusable components, enabling you to package and replicate infrastructure elements efficiently. This not only improves the manageability of your code but also minimizes repetition. Understanding and using these modules is essential to elevate your Terraform projects.

This detailed guide is dedicated to diving into Terraform modules, shedding light on how they enhance and streamline your infrastructure coding efforts. Aimed at both newcomers and experienced Terraform practitioners, this article provides insights and practices for effectively utilizing modules. This will accelerate your infrastructure deployment and ensure its consistency.

Embark with us as we explore the transformative power of Terraform modules, unlocking new possibilities in managing your infrastructure.

## EC2 Module Structure

A typical module comprises a minimum of three distinct files:

- `main.tf`: This file is responsible for defining the resources that need to be created.
- `variables.tf`: This file specifies the variables that require values to be assigned to them by the module.
- `outputs.tf`: This file outlines the data that the module will supply, which can be utilized by other Terraform configurations.

As we move forward, we will explore the significance of each of these components.

### Setting Up the File Structure

Use the following commands in the terminal to establish your file structure:

```shell
mkdir -p modules/ec2
cd modules/ec2
touch var.tf main.tf outputs.tf
```
# EC2 Module Setup for Terraform

To begin setting up our EC2 module in Terraform, you will need to insert specific content into `.tf` files located within the `modules/ec2` directory. Start by focusing on the `var.tf` file.

## File: `modules/ec2/var.tf`

Insert the following content into the `var.tf` file located in the `modules/ec2` directory:

```hcl
variable workspace {
    type = string
    description = "infrastructure environment"
    default = "dev"
}
 
variable workspace_role {
  type = string
  description = "infrastructure role"
  default = "Cloud_Engineer"
  
}
 
variable instance_size {
    type = string
    description = "ec2 instance size"
    default = "t3.small"
}
 
variable instance_ami {
    type = string
    description = "Server image to use"
}
 
variable instance_root_device_size {
    type = number
    description = "Root bock device size in GB"
    default = 12
}
```
## File: `modules/ec2/main.tf`
```hcl
#EC2 Instance
resource "aws_instance" "web_app_1" {

  ami           = var.instance_ami
  instance_type = var.instance_size

  root_block_device {
    volume_size = var.instance_root_device_size
    volume_type = "gp3"
  }

  tags = {
    Name        = "Web-${var.workspace}-App"
    Role        = var.workspace_role
    Project     = "CloudSec.io"
    Environment = var.workspace
    ManagedBy   = "terraform"
  }
 
} 

resource "aws_eip" "web_app_addr" {

  vpc      = true
 
  lifecycle {
    prevent_destroy = false
  }
 
  tags = {
    Name        = "Web-${var.workspace}-App"
    Role        = var.workspace_role
    Project     = "CloudSec.io"
    Environment = var.workspace
    ManagedBy   = "terraform"
  }
}
 
resource "aws_eip_association" "eip_assoc" {
  instance_id   = aws_instance.web_app_1.id
  allocation_id = aws_eip.web_app_addr.id
}
```
## File: `modules/ec2/output.tf`
```hcl
output "app_eip" {
  value = aws_eip.web_app_addr.public_ip
}
 
output "app_instance" {
    value = aws_instance.web_app_1.id
}
```
# Modules are Reusable!

By modifying our root module file, `01-main.tf`, we can efficiently reuse our module multiple times:

The `00-auth.tf` file in Terraform is designed to set up and configure the AWS provider. It specifies the AWS provider version and source, ensuring compatibility and consistency in the infrastructure setup. Additionally, it configures the AWS provider to operate in a specific region, as defined by a variable, facilitating region-specific resource management.

## File: `00-Auth.tf
```hcl
terraform {
  required_providers {
    aws= {
        source = "hashicorp/aws"
        version = "~> 3.0"
    }
  }
}

provider "aws" {
   region = var.default_region
}
```
The `01-main.tf` file in Terraform is responsible for deploying two EC2 instances with distinct roles and configurations using a modular approach. It defines two modules, `ec2_app` and `ec2_worker`, both sourced from a local `ec2` module. 

- **ec2_app module**: Sets up an application server with a smaller instance size and root device.
- **ec2_worker module**: Configures a worker node with a larger capacity.

Each module is customized with specific environment variables, roles, instance sizes, AMIs, and root device sizes.

## File: `01-main.tf`
```hcl
module "ec2_app" {
   source = "./modules/ec2"
 
   workspace = var.workspace
   workspace_role = "app"
   instance_size = "t2.small"
   instance_ami = var.instance_ami
    instance_root_device_size = 12 # Optional
}
 
module "ec2_worker" {
   source = "./modules/ec2"
 
   workspace = var.workspace
   workspace_role = "worker"
   instance_size = "t3.large"
   instance_ami = var.instance_ami
   instance_root_device_size = 50
}
```
The `02-var.tf` file in Terraform is used for declaring and setting defaults for various variables used in the configuration. It includes:

- `instance_ami`: Defines the Amazon Machine Image (AMI) ID to be used for EC2 instances.
- `infra_env`: Specifies the infrastructure environment, like production or development.
- `default_region`: Determines the AWS region where resources will be provisioned.
- `instance_size`: Sets the default size for EC2 instances.

Each variable is accompanied by a description and a default value, providing clarity and fallback options for your Terraform setup.

## File: `02.var.tf`
```hcl
variable "instance_ami" {
    type = string
    description = "instance ami"
    default = "ami-0666c668000b91fcb"
}

variable workspace {
    type = string
    description = "infrastructure environment"
    default = "prod"
}
 
variable default_region {
    type = string
    description = "infrastructure region"
    default = "eu-west-1"
}
 
variable instance_size {
    type = string
    description = "ec2 instance size"
    default = "t3.small"
}
```
To begin utilizing our newly created module, we must first execute the `init` command, followed by `plan` and `apply`:

```shell
terraform init
terraform plan 
terraform apply 
```


