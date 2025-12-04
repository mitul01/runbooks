# Terraform Loops and Iteration Patterns

This document covers various loop patterns in Terraform for creating multiple resources efficiently.

## 1. Count Loop

### Basic Count Usage
```hcl
# Create multiple EC2 instances
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-server-${count.index}"
  }
}

# Reference instances created with count
output "instance_ips" {
  value = aws_instance.web[*].private_ip
}
```

### Count with Conditional Logic
```hcl
variable "create_staging" {
  type    = bool
  default = false
}

resource "aws_instance" "staging" {
  count         = var.create_staging ? 2 : 0
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
  
  tags = {
    Name = "staging-${count.index + 1}"
    Environment = "staging"
  }
}
```

## 2. For_Each Loop

### For_Each with Set
```hcl
variable "users" {
  type = set(string)
  default = ["alice", "bob", "charlie"]
}

resource "aws_iam_user" "users" {
  for_each = var.users
  name     = each.value
  
  tags = {
    Department = "Engineering"
  }
}

# Access each user
output "user_arns" {
  value = {
    for user_name, user in aws_iam_user.users : user_name => user.arn
  }
}
```

### For_Each with Map
```hcl
variable "environments" {
  type = map(object({
    instance_type = string
    min_size      = number
    max_size      = number
  }))
  default = {
    dev = {
      instance_type = "t2.micro"
      min_size      = 1
      max_size      = 2
    }
    prod = {
      instance_type = "t3.medium"
      min_size      = 2
      max_size      = 10
    }
  }
}

resource "aws_autoscaling_group" "app" {
  for_each = var.environments
  
  name                = "app-asg-${each.key}"
  vpc_zone_identifier = var.subnet_ids
  min_size           = each.value.min_size
  max_size           = each.value.max_size
  
  launch_template {
    id      = aws_launch_template.app[each.key].id
    version = "$Latest"
  }
  
  tag {
    key                 = "Environment"
    value               = each.key
    propagate_at_launch = true
  }
}
```

## 3. Dynamic Blocks
### Dynamic S3 Bucket Configuration
```hcl
resource "aws_s3_bucket_lifecycle_configuration" "buckets" {
  for_each = var.s3_buckets
  bucket   = aws_s3_bucket.buckets[each.key].id
  
  dynamic "rule" {
    for_each = each.value.lifecycle_rules
    content {
      id     = rule.value.id
      status = rule.value.enabled ? "Enabled" : "Disabled"
      
      expiration {
        days = rule.value.expiration_days
      }
      
      noncurrent_version_expiration {
        noncurrent_days = rule.value.noncurrent_expiration_days
      }
    }
  }
}
```

## 4. For Expressions

### Transform Lists
```hcl
variable "server_names" {
  type = list(string)
  default = ["web1", "web2", "db1"]
}

# Create uppercase server names
locals {
  uppercase_names = [for name in var.server_names : upper(name)]
  
  # Filter and transform
  web_servers = [for name in var.server_names : name if substr(name, 0, 3) == "web"]
  
  # Create name-index mapping
  server_indices = {
    for idx, name in var.server_names : name => idx
  }
}

output "transformed_names" {
  value = {
    uppercase = local.uppercase_names
    web_only  = local.web_servers
    indices   = local.server_indices
  }
}
```

### Complex Transformations
```hcl
variable "users" {
  type = map(object({
    role        = string
    permissions = list(string)
    active      = bool
  }))
  default = {
    alice = {
      role        = "admin"
      permissions = ["read", "write", "delete"]
      active      = true
    }
    bob = {
      role        = "user"
      permissions = ["read"]
      active      = true
    }
    charlie = {
      role        = "user"
      permissions = ["read", "write"]
      active      = false
    }
  }
}

locals {
  # Get only active users
  active_users = {
    for name, user in var.users : name => user if user.active
  }
  
  # Create policy documents for each user
  user_policies = {
    for name, user in local.active_users : name => {
      actions = [for perm in user.permissions : "s3:${title(perm)}Object"]
      role    = user.role
    }
  }
  
  # Flatten permissions for all users
  all_permissions = flatten([
    for name, user in var.users : [
      for perm in user.permissions : {
        user       = name
        permission = perm
        role       = user.role
      }
    ]
  ])
}
```

## 5. Nested Loops

### Multi-Level Resource Creation
```hcl
variable "regions" {
  type = map(object({
    availability_zones = list(string)
    instance_types     = list(string)
  }))
  default = {
    us-east-1 = {
      availability_zones = ["us-east-1a", "us-east-1b"]
      instance_types     = ["t2.micro", "t2.small"]
    }
    us-west-2 = {
      availability_zones = ["us-west-2a", "us-west-2b"]
      instance_types     = ["t3.micro", "t3.small"]
    }
  }
}

locals {
  # Create all combinations of regions, AZs, and instance types
  instance_combinations = flatten([
    for region, config in var.regions : [
      for az in config.availability_zones : [
        for instance_type in config.instance_types : {
          key           = "${region}-${az}-${instance_type}"
          region        = region
          az            = az
          instance_type = instance_type
        }
      ]
    ]
  ])
  
  # Convert to map for for_each
  instances = {
    for combo in local.instance_combinations : combo.key => combo
  }
}

resource "aws_instance" "multi_region" {
  for_each = local.instances
  
  ami               = data.aws_ami.amazon_linux[each.value.region].id
  instance_type     = each.value.instance_type
  availability_zone = each.value.az
  
  tags = {
    Name   = "instance-${each.key}"
    Region = each.value.region
    AZ     = each.value.az
  }
}
```

## 6. Advanced Patterns

### Conditional Resource Creation with Loops
```hcl
variable "environments" {
  type = map(object({
    create_database = bool
    create_cache    = bool
    instance_count  = number
  }))
  default = {
    dev = {
      create_database = false
      create_cache    = false
      instance_count  = 1
    }
    prod = {
      create_database = true
      create_cache    = true
      instance_count  = 3
    }
  }
}

# Create databases only for environments that need them
resource "aws_db_instance" "main" {
  for_each = {
    for env, config in var.environments : env => config
    if config.create_database
  }
  
  identifier     = "db-${each.key}"
  engine         = "mysql"
  instance_class = "db.t3.micro"
  
  tags = {
    Environment = each.key
  }
}

# Create cache clusters conditionally
resource "aws_elasticache_cluster" "main" {
  for_each = {
    for env, config in var.environments : env => config
    if config.create_cache
  }
  
  cluster_id      = "cache-${each.key}"
  engine          = "redis"
  node_type       = "cache.t3.micro"
  
  tags = {
    Environment = each.key
  }
}
```

### Loop with Dependencies
```hcl
# First create VPCs
resource "aws_vpc" "main" {
  for_each = var.environments
  
  cidr_block           = "10.${index(keys(var.environments), each.key)}.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "vpc-${each.key}"
  }
}

# Then create subnets in each VPC
resource "aws_subnet" "main" {
  for_each = {
    for env, config in var.environments : env => config
  }
  
  vpc_id            = aws_vpc.main[each.key].id
  cidr_block        = "10.${index(keys(var.environments), each.key)}.1.0/24"
  availability_zone = data.aws_availability_zones.available.names[0]
  
  tags = {
    Name = "subnet-${each.key}"
  }
}

# Finally create instances using the subnets
resource "aws_instance" "app" {
  for_each = {
    for env, config in var.environments : env => config
  }
  
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main[each.key].id
  
  tags = {
    Name = "app-${each.key}"
  }
}
```