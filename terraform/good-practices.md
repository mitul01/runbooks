# Terraform Good Practices

### Resource Naming Pattern
```
{project}-{component}-{environment}-{resource-type}
```

#### Examples
```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "terraform"
  }
}

resource "aws_security_group" "web" {
  name = "${local.name_prefix}-web-sg"
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web-sg"
    Type = "web"
  })
}
```

#### Variable Naming
- Use `snake_case` for variables and locals
- Be descriptive: `web_server_instance_type` not `type`
- Add validation where appropriate

### Use `moved` Blocks for Refactoring
```hcl
# When refactoring resource names, use moved blocks to prevent destroy/recreate
moved {
  from = aws_instance.old_name
  to   = aws_instance.new_name
}

# Moving resources between modules
moved {
  from = aws_s3_bucket.logs
  to   = module.storage.aws_s3_bucket.logs
}
```

### Dynamic Configuration with `for_each` Meta-Arguments
```hcl
# Create multiple similar resources based on map
variable "environments" {
  type = map(object({
    instance_count = number
    instance_type  = string
    subnets       = list(string)
  }))
  
  default = {
    dev = {
      instance_count = 1
      instance_type  = "t3.micro"
      subnets       = ["subnet-123"]
    }
    prod = {
      instance_count = 3
      instance_type  = "t3.large"
      subnets       = ["subnet-456", "subnet-789"]
    }
  }
}

# Use for_each instead of count for better state management
resource "aws_instance" "app" {
  for_each = var.environments
  
  count         = each.value.instance_count
  instance_type = each.value.instance_type
  subnet_id     = each.value.subnets[0]
  
  tags = {
    Name        = "${each.key}-app-${count.index + 1}"
    Environment = each.key
  }
}
```

### Check Blocks for Runtime Validation
```hcl
# Use check blocks to validate infrastructure state at runtime
check "s3_bucket_encryption" {
  data "aws_s3_bucket_encryption" "example" {
    bucket = aws_s3_bucket.example.id
  }
  
  assert {
    condition = data.aws_s3_bucket_encryption.example.rule[0].apply_server_side_encryption_by_default[0].sse_algorithm == "AES256"
    error_message = "S3 bucket must be encrypted with AES256"
  }
}

check "database_multi_az" {
  assert {
    condition = var.environment == "prod" ? aws_db_instance.main.multi_az : true
    error_message = "Production databases must be multi-AZ"
  }
}
```

### Resource Preconditions and Postconditions
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  
  # Validate inputs before creating resource
  lifecycle {
    precondition {
      condition     = data.aws_ami.amazon_linux.architecture == "x86_64"
      error_message = "AMI must be x86_64 architecture"
    }
    
    # Validate resource state after creation
    postcondition {
      condition     = self.public_ip != ""
      error_message = "Instance must have a public IP"
    }
  }
}
```

### Use `replace_triggered_by` for Controlled Recreation
```hcl
resource "aws_launch_template" "app" {
  name_prefix   = "${local.name_prefix}-"
  image_id      = data.aws_ami.app.id
  instance_type = var.instance_type
  
  # Force replacement when AMI changes
  lifecycle {
    replace_triggered_by = [
      data.aws_ami.app.id
    ]
  }
}
```

### Custom Validation Functions
```hcl
variable "vpc_cidr" {
  type        = string
  description = "VPC CIDR block"
  
  validation {
    condition = can(regex("^10\\.", var.vpc_cidr)) || can(regex("^172\\.(1[6-9]|2[0-9]|3[0-1])\\.", var.vpc_cidr)) || can(regex("^192\\.168\\.", var.vpc_cidr))
    error_message = "VPC CIDR must be within RFC 1918 private address space."
  }
  
  validation {
    condition = parseint(split("/", var.vpc_cidr)[1], 10) >= 16 && parseint(split("/", var.vpc_cidr)[1], 10) <= 28
    error_message = "VPC CIDR prefix must be between /16 and /28."
  }
}
```

### Use `terraform_data` for Triggers and Dependencies
```hcl
# Replace null_resource with terraform_data
resource "terraform_data" "app_deployment" {
  triggers_replace = [
    var.app_version,
    filesha256("${path.module}/deploy.sh")
  ]
  
  provisioner "local-exec" {
    command = "./deploy.sh ${var.app_version}"
  }
}

# Create logical dependencies
resource "terraform_data" "prerequisite_check" {
  lifecycle {
    postcondition {
      condition     = data.external.health_check.result.status == "healthy"
      error_message = "Prerequisite services are not healthy"
    }
  }
}
```

### Advanced Import Patterns
```hcl
# Use import blocks (Terraform 1.5+) for better import management
import {
  to = aws_instance.existing
  id = "i-1234567890abcdef0"
}

# Generate configuration for imported resources
resource "aws_instance" "existing" {
  # Use terraform show to get current state, then configure
}
```

### Provider Configuration Patterns
```hcl
# Use provider aliases for multi-region deployments
provider "aws" {
  alias  = "primary"
  region = var.primary_region
}

provider "aws" {
  alias  = "disaster_recovery"
  region = var.dr_region
}

# Use providers meta-argument for specific resources
resource "aws_s3_bucket" "backup" {
  provider = aws.disaster_recovery
  bucket   = "${local.name_prefix}-backup"
}
```

### Use Template Files for Complex Configurations
```hcl
# Instead of complex jsonencode(), use template files
data "template_file" "user_data" {
  template = file("${path.module}/templates/user-data.sh")
  
  vars = {
    app_version    = var.app_version
    database_host  = aws_db_instance.main.endpoint
    redis_host     = aws_elasticache_cluster.main.cache_nodes[0].address
  }
}

resource "aws_instance" "app" {
  user_data = data.template_file.user_data.rendered
  # ...
}
```

### Error Handling with Try Functions
```hcl
locals {
  # Gracefully handle missing keys
  instance_type = try(var.instance_config.type, "t3.micro")
  
  # Handle potential null values
  security_groups = try(var.security_group_ids, [])
  
  # Complex fallback logic
  database_config = try(
    var.database_config,
    {
      instance_class = "db.t3.micro"
      storage       = 20
      multi_az      = false
    }
  )
}
```
