# Terraform - Interview Q&A (3+ Years Experience)

## Infrastructure as Code Concepts

### Q1: What is Terraform and how does it differ from other IaC tools?

**Answer:**
Terraform is an Infrastructure as Code (IaC) tool by HashiCorp that lets you define and provision cloud infrastructure using code.

**Key differences from other tools:**
- **Terraform vs Ansible**: Terraform is declarative (you describe desired state) and focuses on infrastructure provisioning. Ansible is more procedural and focuses on configuration management.
- **Terraform vs CloudFormation**: Terraform is cloud-agnostic (works with Azure, AWS, GCP, etc.). CloudFormation only works with AWS.
- **Terraform vs ARM Templates**: Terraform uses HCL (HashiCorp Configuration Language) which is more readable. ARM templates are JSON which can be verbose.

**Terraform advantages:**
- Multi-cloud support
- State management
- Plan before apply (shows what will change)
- Large provider ecosystem

### Q2: Explain the Terraform workflow (init, plan, apply, destroy).

**Answer:**
1. **terraform init**: 
   - Initializes working directory
   - Downloads required providers (Azure, AWS, etc.)
   - Sets up backend for state storage
   - Run this first, or when you add new providers/modules

2. **terraform plan**:
   - Creates execution plan
   - Shows what will be created, modified, or destroyed
   - Doesn't make any changes (dry run)
   - Review this before applying

3. **terraform apply**:
   - Executes the plan
   - Creates/modifies/destroys resources
   - Asks for confirmation (unless -auto-approve)
   - Updates state file

4. **terraform destroy**:
   - Destroys all resources defined in configuration
   - Shows what will be destroyed
   - Use carefully - can delete everything

**Typical workflow**: init → plan → review → apply → verify

### Q3: What is Terraform state and why is it important?

**Answer:**
Terraform state is a file (terraform.tfstate) that tracks:
- Which resources Terraform manages
- Current properties of those resources
- Relationships between resources

