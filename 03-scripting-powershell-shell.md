# Scripting (PowerShell & Shell) - Interview Q&A

## PowerShell for Azure Automation

### Q1: What is PowerShell and why is it important for Azure DevOps?

**Answer:**
PowerShell is a command-line shell and scripting language developed by Microsoft. It's important for Azure DevOps because:
- **Native Azure support**: Azure CLI and Azure PowerShell modules are built for PowerShell
- **Object-oriented**: Works with objects, not just text (unlike bash)
- **Cross-platform**: Works on Windows, Linux, and macOS
- **Automation**: Can automate complex Azure operations
- **Integration**: Works seamlessly with Azure services and APIs

**PowerShell vs Bash**: PowerShell uses objects (structured data), bash uses text (unstructured). PowerShell is better for Windows/Azure environments, bash is better for Linux.

### Q2: Explain the difference between Azure CLI and Azure PowerShell.

**Answer:**
- **Azure CLI**: Command-line interface using bash-like syntax. Cross-platform, good for quick commands, simpler syntax.
  ```bash
  az vm create --name myVM --resource-group myRG
  ```

- **Azure PowerShell**: PowerShell cmdlets (commands) for Azure. More powerful for scripting, object-oriented, better for complex automation.
  ```powershell
  New-AzVM -Name myVM -ResourceGroupName myRG
  ```

**When to use:**
- **Azure CLI**: Quick commands, simple scripts, cross-platform scripts
- **Azure PowerShell**: Complex automation, Windows-heavy environments, when you need object manipulation

**Both can do the same things** - choose based on your preference and team's expertise.

### Q3: How do you authenticate PowerShell scripts to Azure?

**Answer:**
**Method 1: Interactive login** (for local development):
```powershell
Connect-AzAccount
```
Opens browser for login. Credentials cached for session.

**Method 2: Service Principal** (for automation/CI/CD):
```powershell
$securePassword = ConvertTo-SecureString "password" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential("app-id", $securePassword)
Connect-AzAccount -ServicePrincipal -Credential $credential -TenantId "tenant-id"
```

**Method 3: Managed Identity** (when running on Azure VM):
```powershell
Connect-AzAccount -Identity
```
No credentials needed - uses VM's managed identity.

**Method 4: Certificate authentication**:
```powershell
Connect-AzAccount -ServicePrincipal -CertificateThumbprint "thumbprint" -ApplicationId "app-id" -TenantId "tenant-id"
```

**Best practice**: Use Managed Identity in Azure, Service Principal in CI/CD, interactive for local dev.

### Q4: Write a PowerShell script to create a resource group and storage account in Azure.

**Answer:**
```powershell
# Set variables
$resourceGroupName = "myResourceGroup"
$location = "East US"
$storageAccountName = "mystorageaccount$(Get-Random)"

# Connect to Azure (if not already connected)
# Connect-AzAccount

# Create resource group
New-AzResourceGroup -Name $resourceGroupName -Location $location

# Create storage account
$storageAccount = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind StorageV2

# Output storage account name
Write-Host "Storage account created: $($storageAccount.StorageAccountName)"
```

