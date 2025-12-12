# Azure Solutions - Interview Q&A (5+ Years Experience)

## Azure Core Services

### Q1: What are the main Azure compute services and when would you use each?

**Answer:**
- **Azure Virtual Machines (VMs)**: Use when you need full control over the operating system, need to run legacy applications, or have specific software requirements. Good for lift-and-shift migrations.
- **Azure App Service**: Use for web applications, APIs, and mobile backends. Supports multiple languages and auto-scaling. Best for modern web apps.
- **Azure Container Instances (ACI)**: Use for simple container workloads that don't need orchestration. Good for quick deployments.
- **Azure Functions**: Use for serverless event-driven applications, microservices, and scheduled tasks. Pay only for execution time.
- **Azure Kubernetes Service (AKS)**: Use when you need container orchestration at scale with high availability and auto-scaling.

### Q2: Explain Azure Resource Groups and their importance.

**Answer:**
Azure Resource Groups are containers that hold related resources for an Azure solution. They help you:
- **Organize resources**: Group related resources together (like a web app, database, and storage for one application)
- **Manage lifecycle**: Delete all resources in a group at once
- **Apply policies**: Set permissions and policies at the resource group level
- **Track costs**: View costs for all resources in a group together
- **Deploy together**: Deploy multiple resources as a single unit

Best practice: Use one resource group per application or environment (dev, staging, prod).

### Q3: What is the difference between Azure Storage Accounts and Azure Blob Storage?

**Answer:**
- **Azure Storage Account**: This is the top-level container that provides a unique namespace for your Azure Storage data. It contains all your storage services.
- **Azure Blob Storage**: This is one of the four data services within a Storage Account (along with Files, Queues, and Tables). Blob Storage is specifically for storing unstructured data like images, videos, documents, backups, and log files.

Think of Storage Account as the building, and Blob Storage as one room in that building.

### Q4: How does Azure Availability Sets work and why are they important?

**Answer:**
Azure Availability Sets ensure your VMs are distributed across multiple physical servers, storage, and network switches. This provides:
- **Fault domains**: VMs are placed on different physical servers. If one server fails, others continue running.
- **Update domains**: VMs are updated in groups, so some VMs always remain available during updates.

**Why important**: Without Availability Sets, all your VMs could be on the same physical server. If that server fails or needs maintenance, all your VMs go down. Availability Sets guarantee at least 99.95% uptime SLA.

### Q5: Explain Azure Managed Disks and their advantages.

**Answer:**
Azure Managed Disks are block-level storage volumes managed by Azure. Advantages:
- **Simplified management**: Azure handles storage accounts for you
- **Better performance**: Up to 20,000 IOPS per disk
- **High availability**: Three replicas by default
- **Easy backup**: Integrated with Azure Backup
- **Snapshot support**: Create point-in-time copies
- **Disk types**: Premium SSD, Standard SSD, Standard HDD, Ultra SSD

You just create a disk and attach it - no need to manage storage accounts manually.

## Azure Virtual Network

### Q6: What is Azure Virtual Network (VNet) and what are its key components?

**Answer:**
Azure Virtual Network is your private network in the cloud. Key components:
- **Subnets**: Divide your VNet into smaller networks. Each subnet can have different security rules.
- **Network Security Groups (NSG)**: Firewall rules that control traffic to/from subnets and VMs.
- **Route Tables**: Control how traffic flows between subnets and to the internet.
- **DNS**: Name resolution for resources in your VNet.
- **Service Endpoints**: Secure connection to Azure services without public internet.

Think of VNet as your company's private network, but in the cloud.

### Q7: Explain the difference between Azure VNet Peering and VPN Gateway.

**Answer:**
- **VNet Peering**: Connects two Azure VNets in the same or different regions. Traffic stays on Azure's private network (faster, no internet). Used for connecting Azure resources.
- **VPN Gateway**: Creates an encrypted tunnel over the internet to connect:
  - Azure VNet to on-premises network (Site-to-Site VPN)
  - Individual devices to Azure VNet (Point-to-Site VPN)
  - Azure VNet to another Azure VNet (VNet-to-VNet)

