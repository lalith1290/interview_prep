# Terraform Interview Questions & Answers

## 1. What is Terraform, and what does it provide?
Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp. It provides:

- **Declarative Configuration**: Define infrastructure using a high-level configuration language (HCL)
- **Resource Management**: Create, update, and delete cloud resources across multiple providers
- **State Management**: Track the current state of infrastructure to plan and apply changes
- **Execution Plans**: Preview changes before applying them to infrastructure
- **Provider Ecosystem**: Support for 100+ providers including AWS, Azure, GCP, and Kubernetes
- **Modular Design**: Reusable modules for infrastructure components
- **Dependency Management**: Automatic handling of resource dependencies

Terraform enables teams to version, share, and reuse infrastructure configurations while maintaining consistency across environments.

## 2. Terraform components
The key components of Terraform include:

- **Configuration Files (.tf)**: Written in HashiCorp Configuration Language (HCL) or JSON
- **Providers**: Plugins that interface with cloud platforms, services, or APIs
- **Resources**: Infrastructure objects managed by Terraform (e.g., VMs, networks, databases)
- **Data Sources**: Read-only information fetched from providers for use in configurations
- **Variables**: Parameterize configurations for flexibility and reusability
- **State Files**: Track the current state of managed infrastructure
- **Modules**: Reusable, encapsulated configuration packages
- **Outputs**: Return values from resources for use in other configurations
- **Backend**: Storage and locking mechanism for state files
- **Provisioners**: Execute scripts or commands during resource creation or destruction

## 3. Terraform plan vs Terraform apply
**Terraform plan**:
- Creates an execution plan showing what Terraform will do when you apply
- Compares the current state with desired configuration
- Displays additions, changes, and deletions without making any changes
- Helps identify potential issues before modifying infrastructure
- Useful for code reviews and validation

**Terraform apply**:
- Executes the changes proposed in the plan
- Creates, updates, or deletes resources to match the configuration
- Updates the state file with the new infrastructure state
- By default, shows a plan and requires confirmation before making changes
- Can use `-auto-approve` flag to skip confirmation
- Handles dependencies and executes operations in the correct order

## 4. Stages in Terraform
The main stages in the Terraform workflow are:

1. **Write**: Create configuration files to define infrastructure
2. **Initialize**: Run `terraform init` to set up the working directory
   - Downloads providers
   - Sets up backend for state
   - Initializes modules
3. **Plan**: Run `terraform plan` to create an execution plan
   - Compares current state with desired state
   - Shows proposed changes
4. **Apply**: Run `terraform apply` to execute the plan
   - Creates, updates, or deletes resources
   - Updates the state file
5. **Destroy**: Run `terraform destroy` to remove all managed resources
   - Deletes resources in the proper order
   - Updates the state file

Additional stages in more complex workflows might include:
- **Validate**: Syntax and configuration validation
- **Import**: Import existing resources into Terraform management
- **Refresh**: Update the state file to match actual infrastructure

## 5. How to configure a server using Terraform?
To configure a server using Terraform, you would:

1. **Define the provider**:
```hcl
provider "aws" {
  region = "us-west-2"
}
```

2. **Create the server resource**:
```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  key_name      = "my-key-pair"
  
  vpc_security_group_ids = [aws_security_group.web_sg.id]
  subnet_id              = aws_subnet.main.id
  
  tags = {
    Name = "WebServer"
    Environment = "Production"
  }
}
```

3. **Configure networking**:
```hcl
resource "aws_security_group" "web_sg" {
  name        = "web-server-sg"
  description = "Allow web traffic"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

4. **Server provisioning** (optional):
```hcl
resource "aws_instance" "web_server" {
  # ... other configuration ...
  
  user_data = <<-EOF
              #!/bin/bash
              apt-get update -y
              apt-get install -y apache2
              systemctl start apache2
              systemctl enable apache2
              echo "Hello, Terraform!" > /var/www/html/index.html
              EOF
              
  # Or use a file
  # user_data = file("user_data.sh")
}
```

5. **Initialize and apply**:
```bash
terraform init
terraform plan
terraform apply
```

## 6. How to rename resources in Terraform?
To rename a resource in Terraform without destroying and recreating it:

1. **Use the `terraform state mv` command**:
```bash
terraform state mv aws_instance.old_name aws_instance.new_name
```

2. **Update the resource name in the configuration files**:
```hcl
# Change from
resource "aws_instance" "old_name" {
  # ...
}

# To
resource "aws_instance" "new_name" {
  # ...
}
```

3. **For any references to the resource, update them** to use the new name:
```hcl
# Change from
resource "aws_route53_record" "www" {
  # ...
  records = [aws_instance.old_name.public_ip]
}