**Why important:**
- **Mapping**: Maps your code to real-world resources
- **Metadata**: Stores resource attributes not available in API
- **Dependencies**: Tracks resource dependencies
- **Performance**: Speeds up operations (doesn't need to query all resources)

**State file contains sensitive data** - never commit it to version control. Use remote state (Azure Storage, S3) instead.

### Q4: What happens if you lose your Terraform state file?

**Answer:**
If you lose state file:
- **Terraform thinks resources don't exist**: It will try to create them again (will fail - resources already exist)
- **Can't update resources**: Terraform won't know what to update
- **Can't destroy resources**: Terraform won't know what exists

**Solutions:**
1. **Restore from backup**: If you have state file backup
2. **Import existing resources**: Use `terraform import` to bring existing resources into state
3. **Recreate state manually**: Very difficult and error-prone
4. **Prevention**: Always use remote state with versioning/backup enabled

**Best practice**: Store state in Azure Storage with versioning enabled, or use Terraform Cloud/Enterprise.

## Terraform State Management

### Q5: Explain remote state and why you should use it.

**Answer:**
Remote state stores your state file in a shared location (Azure Storage, S3, Terraform Cloud) instead of locally.

**Why use it:**
- **Team collaboration**: Multiple people can work on same infrastructure
- **State locking**: Prevents two people from running Terraform simultaneously (avoids conflicts)
- **Backup**: Cloud storage provides backup and versioning
- **Security**: State file not on developer machines
- **CI/CD integration**: Pipelines can access state

**Example (Azure Storage backend):**
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

### Q6: What is Terraform state locking and how does it work?

**Answer:**
State locking prevents multiple Terraform operations from running at the same time on the same state file.

**How it works:**
- When you run `terraform apply`, Terraform locks the state file
- If someone else tries to run Terraform, they get an error: "Error acquiring the state lock"
- When your operation completes, lock is released
- If operation crashes, lock might remain - you can force unlock (use carefully)

**Backend support:**
- Azure Storage: Uses blob lease (automatic)
- S3: Uses DynamoDB table (need to create table)
- Terraform Cloud: Built-in

**Why important**: Without locking, two people could modify infrastructure simultaneously, causing conflicts and corruption.

### Q7: How do you handle sensitive data in Terraform state?

**Answer:**
Terraform state can contain sensitive data (passwords, keys, connection strings). Protect it:

1. **Remote state**: Store in secure backend (Azure Storage with encryption)
2. **Encryption at rest**: Enable encryption on storage account
3. **Access control**: Limit who can read state file (RBAC)
4. **State file encryption**: Some backends support encryption
5. **Sensitive variables**: Mark variables as sensitive (they won't show in logs)
6. **Never commit state**: Add to .gitignore
7. **Use secrets management**: Store secrets in Azure Key Vault, reference in Terraform

**Example sensitive variable:**
```hcl
variable "db_password" {
  type        = string
  sensitive   = true
  description = "Database password"
}
```

## Terraform Modules

### Q8: What are Terraform modules and when should you use them?

**Answer:**
Terraform modules are reusable packages of Terraform configurations. Think of them as functions in programming.

**When to use:**
- **Reusability**: Same infrastructure pattern used multiple times (e.g., create 5 VMs with same config)
- **Organization**: Break large configurations into smaller, manageable pieces
- **Abstraction**: Hide complexity - users just call module with parameters
- **Versioning**: Tag modules and reuse specific versions
- **Sharing**: Share modules across teams/projects

**Types:**
- **Root module**: Your main configuration
- **Child modules**: Modules you call from root module
- **Published modules**: Modules from Terraform Registry

**Example**: Instead of writing VM configuration 10 times, create a module once and call it 10 times with different parameters.

### Q9: Explain the difference between module source types (local, Git, registry).

**Answer:**
1. **Local modules**:
   ```hcl
   module "vnet" {
     source = "./modules/vnet"
   }
   ```
   - Module in same repository
   - Good for project-specific modules
   - Simple, but not reusable across projects

2. **Git modules**:
   ```hcl
   module "vnet" {
     source = "git::https://github.com/org/terraform-modules.git//vnet"
   }
   ```
   - Module in Git repository
   - Can version with tags/branches
   - Good for sharing across projects
   - Example: `source = "git::https://...?ref=v1.2.0"`

3. **Terraform Registry**:
   ```hcl
   module "vnet" {
     source = "Azure/vnet/azurerm"
     version = "2.0.0"
   }
   ```
   - Published modules from Terraform Registry
   - Versioned and maintained
   - Easy to use, well-documented
   - Public or private registry

**Best practice**: Start local, move to Git when sharing, use registry for common patterns.

### Q10: How do you structure a Terraform project with modules?

**Answer:**
**Recommended structure:**
```
project/
├── main.tf                 # Main configuration, calls modules
├── variables.tf            # Input variables
├── outputs.tf             # Output values
├── terraform.tfvars       # Variable values
├── modules/
│   ├── vnet/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── vm/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── storage/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── environments/
    ├── dev/
    │   └── terraform.tfvars
    ├── staging/
    │   └── terraform.tfvars
    └── prod/
        └── terraform.tfvars
```

**Principles:**
- **Root level**: Orchestrates modules, defines environment
- **Modules**: Self-contained, reusable components
- **Environments**: Separate variable files per environment
- **DRY**: Don't Repeat Yourself - use modules for repeated patterns

## Azure Provider

### Q11: How do you authenticate Terraform to Azure?

**Answer:**
Multiple ways to authenticate:

1. **Azure CLI** (easiest for local development):
   ```bash
   az login
   ```
   Terraform automatically uses Azure CLI credentials

2. **Service Principal** (for CI/CD):
   ```hcl
   provider "azurerm" {
     subscription_id = var.subscription_id
     client_id       = var.client_id
     client_secret   = var.client_secret
     tenant_id       = var.tenant_id
   }
   ```
   Store credentials in environment variables or Key Vault

3. **Managed Identity** (when running on Azure):
   ```hcl
   provider "azurerm" {
     use_msi = true
   }
   ```
   No credentials needed - uses VM's managed identity

4. **Environment variables**:
   ```bash
   export ARM_CLIENT_ID="..."
   export ARM_CLIENT_SECRET="..."
   export ARM_SUBSCRIPTION_ID="..."
   export ARM_TENANT_ID="..."
   ```

**Best practice**: Use Managed Identity in Azure, Service Principal in CI/CD, Azure CLI for local dev.

### Q12: Explain Terraform data sources and when to use them.

**Answer:**
Data sources let you fetch information about existing resources (that weren't created by Terraform).

**When to use:**
- Reference existing resources
- Get information about current Azure environment
- Avoid hardcoding values

**Example:**
```hcl
# Get existing resource group
data "azurerm_resource_group" "existing" {
  name = "my-existing-rg"
}

# Use it in resource
resource "azurerm_storage_account" "example" {
  name                = "mystorageaccount"
  resource_group_name = data.azurerm_resource_group.existing.name
  location            = data.azurerm_resource_group.existing.location
}
```

**Common data sources:**
- `azurerm_resource_group` - Get existing resource group
- `azurerm_virtual_network` - Get existing VNet
- `azurerm_client_config` - Get current Azure client info
- `azurerm_key_vault_secret` - Get secret from Key Vault

**Difference from resources**: Data sources are read-only - they fetch info but don't create/modify anything.

## Terraform Best Practices

### Q13: What are Terraform workspaces and when should you use them?

**Answer:**
Terraform workspaces let you manage multiple environments (dev, staging, prod) with the same configuration but different state files.

**How it works:**
```bash
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod
terraform workspace select dev
terraform apply
```

Each workspace has its own state file, so you can have:
- `terraform.tfstate.d/dev/terraform.tfstate`
- `terraform.tfstate.d/staging/terraform.tfstate`
- `terraform.tfstate.d/prod/terraform.tfstate`

**When to use:**
- Small projects with few environments
- Quick way to separate environments

**When NOT to use:**
- Large teams (workspaces share same backend)
- Need different backends per environment
- Better alternative: Separate directories per environment with different backends

**Best practice**: For production, use separate directories/backends instead of workspaces for better isolation.

### Q14: How do you handle Terraform variable validation and defaults?

**Answer:**
**Variable validation:**
```hcl
variable "environment" {
  type = string
  description = "Environment name"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

**Default values:**
```hcl
variable "vm_size" {
  type        = string
  default     = "Standard_B2s"
  description = "VM size"
}
```

**Variable types:**
```hcl
variable "tags" {
  type = map(string)
  default = {
    Environment = "dev"
    Project     = "myproject"
  }
}

variable "allowed_ips" {
  type    = list(string)
  default = ["10.0.0.0/8"]
}
```

**Best practices:**
- Always provide descriptions
- Use validation for critical variables
- Set sensible defaults
- Use type constraints (prevents errors)

### Q15: Explain Terraform dependency management and implicit vs explicit dependencies.

**Answer:**
Terraform automatically detects dependencies, but sometimes you need to create explicit ones.

**Implicit dependencies** (automatic):
```hcl
resource "azurerm_resource_group" "example" {
  name = "my-rg"
}

resource "azurerm_storage_account" "example" {
  name                = "mystorage"
  resource_group_name = azurerm_resource_group.example.name  # Implicit dependency
}
```
Terraform knows storage account depends on resource group because it references it.

**Explicit dependencies** (using depends_on):
```hcl
resource "azurerm_virtual_machine" "example" {
  # ... config ...
  
  depends_on = [
    azurerm_network_interface.example,
    azurerm_public_ip.example
  ]
}
```
Use when Terraform can't detect dependency automatically (e.g., custom scripts, external processes).

**Best practice**: Rely on implicit dependencies when possible. Use `depends_on` only when necessary.

## Troubleshooting Scenarios

### Q16: Terraform plan shows it will destroy and recreate a resource, but you only changed a small property. How do you fix this?

**Answer:**
This happens when a property change requires destroying and recreating the resource (provider limitation).

**Solutions:**
1. **Check if property supports update**: Some properties can't be updated in-place
2. **Use lifecycle block**:
   ```hcl
   resource "azurerm_virtual_machine" "example" {
     # ... config ...
     
     lifecycle {
       create_before_destroy = true  # Create new before destroying old
       prevent_destroy       = true  # Prevent accidental destruction
       ignore_changes = [           # Ignore changes to these properties
         tags,
       ]
     }
   }
   ```
3. **Use terraform taint/untaint**: Mark resource for recreation
   ```bash
   terraform taint azurerm_virtual_machine.example
   terraform apply
   ```
4. **Accept the change**: If recreation is necessary, proceed (during maintenance window)

**Prevention**: Review plan output carefully before applying. Test changes in dev first.

### Q17: You get "Error: resource already exists" when running terraform apply. How do you resolve this?

**Answer:**
This means a resource exists in Azure but not in Terraform state.

**Solutions:**
1. **Import the resource** (recommended):
   ```bash
   terraform import azurerm_storage_account.example /subscriptions/.../resourceGroups/.../storageAccounts/mystorage
   ```
   This adds existing resource to state.

2. **Remove from code**: If resource shouldn't be managed by Terraform, remove it from code and import to a null resource or data source.

3. **Delete and recreate**: If resource can be recreated, delete it manually in Azure, then run terraform apply.

**How to find resource ID**: Use Azure Portal or Azure CLI:
```bash
az storage account show --name mystorage --resource-group myrg --query id
```

**Prevention**: Always use Terraform to create resources, or import existing ones before managing them.

### Q18: Your Terraform state is out of sync with Azure (drift). How do you detect and fix it?

**Answer:**
**Detect drift:**
```bash
terraform plan
```
If plan shows changes but you didn't modify code, that's drift (someone changed resources manually in Azure).

**Fix drift:**
1. **Refresh state** (if changes are acceptable):
   ```bash
   terraform refresh
   terraform plan  # Should show no changes now
   ```
   This updates state to match reality.

2. **Revert changes**: If manual changes were wrong, revert them in Azure Portal/CLI, then refresh state.

3. **Update code**: If manual changes should be permanent, update Terraform code to match, then apply.

**Prevent drift:**
- Use Terraform for all changes (no manual changes)
- Use RBAC to restrict who can modify resources
- Use Azure Policy to enforce configurations
- Regular `terraform plan` to detect drift early

**Best practice**: Run `terraform plan` regularly (daily/weekly) to detect drift.

### Q19: How would you refactor a large Terraform configuration into modules?

**Answer:**
**Step-by-step approach:**

1. **Identify patterns**: Find repeated resource configurations
   - Example: 5 VMs with similar config → create VM module

2. **Create module structure**:
   ```
   modules/
   └── vm/
       ├── main.tf
       ├── variables.tf
       └── outputs.tf
   ```

3. **Move resources to module**: Copy resource definitions to module's main.tf

4. **Define inputs**: Move hardcoded values to variables.tf

5. **Define outputs**: Export values needed by other resources

6. **Update root module**: Replace resource blocks with module calls:
   ```hcl
   # Before
   resource "azurerm_virtual_machine" "vm1" { ... }
   resource "azurerm_virtual_machine" "vm2" { ... }
   
   # After
   module "vm1" {
     source = "./modules/vm"
     vm_name = "vm1"
     # ... other variables
   }
   
   module "vm2" {
     source = "./modules/vm"
     vm_name = "vm2"
     # ... other variables
   }
   ```

7. **Test incrementally**: 
   - Move one module at a time
   - Run terraform plan after each change
   - Verify no unexpected changes

8. **Update state**: Use `terraform state mv` to move resources in state:
   ```bash
   terraform state mv azurerm_virtual_machine.vm1 module.vm1.azurerm_virtual_machine.vm
   ```

**Benefits**: Reusability, maintainability, easier testing, better organization.

### Q20: Explain how you would use Terraform in a CI/CD pipeline for infrastructure deployment.

**Answer:**
**Pipeline stages:**

1. **Validate**:
   ```bash
   terraform init -backend=false
   terraform validate
   terraform fmt -check
   ```
   Check syntax and formatting

2. **Plan**:
   ```bash
   terraform init
   terraform plan -out=tfplan
   ```
   Generate plan file

3. **Review** (optional):
   - Store plan as artifact
   - Manual approval gate
   - Review changes before apply

4. **Apply** (after approval):
   ```bash
   terraform apply tfplan
   ```
   Apply the saved plan

5. **Verify**:
   - Run tests against deployed infrastructure
   - Check health endpoints
   - Validate configuration

**Best practices:**
- **Separate pipelines per environment**: Dev, staging, prod
- **Use remote state**: Azure Storage backend
- **Service Principal authentication**: Store credentials in pipeline variables (secret)
- **State locking**: Automatic with Azure Storage backend
- **Plan in PR**: Run terraform plan in pull requests to show changes
- **Tag resources**: Tag with pipeline run ID, commit hash
- **Rollback strategy**: Keep previous state, can import if needed

**Example Azure DevOps pipeline:**
```yaml
- task: TerraformInstaller@0
- task: TerraformTaskV2@2
  inputs:
    provider: 'azurerm'
    command: 'init'
- task: TerraformTaskV2@2
  inputs:
    command: 'plan'
- task: TerraformTaskV2@2
  inputs:
    command: 'apply'
    environmentServiceName: 'Azure-SP'
```