**Use VNet Peering** when both networks are in Azure.
**Use VPN Gateway** when you need to connect Azure to on-premises or use internet-based connection.

### Q8: What are Azure Private Endpoints and why use them?

**Answer:**
Azure Private Endpoints give you a private IP address in your VNet for Azure services (like Storage, SQL Database, Key Vault). Benefits:
- **Security**: Traffic never goes over the public internet - stays on Azure's private network
- **Compliance**: Meet requirements that prohibit public internet access
- **Network isolation**: Services appear as if they're in your VNet
- **DNS integration**: Automatic DNS resolution in your VNet

Example: Instead of accessing storage account via public URL, you access it via a private IP like 10.0.1.5, and traffic never leaves Azure's network.

## Azure Load Balancer

### Q9: Explain the difference between Azure Load Balancer and Application Gateway.

**Answer:**
- **Azure Load Balancer (Layer 4)**: Works at transport layer (TCP/UDP). Distributes traffic based on source IP and port. Fast, simple, but doesn't understand HTTP. Use for non-HTTP traffic or simple HTTP load balancing.

- **Application Gateway (Layer 7)**: Works at application layer (HTTP/HTTPS). Understands HTTP requests. Can:
  - Route based on URL path (e.g., /api goes to backend, /images goes to storage)
  - Do SSL termination (decrypt at gateway)
  - Web Application Firewall (WAF) protection
  - Cookie-based session affinity
  - URL rewriting

**Use Load Balancer** for simple TCP/UDP load balancing.
**Use Application Gateway** when you need HTTP-aware routing, SSL termination, or WAF.

### Q10: What is Azure Front Door and when would you use it instead of Application Gateway?

**Answer:**
Azure Front Door is a global CDN and application accelerator. Use it when:
- **Global presence**: Your users are worldwide and you need low latency everywhere
- **Multiple regions**: You have applications in multiple Azure regions and want automatic failover
- **DDoS protection**: Built-in protection against attacks
- **SSL termination**: Handles SSL certificates globally
- **WAF**: Web Application Firewall at the edge

**Key difference**: Application Gateway is regional (one region), Front Door is global (edge locations worldwide).

**Use Application Gateway** for regional applications.
**Use Front Door** for global applications or when you need edge caching and global load balancing.

## Azure Security

### Q11: What is Azure Security Center and what does it do?

**Answer:**
Azure Security Center is a unified security management system that:
- **Monitors security**: Continuously monitors your Azure resources for security issues
- **Provides recommendations**: Suggests security improvements (like enabling encryption, fixing vulnerabilities)
- **Threat detection**: Identifies suspicious activities and potential attacks
- **Compliance**: Checks if your resources meet compliance standards (PCI-DSS, HIPAA, etc.)
- **Security score**: Gives you a score showing how secure your environment is

Think of it as a security guard that watches your Azure resources 24/7 and alerts you to problems.

### Q12: Explain Azure Firewall and its key features.

**Answer:**
Azure Firewall is a managed, cloud-based network security service that protects your Azure VNet resources. Features:
- **Network rules**: Allow/deny traffic based on source/destination IP and port
- **Application rules**: Control outbound access to FQDNs (like *.google.com)
- **Threat intelligence**: Automatically blocks traffic from known malicious IPs
- **FQDN tags**: Pre-configured groups of FQDNs (like Windows Update)
- **Service tags**: Pre-defined groups of Azure service IP ranges

It's a stateful firewall, meaning it remembers connections and only allows return traffic for established connections.

### Q13: What is the difference between Azure Key Vault and Azure Secrets Manager?

