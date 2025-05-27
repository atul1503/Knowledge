Notes:

1. **Terraform is like a blueprint for your cloud stuff.** You write what you want, and it creates/updates/destroys resources to match that blueprint.

2. **Always start with provider configuration.** This tells Terraform which cloud provider to use and how to connect to it.
   ```hcl
   terraform {                    # Main Terraform configuration block
     required_providers {         # Which providers this code needs
       aws = {                    # Name you'll use to reference this provider
         source  = "hashicorp/aws"     # Where to download the provider from
         version = "~> 5.0"            # Which version to use (~> means "5.x but not 6.x")
       }
     }
   }

   provider "aws" {               # Configure the AWS provider
     region  = "us-east-1"        # Which AWS region to create resources in
     profile = "default"          # Which AWS profile from ~/.aws/credentials to use
   }
   ```

3. **Resources are the actual things you want to create.** Like servers, databases, networks, etc.
   ```hcl
   resource "aws_instance" "my_server" {    # resource "type" "name"
     ami           = "ami-0abcdef1234567890" # Which machine image to use
     instance_type = "t3.micro"             # What size/type of server
     
     tags = {                               # Labels to put on this resource
       Name = "MyWebServer"                 # What to call it in AWS console
       Environment = "dev"                  # Which environment this belongs to
     }
   }
   ```

4. **Variables let you reuse values and make your code flexible.** Define them once, use them everywhere.
   ```hcl
   variable "instance_type" {     # variable "name"
     description = "EC2 instance type"      # What this variable is for
     type        = string                   # What kind of data (string, number, bool, list, map)
     default     = "t3.micro"               # Default value if none provided
   }

   variable "environment" {
     description = "Environment name"
     type        = string
     # No default means this MUST be provided
   }

   # Use variables like this
   resource "aws_instance" "web" {
     instance_type = var.instance_type      # var.variable_name
     tags = {
       Environment = var.environment
     }
   }
   ```

5. **Outputs let you get information about created resources.** Like IP addresses, URLs, etc.
   ```hcl
   output "server_ip" {           # output "name"
     description = "Public IP of the web server"    # What this output contains
     value       = aws_instance.web.public_ip       # resource_type.resource_name.attribute
   }

   output "server_url" {
     value = "http://${aws_instance.web.public_ip}"  # ${} for string interpolation
   }
   ```

6. **Data sources let you look up existing resources.** Like finding an AMI or getting VPC info.
   ```hcl
   data "aws_ami" "ubuntu" {      # data "type" "name"
     most_recent = true           # Get the newest one
     owners      = ["099720109477"]    # Canonical's AWS account ID

     filter {                     # Search criteria
       name   = "name"            # Filter by name
       values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
     }
   }

   # Use it like this
   resource "aws_instance" "web" {
     ami = data.aws_ami.ubuntu.id    # data.type.name.attribute
   }
   ```

7. **Local values are like variables but calculated from other values.** Good for complex expressions.
   ```hcl
   locals {                       # locals block
     common_tags = {              # local.common_tags
       Project     = "MyApp"
       Environment = var.environment
       Owner       = "DevTeam"
     }
     
     server_name = "${var.environment}-web-server"    # local.server_name
   }

   resource "aws_instance" "web" {
     tags = local.common_tags     # Use with local.name
   }
   ```

8. **Use terraform commands to manage your infrastructure:**
   ```bash
   terraform init        # Download providers and set up backend
   terraform plan        # Show what changes will be made (dry run)
   terraform apply       # Actually make the changes
   terraform destroy     # Delete everything
   terraform fmt         # Format your code nicely
   terraform validate    # Check for syntax errors
   ```

9. **State file tracks what Terraform has created.** Don't delete `terraform.tfstate` or you'll lose track of your resources.
   ```hcl
   # Store state remotely for team collaboration
   terraform {
     backend "s3" {               # backend "type"
       bucket = "my-terraform-state"      # S3 bucket name
       key    = "prod/terraform.tfstate"  # Path within bucket
       region = "us-east-1"               # Bucket region
     }
   }
   ```

10. **Modules are reusable pieces of Terraform code.** Like functions but for infrastructure.
    ```hcl
    module "web_server" {         # module "name"
      source = "./modules/web"    # Where to find the module code
      
      # Pass variables to the module
      instance_type = "t3.small"
      environment   = "prod"
      server_count  = 3
    }

    # Use outputs from modules
    output "web_ips" {
      value = module.web_server.instance_ips    # module.name.output_name
    }
    ```

11. **Count creates multiple copies of the same resource.**
    ```hcl
    resource "aws_instance" "web" {
      count         = 3           # Create 3 instances
      ami           = data.aws_ami.ubuntu.id
      instance_type = "t3.micro"
      
      tags = {
        Name = "web-server-${count.index}"    # count.index gives 0, 1, 2
      }
    }

    # Reference specific instances
    output "first_server_ip" {
      value = aws_instance.web[0].public_ip    # [index] to get specific one
    }
    ```