**Key points:**
- Use variables for reusability
- Use backtick (`) for line continuation
- Use `Write-Host` for output
- Check if resource exists before creating (use `Get-AzResourceGroup`)

### Q5: How do you handle errors in PowerShell scripts?

**Answer:**
**Error handling methods:**

1. **Try-Catch-Finally**:
```powershell
try {
    New-AzResourceGroup -Name "myRG" -Location "East US"
    Write-Host "Resource group created successfully"
}
catch {
    Write-Error "Failed to create resource group: $($_.Exception.Message)"
    # Handle error - maybe retry, log, or exit
}
finally {
    # Cleanup code that always runs
    Write-Host "Script execution completed"
}
```

2. **ErrorAction parameter**:
```powershell
# Stop on error
New-AzVM -Name "myVM" -ErrorAction Stop

# Silently continue (suppress error)
Get-AzResourceGroup -Name "nonexistent" -ErrorAction SilentlyContinue

# Continue but record error
Get-AzVM -Name "myVM" -ErrorAction Continue
```

3. **$Error variable**:
```powershell
# Check last error
if ($Error.Count -gt 0) {
    Write-Host "Last error: $($Error[0].Exception.Message)"
}
```

4. **Set error preference**:
```powershell
$ErrorActionPreference = "Stop"  # Stop on any error
$ErrorActionPreference = "Continue"  # Continue on error (default)
```

**Best practice**: Use Try-Catch for critical operations, set `$ErrorActionPreference = "Stop"` at script start.

### Q6: Explain PowerShell variables, arrays, and hashtables.

**Answer:**
**Variables:**
```powershell
$name = "John"
$age = 30
$isActive = $true
```
Variables start with `$`. PowerShell is loosely typed (automatically determines type).

**Arrays:**
```powershell
$fruits = @("apple", "banana", "orange")
$numbers = 1, 2, 3, 4, 5

# Access elements
$fruits[0]  # apple
$fruits.Count  # 3
```

**Hashtables (key-value pairs):**
```powershell
$person = @{
    Name = "John"
    Age = 30
    City = "New York"
}

# Access values
$person.Name
$person["Age"]

# Add/Modify
$person["Email"] = "john@example.com"
```

**Common use**: Hashtables are great for tags, configuration, and structured data.

### Q7: How do you loop through Azure resources in PowerShell?

**Answer:**
**Foreach loop:**
```powershell
$resourceGroups = Get-AzResourceGroup

foreach ($rg in $resourceGroups) {
    Write-Host "Resource Group: $($rg.ResourceGroupName)"
    Write-Host "Location: $($rg.Location)"
    
    # Get resources in this group
    $resources = Get-AzResource -ResourceGroupName $rg.ResourceGroupName
    foreach ($resource in $resources) {
        Write-Host "  - $($resource.Name) ($($resource.ResourceType))"
    }
}
```

**Foreach-Object (pipeline):**
```powershell
Get-AzVM | ForEach-Object {
    Write-Host "VM: $($_.Name)"
    Write-Host "Status: $($_.PowerState)"
}
```

**While loop:**
```powershell
$counter = 0
while ($counter -lt 10) {
    Write-Host "Counter: $counter"
    $counter++
}
```

**For loop:**
```powershell
for ($i = 0; $i -lt 10; $i++) {
    Write-Host "Iteration: $i"
}
```

### Q8: Write a PowerShell script to backup all VMs in a resource group.

**Answer:**
```powershell
# Parameters
param(
    [string]$ResourceGroupName = "myResourceGroup",
    [string]$BackupVaultName = "myBackupVault",
    [string]$BackupPolicyName = "DailyBackupPolicy"
)

# Get all VMs in resource group
$vms = Get-AzVM -ResourceGroupName $ResourceGroupName

if ($vms.Count -eq 0) {
    Write-Host "No VMs found in resource group: $ResourceGroupName"
    exit
}

# Get backup vault
$backupVault = Get-AzRecoveryServicesVault -Name $BackupVaultName

# Set vault context
Set-AzRecoveryServicesVaultContext -Vault $backupVault

# Get backup policy
$backupPolicy = Get-AzRecoveryServicesBackupProtectionPolicy -Name $BackupPolicyName

# Enable backup for each VM
foreach ($vm in $vms) {
    try {
        Write-Host "Enabling backup for VM: $($vm.Name)"
        
        Enable-AzRecoveryServicesBackupProtection `
            -ResourceGroupName $ResourceGroupName `
            -Name $vm.Name `
            -Policy $backupPolicy
        
        Write-Host "Backup enabled successfully for: $($vm.Name)" -ForegroundColor Green
    }
    catch {
        Write-Error "Failed to enable backup for $($vm.Name): $($_.Exception.Message)"
    }
}

Write-Host "Backup configuration completed for all VMs"
```

## Bash/Shell Scripting

### Q9: What are the basic components of a bash script?

**Answer:**
**Basic structure:**
```bash
#!/bin/bash
# This is a comment

# Variables
NAME="John"
AGE=30

# Functions
greet() {
    echo "Hello, $1"
}

# Main script
greet "$NAME"
echo "Age: $AGE"
```

**Key components:**
- **Shebang** (`#!/bin/bash`): Tells system which interpreter to use
- **Variables**: `VARIABLE="value"` (no spaces around `=`)
- **Functions**: Reusable code blocks
- **Commands**: Execute system commands
- **Control flow**: if/else, loops, case statements

**Make executable:**
```bash
chmod +x script.sh
./script.sh
```

### Q10: Explain bash variable types and how to use them.

**Answer:**
**Regular variables:**
```bash
NAME="John"
AGE=30
IS_ACTIVE=true

# Use variable
echo $NAME
echo ${NAME}  # Preferred (clearer boundaries)
```

**Environment variables:**
```bash
# Set
export DATABASE_URL="postgresql://localhost:5432/mydb"

# Use
echo $DATABASE_URL

# In script, access system env vars
echo $HOME
echo $PATH
```

**Command substitution:**
```bash
# Store command output in variable
CURRENT_DATE=$(date)
# or
CURRENT_DATE=`date`

echo "Today is: $CURRENT_DATE"
```

**Arrays:**
```bash
FRUITS=("apple" "banana" "orange")

# Access
echo ${FRUITS[0]}  # apple
echo ${FRUITS[@]}  # all elements
echo ${#FRUITS[@]}  # length
```

**Special variables:**
```bash
$0  # Script name
$1, $2, $3  # Command line arguments
$#  # Number of arguments
$?  # Exit status of last command
$$  # Process ID
```

### Q11: How do you handle errors in bash scripts?

**Answer:**
**Exit on error:**
```bash
#!/bin/bash
set -e  # Exit immediately if a command exits with non-zero status
set -u  # Exit if undefined variable is used
set -o pipefail  # Exit if any command in pipeline fails

# Your script here
```

**Manual error checking:**
```bash
if ! command_that_might_fail; then
    echo "Command failed"
    exit 1
fi
```

**Trap errors:**
```bash
#!/bin/bash

# Function to handle errors
error_handler() {
    echo "Error occurred at line $1"
    exit 1
}

# Set trap
trap 'error_handler $LINENO' ERR

# Your script
```

**Check exit status:**
```bash
command_that_might_fail
if [ $? -ne 0 ]; then
    echo "Command failed with exit code $?"
    exit 1
fi
```

**Best practice**: Use `set -euo pipefail` at the start of scripts for strict error handling.

### Q12: Write a bash script to check if Azure resources exist and create them if they don't.

**Answer:**
```bash
#!/bin/bash
set -euo pipefail

# Variables
RESOURCE_GROUP="myResourceGroup"
LOCATION="eastus"
STORAGE_ACCOUNT="mystorageaccount$(date +%s)"

# Function to check if resource group exists
check_resource_group() {
    if az group show --name "$RESOURCE_GROUP" &>/dev/null; then
        echo "Resource group $RESOURCE_GROUP already exists"
        return 0
    else
        echo "Resource group $RESOURCE_GROUP does not exist"
        return 1
    fi
}

# Function to create resource group
create_resource_group() {
    echo "Creating resource group: $RESOURCE_GROUP"
    az group create \
        --name "$RESOURCE_GROUP" \
        --location "$LOCATION"
    echo "Resource group created successfully"
}

# Function to check if storage account exists
check_storage_account() {
    if az storage account show \
        --name "$STORAGE_ACCOUNT" \
        --resource-group "$RESOURCE_GROUP" &>/dev/null; then
        echo "Storage account $STORAGE_ACCOUNT already exists"
        return 0
    else
        echo "Storage account $STORAGE_ACCOUNT does not exist"
        return 1
    fi
}

# Function to create storage account
create_storage_account() {
    echo "Creating storage account: $STORAGE_ACCOUNT"
    az storage account create \
        --name "$STORAGE_ACCOUNT" \
        --resource-group "$RESOURCE_GROUP" \
        --location "$LOCATION" \
        --sku Standard_LRS
    echo "Storage account created successfully"
}

# Main execution
if ! check_resource_group; then
    create_resource_group
fi

if ! check_storage_account; then
    create_storage_account
fi

echo "All resources are ready"
```

### Q13: Explain bash conditional statements and loops.

**Answer:**
**If-else:**
```bash
if [ "$AGE" -gt 18 ]; then
    echo "Adult"
elif [ "$AGE" -gt 13 ]; then
    echo "Teenager"
else
    echo "Child"
fi

# File checks
if [ -f "file.txt" ]; then
    echo "File exists"
fi

if [ -d "directory" ]; then
    echo "Directory exists"
fi
```

**Case statement:**
```bash
case "$ENVIRONMENT" in
    dev)
        echo "Development environment"
        ;;
    staging)
        echo "Staging environment"
        ;;
    prod)
        echo "Production environment"
        ;;
    *)
        echo "Unknown environment"
        ;;
esac
```

**For loop:**
```bash
# Loop through list
for fruit in apple banana orange; do
    echo "Fruit: $fruit"
done

# Loop through array
FRUITS=("apple" "banana" "orange")
for fruit in "${FRUITS[@]}"; do
    echo "Fruit: $fruit"
done

# C-style loop
for ((i=0; i<10; i++)); do
    echo "Number: $i"
done
```

**While loop:**
```bash
counter=0
while [ $counter -lt 10 ]; do
    echo "Counter: $counter"
    counter=$((counter + 1))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt
```

**Until loop:**
```bash
counter=0
until [ $counter -ge 10 ]; do
    echo "Counter: $counter"
    counter=$((counter + 1))
done
```

### Q14: How do you pass parameters to bash scripts?

**Answer:**
**Positional parameters:**
```bash
#!/bin/bash
# script.sh arg1 arg2 arg3

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"
```

**Named parameters (using getopts):**
```bash
#!/bin/bash

while getopts "u:p:h" opt; do
    case $opt in
        u)
            USERNAME="$OPTARG"
            ;;
        p)
            PASSWORD="$OPTARG"
            ;;
        h)
            echo "Usage: script.sh -u username -p password"
            exit 0
            ;;
        *)
            echo "Invalid option"
            exit 1
            ;;
    esac
done

echo "Username: $USERNAME"
echo "Password: $PASSWORD"
```

**Usage:**
```bash
./script.sh -u john -p secret123
```

**Default values:**
```bash
RESOURCE_GROUP="${1:-myDefaultRG}"
LOCATION="${2:-eastus}"
```

### Q15: Write a bash script that monitors Azure VM status and sends alerts.

**Answer:**
```bash
#!/bin/bash
set -euo pipefail

# Configuration
RESOURCE_GROUP="myResourceGroup"
ALERT_EMAIL="admin@example.com"
CHECK_INTERVAL=300  # 5 minutes

# Function to get VM status
get_vm_status() {
    local vm_name=$1
    az vm show \
        --name "$vm_name" \
        --resource-group "$RESOURCE_GROUP" \
        --show-details \
        --query "powerState" \
        --output tsv
}

# Function to send alert (simplified - would use actual email service)
send_alert() {
    local vm_name=$1
    local status=$2
    echo "ALERT: VM $vm_name is in state: $status" | \
        mail -s "VM Status Alert" "$ALERT_EMAIL" 2>/dev/null || \
        echo "ALERT: VM $vm_name is in state: $status"
}

# Function to check all VMs
check_all_vms() {
    local vms=$(az vm list \
        --resource-group "$RESOURCE_GROUP" \
        --query "[].name" \
        --output tsv)
    
    for vm in $vms; do
        status=$(get_vm_status "$vm")
        echo "VM: $vm, Status: $status"
        
        # Alert if VM is deallocated
        if [ "$status" = "VM deallocated" ]; then
            send_alert "$vm" "$status"
        fi
    done
}

# Main monitoring loop
echo "Starting VM monitoring..."
while true; do
    check_all_vms
    echo "Waiting $CHECK_INTERVAL seconds before next check..."
    sleep $CHECK_INTERVAL
done
```

## Automation Scenarios

### Q16: How would you automate Azure resource cleanup using PowerShell?

**Answer:**
```powershell
# Script to delete resources older than 30 days
param(
    [string]$ResourceGroupName = "test-resources",
    [int]$DaysOld = 30
)

# Get all resources in resource group
$resources = Get-AzResource -ResourceGroupName $ResourceGroupName

$cutoffDate = (Get-Date).AddDays(-$DaysOld)

foreach ($resource in $resources) {
    # Check if resource has creation time tag
    $createdDate = $resource.Tags.CreatedDate
    
    if ($createdDate) {
        $createdDateTime = [DateTime]::Parse($createdDate)
        
        if ($createdDateTime -lt $cutoffDate) {
            Write-Host "Deleting resource: $($resource.Name) (Created: $createdDate)"
            
            try {
                Remove-AzResource `
                    -ResourceId $resource.ResourceId `
                    -Force `
                    -ErrorAction Stop
                
                Write-Host "Deleted: $($resource.Name)" -ForegroundColor Green
            }
            catch {
                Write-Error "Failed to delete $($resource.Name): $($_.Exception.Message)"
            }
        }
    }
    else {
        Write-Warning "Resource $($resource.Name) has no CreatedDate tag - skipping"
    }
}
```

**Best practice**: Tag resources with creation date when creating them, so cleanup scripts can identify old resources.

### Q17: Write a shell script to automate Azure backup verification.

**Answer:**
```bash
#!/bin/bash
set -euo pipefail