# To
resource "aws_route53_record" "www" {
  # ...
  records = [aws_instance.new_name.public_ip]
}
```

4. **Run `terraform plan`** to verify the changes don't include unnecessary resource destruction.

## 7. How does Terraform trigger changes after editing or updating resources?
Terraform triggers changes through a multi-step process:

1. **Configuration Changes**: When you edit .tf files, Terraform doesn't immediately detect changes.

2. **State Comparison**: When you run `terraform plan` or `terraform apply`, Terraform:
   - Reads the current state file
   - Parses the updated configuration
   - Compares the desired state with the current state
   - Determines what changes are needed

3. **Change Detection**: Terraform identifies:
   - Resources that need to be created
   - Resources that need to be updated in-place
   - Resources that need to be destroyed and recreated
   - Resources that need to be destroyed

4. **Execution Plan**: Terraform creates a plan detailing all changes with proper dependency ordering.

5. **Apply Changes**: When you run `terraform apply`, Terraform:
   - Makes API calls to the provider
   - Creates, updates, or deletes resources as needed
   - Updates the state file with the new resource attributes

6. **Dependency Handling**: Terraform respects dependencies between resources and applies changes in the correct order.

7. **Force Replacement**: Some attribute changes (like changing an EC2 AMI) cannot be updated in-place and require recreation of the resource.

## 8 & 14. What is a Terraform state file?
A Terraform state file is a JSON document that:

- **Maps real-world resources to Terraform configuration**
- **Tracks metadata** such as resource dependencies
- **Stores resource attributes** for interpolation in other resources
- **Tracks resource dependencies** to determine the correct order of operations
- **Improves performance** by caching resource attributes
- **Enables collaboration** by sharing infrastructure state

Key aspects of Terraform state:

- **Default Location**: By default, stored as `terraform.tfstate` in the working directory
- **Remote State**: Can be stored remotely in S3, Azure Blob Storage, GCS, Terraform Cloud, etc.
- **State Locking**: Prevents concurrent modifications to prevent corruption
- **Sensitive Data**: May contain secrets, so should be handled securely
- **Versioning**: State files can be versioned for backup and rollback
- **Backends**: Configure where and how state is stored

Example remote state configuration:
```hcl
terraform {
  backend "s3" {
    bucket = "terraform-state-bucket"
    key    = "project/environment/terraform.tfstate"
    region = "us-east-1"
    dynamodb_table = "terraform-lock-table"
    encrypt = true
  }
}
```

## 9. What is Terraform?
Terraform is an open-source Infrastructure as Code (IaC) tool created by HashiCorp that allows you to define, provision, and manage infrastructure using a declarative configuration language. It's designed to make infrastructure management consistent, repeatable, and version-controlled.

Key features of Terraform:

- **Declarative**: You specify the desired end state, not the steps to get there
- **Platform-Agnostic**: Works with various cloud providers and services
- **Version Control**: Infrastructure definitions can be versioned in Git
- **Change Automation**: Automatically handles creating, updating, and deleting resources
- **Dependency Management**: Understands dependencies between resources
- **Modularity**: Supports reusable modules for common infrastructure patterns
- **Plan & Apply**: Preview changes before applying them
- **State Management**: Tracks the current state of your infrastructure

Terraform follows an immutable infrastructure approach, where resources are replaced rather than modified in-place when necessary.

## 10. Use of the provider in Terraform
Providers in Terraform:

- **Define the API Interaction**: Providers are plugins that interface with APIs of service providers
- **Resource Management**: Each provider offers resources that can be created, updated, and deleted
- **Authentication**: Providers handle authentication to the target platforms
- **Data Sources**: Providers expose data sources to query existing infrastructure

Example provider configuration:
```hcl
provider "aws" {
  region = "us-east-1"
  profile = "production"
  version = "~> 3.0"
}

provider "google" {
  credentials = file("account.json")
  project     = "my-project"
  region      = "us-central1"
}