**Answer:**
Actually, Azure doesn't have a separate "Secrets Manager" - secrets are stored in **Azure Key Vault**, which handles:
- **Secrets**: Passwords, API keys, connection strings
- **Keys**: Encryption keys for encrypting data
- **Certificates**: SSL/TLS certificates

Key Vault provides:
- **Secure storage**: Encrypted at rest and in transit
- **Access control**: Who can read/write secrets
- **Audit logging**: Track who accessed what and when
- **Automatic rotation**: Can automatically rotate secrets
- **Versioning**: Keep multiple versions of secrets

Best practice: Never hardcode secrets in code. Store them in Key Vault and retrieve at runtime.

## Azure DevOps Integration

### Q14: How do you integrate Azure resources with Azure DevOps pipelines?

**Answer:**
1. **Service Connections**: Create service connections in Azure DevOps that authenticate to your Azure subscription
2. **Azure Resource Manager tasks**: Use built-in tasks like "Azure Web App Deploy", "Azure SQL Database Deployment"
3. **Azure CLI/PowerShell tasks**: Run Azure CLI or PowerShell commands in pipelines
4. **Managed Identity**: Use managed identities so pipelines don't need stored credentials
5. **Azure Key Vault task**: Retrieve secrets from Key Vault during pipeline execution

Example pipeline:
- Build your application
- Run tests
- Deploy to Azure App Service using "Azure Web App Deploy" task
- Retrieve connection strings from Key Vault
- Update application settings

### Q15: What is Azure Managed Identity and why is it important?

**Answer:**
Azure Managed Identity provides an automatically managed identity in Azure Active Directory. Types:
- **System-assigned**: Tied to a specific Azure resource (VM, App Service). Deleted when resource is deleted.
- **User-assigned**: Standalone identity that can be assigned to multiple resources.

**Why important**:
- **No passwords**: No need to store credentials in code or configuration
- **Automatic rotation**: Azure manages the credentials
- **Secure**: Credentials never exposed to developers
- **Easy**: Just enable it and grant permissions

Example: App Service with managed identity can access Azure SQL Database without storing connection strings - just grant the identity permission to the database.

## Scenario-Based Questions

### Q16: You need to deploy a web application that must be highly available across multiple regions. How would you design this in Azure?

**Answer:**
1. **Multiple App Service Plans**: Deploy App Service in at least 2 regions (e.g., East US, West Europe)
2. **Azure Front Door**: Use Front Door for global load balancing and automatic failover
3. **Traffic Manager (alternative)**: For DNS-based load balancing if not using Front Door
4. **Database**: Use Azure SQL Database with geo-replication or Cosmos DB with multi-region writes
5. **Storage**: Use Azure Storage with geo-redundant replication
6. **Health Probes**: Configure Front Door to check application health and route traffic only to healthy regions
7. **CDN**: Use Azure CDN for static content caching globally
8. **Monitoring**: Set up Application Insights in all regions to monitor health

This ensures if one region fails, traffic automatically routes to healthy regions.

### Q17: A client reports their Azure VM is slow. How would you troubleshoot this?

**Answer:**
1. **Check Azure Monitor**: Look at CPU, memory, disk IOPS, and network metrics
2. **Check VM size**: Verify if VM size is appropriate for workload (maybe need more CPU/RAM)
3. **Check disk performance**: 
   - Is it using Standard HDD? Upgrade to Premium SSD
   - Check disk IOPS limits
   - Consider using Ultra SSD for high IOPS
4. **Check network**: Look at network latency and bandwidth usage
5. **Check application**: Use Application Insights or logs to see if application has issues
6. **Check resource limits**: Verify you're not hitting subscription or resource group limits
7. **Check for throttling**: Look for throttling errors in logs
8. **Compare baseline**: Compare current metrics to historical baseline

Common fixes: Upgrade VM size, change disk type, add more disks in RAID, optimize application code.

### Q18: How would you secure an Azure environment for a financial application handling sensitive data?