# Configuration
VAULT_NAME="myBackupVault"
RESOURCE_GROUP="myResourceGroup"
VM_NAME="myVM"
RETENTION_DAYS=30

# Function to verify backup exists
verify_backup() {
    local vm_name=$1
    local backup_date=$(date -d "$RETENTION_DAYS days ago" +%Y-%m-%d)
    
    # Get backup items
    local backup_items=$(az backup protection list \
        --vault-name "$VAULT_NAME" \
        --resource-group "$RESOURCE_GROUP" \
        --query "[?contains(name, '$vm_name')].name" \
        --output tsv)
    
    if [ -z "$backup_items" ]; then
        echo "ERROR: No backup found for VM: $vm_name"
        return 1
    fi
    
    # Check if backup is recent enough
    local latest_backup=$(az backup recoverypoint list \
        --vault-name "$VAULT_NAME" \
        --resource-group "$RESOURCE_GROUP" \
        --container-name "$vm_name" \
        --item-name "$vm_name" \
        --query "[0].properties.recoveryPointTime" \
        --output tsv)
    
    if [ -z "$latest_backup" ]; then
        echo "ERROR: No recovery points found for VM: $vm_name"
        return 1
    fi
    
    echo "Latest backup for $vm_name: $latest_backup"
    echo "Backup verification passed"
    return 0
}

