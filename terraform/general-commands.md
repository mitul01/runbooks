# Terraform & Terragrunt Commands Runbook

## Terraform Commands

### Basic Workflow Commands

#### Initialize Terraform
```bash
# Initialize a working directory containing Terraform configuration files
terraform init

# Initialize with backend reconfiguration
terraform init -reconfigure

# Initialize without backend initialization
terraform init -backend=false

# Upgrade provider versions
terraform init -upgrade
```

#### Plan Changes
```bash
# Create an execution plan
terraform plan

# Save plan to a file
terraform plan -out=tfplan

# Plan with variable file
terraform plan -var-file="terraform.tfvars"

# Plan for specific resource
terraform plan -target=resource_type.resource_name

# Destroy plan
terraform plan -destroy
```

#### Apply Changes
```bash
# Apply the configuration
terraform apply

# Apply a saved plan
terraform apply tfplan

# Auto-approve without interactive approval
terraform apply -auto-approve

# Apply with variable override
terraform apply -var="instance_type=t3.micro"

# Apply to specific target
terraform apply -target=aws_instance.example
```

#### Destroy Resources
```bash
# Destroy all resources
terraform destroy

# Auto-approve destroy
terraform destroy -auto-approve

# Destroy specific resource
terraform destroy -target=aws_instance.example
```

### State Management Commands

#### State Inspection
```bash
# List resources in state
terraform state list

# Show detailed state information
terraform state show

# Show specific resource state
terraform state show aws_instance.example

# Pull remote state
terraform state pull
```

#### State Manipulation
```bash
# Import existing resource into state
terraform import aws_instance.example i-1234567890abcdef0

# Remove resource from state (without destroying)
terraform state rm aws_instance.example

# Move resource in state
terraform state mv aws_instance.old_name aws_instance.new_name

# Push local state to remote
terraform state push terraform.tfstate
```

### Validation and Formatting
```bash
# Validate configuration
terraform validate

# Format configuration files
terraform fmt

# Format and check for changes
terraform fmt -check

# Recursively format files
terraform fmt -recursive
```

### Output Commands
```bash
# Show all outputs
terraform output

# Show specific output
terraform output instance_ip

# Output in JSON format
terraform output -json

# Show raw output (no quotes)
terraform output -raw instance_ip
```

### Provider Commands
```bash
# Show provider requirements
terraform providers

# Lock provider versions
terraform providers lock

# Mirror providers for offline use
terraform providers mirror /path/to/mirror
```

## Terragrunt Commands

### Basic Terragrunt Workflow

#### Initialize and Plan
```bash
# Initialize with Terragrunt
terragrunt init

# Plan with Terragrunt
terragrunt plan

# Plan all modules in directory tree
terragrunt run-all plan

# Plan with output to file
terragrunt plan -out=tfplan
```

#### Apply Changes
```bash
# Apply configuration
terragrunt apply

# Apply all modules
terragrunt run-all apply

# Auto-approve apply
terragrunt apply -auto-approve

# Apply all with auto-approve
terragrunt run-all apply --terragrunt-non-interactive
```

#### Destroy Resources
```bash
# Destroy resources
terragrunt destroy

# Destroy all modules
terragrunt run-all destroy

# Destroy with auto-approve
terragrunt destroy -auto-approve
```

### Terragrunt-Specific Commands

#### Dependency Management
```bash
# Show dependency graph
terragrunt graph-dependencies

# Apply dependencies first
terragrunt apply-all

# Destroy in reverse dependency order
terragrunt destroy-all
```

#### Configuration Management
```bash
# Validate all configurations
terragrunt run-all validate

# Format all configurations
terragrunt hclfmt

# Show effective configuration
terragrunt terragrunt-info

# Render JSON configuration
terragrunt render-json
```

#### State Operations
```bash
# Refresh state for all modules
terragrunt run-all refresh

# Output values from all modules
terragrunt run-all output

# Show state for all modules
terragrunt run-all state list
```

### Advanced Terragrunt Commands
```bash
# Run command with specific working directory
terragrunt --terragrunt-working-dir /path/to/module plan

# Include/exclude specific modules
terragrunt run-all apply --terragrunt-include-dir dev/
terragrunt run-all apply --terragrunt-exclude-dir prod/

# Parallelism control
terragrunt run-all apply --terragrunt-parallelism 5

# Source map for debugging
terragrunt --terragrunt-source-map plan
```

## Troubleshooting Commands

### Debug Information
```bash
# Enable debug logging
export TF_LOG=DEBUG
terraform apply

# Terragrunt debug
terragrunt apply --terragrunt-log-level debug

# Show configuration
terraform show

# Show providers
terraform providers
```

### State Issues
```bash
# Refresh state
terraform refresh

# Force unlock state
terraform force-unlock LOCK_ID

# Backup state
cp terraform.tfstate terraform.tfstate.backup

# Import missing resource
terraform import aws_instance.example i-1234567890abcdef0
```

### Common Fixes
```bash
# Fix corrupted state
terraform state pull > terraform.tfstate.backup
terraform state push terraform.tfstate.backup

# Reset to clean state
rm -rf .terraform
terraform init

# Force provider re-download
terraform init -upgrade

# Clear Terragrunt cache
rm -rf .terragrunt-cache
```

## Useful Aliases

Add these to your shell configuration:

```bash
alias tf='terraform'
alias tg='terragrunt'
alias tfi='terraform init'
alias tfp='terraform plan'
alias tfa='terraform apply'
alias tfd='terraform destroy'
alias tfv='terraform validate'
alias tff='terraform fmt'
alias tfs='terraform show'
alias tfo='terraform output'

alias tgi='terragrunt init'
alias tgp='terragrunt plan'
alias tga='terragrunt apply'
alias tgd='terragrunt destroy'
alias tgpa='terragrunt run-all plan'
alias tgaa='terragrunt run-all apply'
alias tgda='terragrunt run-all destroy'
```