provider "azurerm" {
  features {}
  subscription_id = "your-subscription-id"
}
```

Common providers include:
- **Cloud providers**: AWS, Azure, GCP, Oracle Cloud
- **Infrastructure software**: VMware, Kubernetes, Docker
- **Services**: GitHub, GitLab, Cloudflare, Datadog
- **Databases**: MySQL, PostgreSQL, MongoDB
- **Platforms**: Heroku, Fastly, Netlify

Provider versioning is important for stable infrastructure management.

## 11. Difference between Ansible and Terraform

| Feature | Terraform | Ansible |
|---------|-----------|---------|
| **Primary Purpose** | Infrastructure provisioning | Configuration management |
| **Approach** | Declarative | Procedural with declarative elements |
| **State Management** | Maintains state file | Stateless |
| **Language** | HCL (HashiCorp Configuration Language) | YAML |
| **Execution** | Creates execution plan, then applies | Executes tasks in order |
| **Idempotency** | Built-in | Requires careful module usage |
| **Mutable vs. Immutable** | Favors immutable infrastructure | Designed for mutable infrastructure |
| **Agent vs. Agentless** | Agentless | Agentless (uses SSH/WinRM) |
| **Strengths** | Creating and managing infrastructure | Configuring systems and deploying applications |
| **Extensibility** | Providers and modules | Modules and roles |
| **Learning Curve** | Moderate | Relatively easy |

**Complementary Usage**:
- Use Terraform to provision the infrastructure (VMs, networks, etc.)
- Use Ansible to configure the provisioned infrastructure (install software, configure services)

## 12. Terraform commands
Essential Terraform commands:

- **`terraform init`**: Initialize a working directory, download providers and modules
- **`terraform plan`**: Create an execution plan to show what Terraform will do
- **`terraform apply`**: Apply changes to reach the desired state
- **`terraform destroy`**: Destroy all resources managed by the configuration
- **`terraform validate`**: Validate the configuration syntax
- **`terraform fmt`**: Format the configuration files according to conventions
- **`terraform show`**: Show the current state or a saved plan
- **`terraform state list`**: List resources in the state
- **`terraform state show [resource]`**: Show details of a specific resource
- **`terraform state mv [source] [destination]`**: Move resources in the state
- **`terraform state rm [resource]`**: Remove resources from the state
- **`terraform import [resource] [id]`**: Import existing infrastructure into Terraform
- **`terraform output`**: Show output values from the state
- **`terraform refresh`**: Update the state file with actual infrastructure
- **`terraform workspace list/new/select/delete`**: Manage Terraform workspaces
- **`terraform graph`**: Generate a visual representation of dependencies
- **`terraform providers`**: Show the providers required for this configuration
- **`terraform version`**: Show the current Terraform version

## 13. Terraform modules
Terraform modules are reusable, encapsulated units of Terraform configuration that:

- **Promote code reuse** across different projects and environments
- **Abstract complexity** by encapsulating related resources
- **Enforce best practices** through standardized configurations
- **Improve maintainability** by organizing code into logical components

**Module Structure**:
```
module/
  ├── main.tf         # Main resource definitions
  ├── variables.tf    # Input variables
  ├── outputs.tf      # Output values
  ├── versions.tf     # Required providers and versions
  └── README.md       # Documentation
```

**Using a Module**:
```hcl
module "vpc" {
  source = "./modules/vpc"
  
  name = "production-vpc"
  cidr = "10.0.0.0/16"
  azs  = ["us-west-2a", "us-west-2b", "us-west-2c"]
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  subnet_id     = module.vpc.public_subnets[0]
}
```

**Module Sources**:
- Local paths
- Terraform Registry (public and private)
- GitHub and other version control systems
- S3 buckets or other HTTP URLs

**Best Practices**:
- Design modules for reusability with appropriate variables
- Version modules using semantic versioning
- Document inputs, outputs, and examples
- Use conditional creation for optional resources
- Test modules thoroughly before production use

## 15. Terraform code for an S3 bucket
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "example_bucket" {
  bucket = "my-unique-bucket-name"
  acl    = "private"
  
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
    id      = "archive-rule"
    enabled = true
    
    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }
    
    transition {
      days          = 90
      storage_class = "GLACIER"
    }
    
    expiration {
      days = 365
    }
  }
  
  tags = {
    Name        = "ExampleBucket"
    Environment = "Production"
    Terraform   = "True"
  }
}

resource "aws_s3_bucket_public_access_block" "example_bucket_public_access" {
  bucket = aws_s3_bucket.example_bucket.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

output "bucket_name" {
  value = aws_s3_bucket.example_bucket.id
}

output "bucket_arn" {
  value = aws_s3_bucket.example_bucket.arn
}
```

## 16. Integration of Jenkins with Terraform for managing AWS infrastructure
To integrate Jenkins with Terraform for AWS infrastructure management:

1. **Set up Jenkins prerequisites**:
   - Install Terraform plugin for Jenkins
   - Configure AWS credentials in Jenkins credentials store
   - Set up a Jenkins agent with Terraform installed

2. **Create a Jenkins pipeline**:
```groovy
pipeline {
    agent any
    
    environment {
        AWS_CREDENTIALS = credentials('aws-credentials-id')
        TF_IN_AUTOMATION = 'true'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Terraform Init') {
            steps {
                sh 'terraform init -input=false'
            }
        }
        
        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan -input=false'
            }
        }
        
        stage('Approval') {
            when {
                not {
                    equals expected: true, actual: params.AUTO_APPROVE
                }
            }
            steps {
                script {
                    def plan = readFile 'terraform.plan.out'
                    input message: "Do you want to apply the plan?",
                          parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                }
            }
        }
        
        stage('Terraform Apply') {
            steps {
                sh 'terraform apply -input=false tfplan'
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'tfplan', fingerprint: true
        }
    }
}
```

3. **Set up Terraform backend for collaboration**:
```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-jenkins"
    key            = "infrastructure/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}
```

4. **Best practices for the integration**:
   - Use Terraform workspaces for different environments
   - Implement proper access controls on Jenkins and AWS
   - Store Terraform code in version control
   - Use parameters in Jenkins for environment selection
   - Add validation steps before applying changes
   - Implement notifications for successful/failed deployments

5. **Advanced implementation**:
   - Use Terraform Enterprise or Terraform Cloud for better collaboration
   - Implement drift detection by scheduling regular plan jobs
   - Use Jenkins multibranch pipelines for feature branch testing
   - Add automated testing for infrastructure
   - Implement infrastructure validation after deployment