# Main execution
if verify_backup "$VM_NAME"; then
    echo "Backup verification successful"
    exit 0
else
    echo "Backup verification failed"
    exit 1
fi
```

### Q18: Explain how to create reusable PowerShell functions for Azure operations.

**Answer:**
**Create a module file (AzureHelpers.psm1):**
```powershell
# AzureHelpers.psm1

function New-AzureResourceGroupIfNotExists {
    param(
        [Parameter(Mandatory=$true)]
        [string]$ResourceGroupName,
        
        [Parameter(Mandatory=$true)]
        [string]$Location
    )
    
    $rg = Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue
    
    if (-not $rg) {
        Write-Host "Creating resource group: $ResourceGroupName"
        New-AzResourceGroup -Name $ResourceGroupName -Location $Location
        return $true
    }
    else {
        Write-Host "Resource group already exists: $ResourceGroupName"
        return $false
    }
}

function Get-AzureVMStatus {
    param(
        [Parameter(Mandatory=$true)]
        [string]$ResourceGroupName,
        
        [Parameter(Mandatory=$true)]
        [string]$VMName
    )
    
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName -Status
    return $vm.Statuses | Where-Object { $_.Code -like "PowerState/*" } | Select-Object -ExpandProperty Code
}

function Set-AzureVMTags {
    param(
        [Parameter(Mandatory=$true)]
        [string]$ResourceGroupName,
        
        [Parameter(Mandatory=$true)]
        [string]$VMName,
        
        [Parameter(Mandatory=$true)]
        [hashtable]$Tags
    )
    
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName
    
    foreach ($key in $Tags.Keys) {
        $vm.Tags[$key] = $Tags[$key]
    }
    
    Update-AzVM -ResourceGroupName $ResourceGroupName -VM $vm
}

# Export functions
Export-ModuleMember -Function New-AzureResourceGroupIfNotExists, Get-AzureVMStatus, Set-AzureVMTags
```

**Use the module:**
```powershell
# Import module
Import-Module ./AzureHelpers.psm1

# Use functions
New-AzureResourceGroupIfNotExists -ResourceGroupName "myRG" -Location "East US"
$status = Get-AzureVMStatus -ResourceGroupName "myRG" -VMName "myVM"
Set-AzureVMTags -ResourceGroupName "myRG" -VMName "myVM" -Tags @{Environment="Prod"; Owner="DevOps"}
```

**Benefits**: Reusability, consistency, easier maintenance, team sharing.