**Answer:**
1. **Network Security**:
   - Use VNets with subnets (web tier, app tier, data tier)
   - NSGs to restrict traffic between tiers
   - Azure Firewall for outbound traffic control
   - Private Endpoints for Azure services (no public internet access)
   - VPN Gateway for secure on-premises connection

2. **Identity & Access**:
   - Azure AD with MFA for all users
   - Role-Based Access Control (RBAC) - least privilege
   - Managed Identities (no passwords in code)
   - Key Vault for all secrets

3. **Data Protection**:
   - Encryption at rest (Azure Storage, SQL Database)
   - Encryption in transit (TLS 1.2+)
   - Azure SQL Database with Transparent Data Encryption (TDE)
   - Azure Disk Encryption for VMs

4. **Monitoring & Compliance**:
   - Azure Security Center (Standard tier) for advanced threat detection
   - Azure Monitor and Log Analytics for audit logs
   - Compliance policies (PCI-DSS, SOC 2)
   - Regular security assessments

5. **Application Security**:
   - WAF on Application Gateway or Front Door
   - SAST/DAST scanning in CI/CD pipeline
   - Regular vulnerability scanning
   - DDoS Protection Standard

6. **Backup & Recovery**:
   - Azure Backup for VMs and databases
   - Geo-redundant storage
   - Documented disaster recovery plan

### Q19: Explain how you would migrate an on-premises application to Azure with minimal downtime.

**Answer:**
1. **Assessment Phase**:
   - Use Azure Migrate to assess on-premises servers
   - Identify dependencies between servers
   - Calculate costs and right-size VMs

2. **Replication Phase**:
   - Use Azure Site Recovery (ASR) to replicate VMs to Azure
   - Replicate continuously (changes sync every few minutes)
   - Test failover in isolated network (doesn't affect production)

3. **Network Setup**:
   - Create VNet in Azure matching on-premises network
   - Set up VPN Gateway for connectivity
   - Configure DNS to resolve both environments

4. **Data Migration**:
   - Use Azure Database Migration Service for databases
   - Use Azure Data Box for large file transfers
   - Sync data continuously until cutover

5. **Cutover (Minimal Downtime)**:
   - Schedule maintenance window
   - Stop application
   - Final data sync
   - Perform failover in ASR (brings up VMs in Azure)
   - Update DNS to point to Azure
   - Start application
   - Verify everything works

6. **Post-Migration**:
   - Monitor closely for issues
   - Keep on-premises running for rollback if needed
   - Optimize Azure resources
   - Decommission on-premises after validation period

Total downtime: Usually 1-4 hours depending on application complexity.

### Q20: Your Azure costs are unexpectedly high. How would you identify and reduce them?

**Answer:**
1. **Cost Analysis**:
   - Use Azure Cost Management to see spending by resource, resource group, service
   - Identify top spending areas
   - Check for unused resources (stopped VMs, unattached disks)

2. **Right-Sizing**:
   - Review VM sizes - are they too large for workload?
   - Use Azure Advisor recommendations
   - Consider reserved instances for predictable workloads (save up to 72%)

3. **Optimize Storage**:
   - Move cold data to Archive tier (cheaper)
   - Delete old backups and snapshots
   - Use appropriate storage tier (Hot/Cool/Archive)

4. **Optimize Compute**:
   - Use App Service instead of VMs where possible (cheaper)
   - Use Azure Functions for sporadic workloads
   - Shut down dev/test VMs when not in use (automation)

5. **Network Costs**:
   - Reduce data transfer (use CDN, keep data in same region)
   - Use VNet Peering instead of VPN Gateway where possible
   - Optimize bandwidth usage

6. **Monitoring & Automation**:
   - Set up budget alerts
   - Automate shutdown of non-production resources
   - Use tags to track costs by project/department

7. **Licensing**:
   - Use Azure Hybrid Benefit if you have Windows Server/SQL licenses
   - Consider spot instances for non-critical workloads

Common savings: 30-50% with proper optimization.