12. **for_each creates resources based on a map or set.** More flexible than count.
    ```hcl
    variable "servers" {
      default = {
        web = "t3.micro"
        api = "t3.small"
        db  = "t3.medium"
      }
    }

    resource "aws_instance" "servers" {
      for_each      = var.servers           # for_each = map_or_set
      ami           = data.aws_ami.ubuntu.id
      instance_type = each.value            # each.value = map value
      
      tags = {
        Name = each.key                     # each.key = map key
      }
    }
    ```

13. **Conditionals let you create resources only when needed.**
    ```hcl
    resource "aws_instance" "web" {
      count = var.create_server ? 1 : 0    # Create only if variable is true
      # ... rest of config
    }

    # Or with for_each
    resource "aws_s3_bucket" "backup" {
      for_each = var.environment == "prod" ? toset(["backup"]) : toset([])
      bucket   = "my-app-backup-${each.key}"
    }
    ```

14. **Dependencies control the order things get created.** Terraform usually figures this out automatically.
    ```hcl
    resource "aws_vpc" "main" {
      cidr_block = "10.0.0.0/16"
    }

    resource "aws_subnet" "web" {
      vpc_id     = aws_vpc.main.id    # Implicit dependency - subnet needs VPC first
      cidr_block = "10.0.1.0/24"
    }

    resource "aws_instance" "web" {
      subnet_id = aws_subnet.web.id
      depends_on = [aws_internet_gateway.main]    # Explicit dependency
    }
    ```

15. **Lifecycle rules control how resources are updated or replaced.**
    ```hcl
    resource "aws_instance" "web" {
      ami           = data.aws_ami.ubuntu.id
      instance_type = var.instance_type
      
      lifecycle {
        create_before_destroy = true    # Create new one before destroying old
        prevent_destroy       = true    # Don't allow terraform destroy on this
        ignore_changes       = [ami]    # Don't update if AMI changes
      }
    }
    ```

16. **Provisioners run commands on resources after they're created.** Use sparingly - prefer cloud-init or user data.
    ```hcl
    resource "aws_instance" "web" {
      ami           = data.aws_ami.ubuntu.id
      instance_type = "t3.micro"
      key_name      = aws_key_pair.deployer.key_name

      provisioner "remote-exec" {       # Run commands on the remote machine
        inline = [                      # List of commands to run
          "sudo apt update",
          "sudo apt install -y nginx",
          "sudo systemctl start nginx"
        ]

        connection {                    # How to connect to the machine
          type        = "ssh"           # Connection type
          user        = "ubuntu"        # Username to connect as
          private_key = file("~/.ssh/id_rsa")    # SSH private key
          host        = self.public_ip  # self refers to this resource
        }
      }
    }
    ```

17. **Workspaces let you manage multiple environments with the same code.**
    ```bash
    terraform workspace new dev      # Create new workspace
    terraform workspace new prod
    terraform workspace list        # See all workspaces
    terraform workspace select dev  # Switch to workspace
    ```
    
    ```hcl
    # Use workspace name in your code
    resource "aws_instance" "web" {
      instance_type = terraform.workspace == "prod" ? "t3.large" : "t3.micro"
      
      tags = {
        Environment = terraform.workspace
      }
    }
    ```

18. **Common gotchas and best practices:**
    - Always run `terraform plan` before `apply` to see what will change
    - Use version constraints for providers (`~> 5.0` not `>= 5.0`)
    - Store state remotely (S3, Terraform Cloud) for team work
    - Use modules to avoid repeating code
    - Tag everything for cost tracking and organization
    - Use `terraform fmt` to keep code formatted consistently
    - Don't hardcode secrets - use AWS Secrets Manager or environment variables
    - Use data sources instead of hardcoding AMI IDs or other changing values

19. **File organization that works well:**
    ```
    main.tf         # Main resources
    variables.tf    # Variable definitions
    outputs.tf      # Output definitions
    versions.tf     # Provider requirements
    terraform.tfvars # Variable values (don't commit secrets!)
    ```

20. **Example: Complete web server setup**
    ```hcl
    # variables.tf
    variable "environment" {
      description = "Environment name"
      type        = string
      default     = "dev"
    }

    # main.tf
    data "aws_ami" "ubuntu" {
      most_recent = true
      owners      = ["099720109477"]
      filter {
        name   = "name"
        values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
      }
    }

    resource "aws_security_group" "web" {
      name_prefix = "${var.environment}-web-"
      
      ingress {
        from_port   = 80
        to_port     = 80
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }
      
      egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
      }
    }

    resource "aws_instance" "web" {
      ami                    = data.aws_ami.ubuntu.id
      instance_type          = "t3.micro"
      vpc_security_group_ids = [aws_security_group.web.id]
      
      user_data = <<-EOF
        #!/bin/bash
        apt update
        apt install -y nginx
        systemctl start nginx
        systemctl enable nginx
      EOF
      
      tags = {
        Name        = "${var.environment}-web-server"
        Environment = var.environment
      }
    }

    # outputs.tf
    output "web_url" {
      description = "URL of the web server"
      value       = "http://${aws_instance.web.public_ip}"
    }
    ```
