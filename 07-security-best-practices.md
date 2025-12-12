# Security Best Practices - Interview Q&A

## Azure Security Center

### Q1: What is Azure Security Center and what does it do?

**Answer:**
Azure Security Center is a unified security management system that helps you:
- **Monitor security**: Continuously watches your Azure resources for security issues
- **Provide recommendations**: Suggests security improvements (enable encryption, fix vulnerabilities)
- **Detect threats**: Identifies suspicious activities and potential attacks
- **Check compliance**: Verifies if resources meet compliance standards (PCI-DSS, HIPAA, ISO 27001)
- **Security score**: Gives you a score (0-100) showing how secure your environment is

**How it works:**
- Automatically discovers Azure resources
- Analyzes configurations and compares to best practices
- Monitors network traffic and activities
- Uses machine learning to detect anomalies
- Provides actionable recommendations

**Tiers:**
- **Free tier**: Basic security recommendations
- **Standard tier**: Advanced threat detection, compliance monitoring, security alerts

**Example recommendations:**
- "Enable encryption on storage accounts"
- "Fix SQL database vulnerabilities"
- "Enable MFA for subscription admins"
- "Remove unused network security rules"

### Q2: How do you use Azure Security Center to improve security posture?

**Answer:**
**Steps to improve security:**

1. **Enable Security Center**:
   - Go to Azure Portal → Security Center
   - Enable Standard tier for advanced features
   - Enable auto-provisioning of agents

2. **Review security score**:
   - See current score (0-100)
   - Identify areas with low scores
   - Prioritize recommendations by impact

3. **Address recommendations**:
   - **High severity**: Fix immediately (critical vulnerabilities)
   - **Medium severity**: Fix soon (security improvements)
   - **Low severity**: Plan to fix (best practices)

4. **Enable security policies**:
   - Set policies for resource groups
   - Enforce security standards
   - Get alerts when policies violated

5. **Set up alerts**:
   - Configure email notifications
   - Set up Logic Apps for automated responses
   - Integrate with SIEM systems

6. **Monitor compliance**:
   - Check compliance with standards (PCI-DSS, HIPAA)
   - Generate compliance reports
   - Fix non-compliant resources

**Example workflow:**
```
Security Center → Recommendations → "Enable encryption" → Click "Fix" → 
Azure Policy applies → Resources encrypted → Security score increases
```

**Best practices:**
- Review recommendations weekly
- Fix high-severity issues immediately
- Use security policies to prevent issues
- Monitor security score trends

## Azure Firewall

### Q3: What is Azure Firewall and how does it work?

**Answer:**
Azure Firewall is a managed, cloud-based network security service that protects your Azure Virtual Network resources.

**Key features:**
- **Network rules**: Allow/deny traffic based on source IP, destination IP, port, protocol
- **Application rules**: Control outbound access to FQDNs (fully qualified domain names)
- **Threat intelligence**: Automatically blocks traffic from known malicious IPs
- **FQDN tags**: Pre-configured groups of FQDNs (like Windows Update, Azure services)
- **Service tags**: Pre-defined groups of Azure service IP ranges

**How it works:**
- Deployed in your VNet (usually in dedicated subnet)
- All traffic routed through firewall
- Firewall inspects and filters traffic
- Stateful firewall (remembers connections)

**Example rules:**
```
Network Rule: Allow outbound HTTPS (443) to any IP
Application Rule: Allow access to *.microsoft.com
Threat Intelligence: Block known malicious IPs automatically
```

**Use cases:**
- Control outbound internet access from VNet
- Filter traffic between subnets
- Protect against known threats
- Enforce security policies

### Q4: Explain the difference between Azure Firewall and Network Security Groups (NSG).

**Answer:**
**Azure Firewall:**
- **Managed service**: Fully managed by Azure
- **Stateful**: Remembers connections
- **Application-aware**: Can filter by FQDN, URLs
- **Threat intelligence**: Blocks known malicious IPs
- **Centralized**: Single firewall for entire VNet
- **More expensive**: Pay for service
- **Use for**: Outbound internet control, application-level filtering, threat protection

**Network Security Groups (NSG):**
- **Resource-level**: Attach to subnets or network interfaces
- **Stateless**: Each rule evaluated independently
- **Network-level**: Filters by IP, port, protocol only
- **Distributed**: Multiple NSGs across resources
- **Free**: No additional cost
- **Use for**: Basic network filtering, subnet isolation, VM-level rules

**Comparison:**

| Feature | Azure Firewall | NSG |
|---------|----------------|-----|
| Management | Managed service | You manage rules |
| Stateful | Yes | No |
| Application filtering | Yes (FQDN) | No |
| Threat intelligence | Yes | No |
| Cost | Paid service | Free |
| Granularity | VNet level | Subnet/VM level |

**When to use both:**
- **NSG**: Basic filtering at subnet/VM level (defense in depth)
- **Azure Firewall**: Centralized outbound control, application filtering, threat protection

**Best practice**: Use NSG for basic filtering, Azure Firewall for advanced outbound control and threat protection.

## Encryption

### Q5: Explain encryption at rest vs encryption in transit.

**Answer:**
**Encryption at Rest:**
- Data encrypted when stored on disk/storage
- Protects data if storage is compromised
- Example: Encrypted hard drive, encrypted database

**Azure examples:**
- **Azure Storage**: Encryption enabled by default (Azure-managed keys or customer-managed keys)
- **Azure SQL Database**: Transparent Data Encryption (TDE)
- **Azure VMs**: Azure Disk Encryption
- **Azure Key Vault**: All data encrypted

**Encryption in Transit:**
- Data encrypted while traveling over network
- Protects data from interception
- Example: HTTPS, VPN, TLS

**Azure examples:**
- **HTTPS**: All Azure services use TLS/SSL
- **VPN**: Encrypted tunnel for site-to-site connections
- **Azure Service Bus**: TLS encryption
- **Azure Storage**: HTTPS required for secure access

**Why both matter:**
- **At rest**: Protects if someone steals hard drive or accesses storage
- **In transit**: Protects if someone intercepts network traffic

**Best practice**: Always enable both. Most Azure services have encryption at rest enabled by default, always use HTTPS for in-transit.

### Q6: How do you implement encryption in Azure?

**Answer:**
**1. Storage Account Encryption:**
```bash
# Enable encryption (enabled by default)
az storage account update \
  --name mystorageaccount \
  --resource-group myRG \
  --encryption-services blob file
```

**2. SQL Database Encryption (TDE):**
```sql
-- Enable Transparent Data Encryption
ALTER DATABASE MyDatabase
SET ENCRYPTION ON;
```

**3. VM Disk Encryption:**
```bash
# Enable Azure Disk Encryption
az vm encryption enable \
  --resource-group myRG \
  --name myVM \
  --disk-encryption-keyvault myKeyVault
```

**4. Key Vault for Key Management:**
- Store encryption keys in Azure Key Vault
- Use customer-managed keys instead of Azure-managed
- Control key rotation and access

**5. HTTPS/TLS:**
- Always use HTTPS for web applications
- Use Application Gateway with SSL certificates
- Enable TLS 1.2 or higher (disable older versions)

**6. VPN Encryption:**
- Site-to-site VPN uses IPSec encryption
- Point-to-site VPN uses SSL/TLS

**Best practices:**
- Use customer-managed keys for critical data
- Enable encryption by default
- Regularly rotate encryption keys
- Use Azure Key Vault for key management
- Enable encryption on all storage types

## SAST/DAST Scanning

### Q7: What is SAST and how do you implement it?

**Answer:**
SAST (Static Application Security Testing) analyzes source code to find security vulnerabilities without running the application.

**How it works:**
- Scans source code files
- Looks for security patterns (SQL injection, XSS, hardcoded secrets)
- Doesn't need running application
- Fast, can run in CI/CD pipeline

**Tools:**
- **SonarQube**: Popular open-source SAST tool
- **Checkmarx**: Commercial SAST solution
- **Veracode**: Cloud-based SAST
- **GitLab SAST**: Built into GitLab CI/CD
- **Azure DevOps**: Security scanning extensions

**Implementation in CI/CD:**
```yaml
# GitLab CI/CD example
sast:
  stage: test
  script:
    - sonar-scanner
  artifacts:
    reports:
      sast: sast-report.json
```

**What it finds:**
- SQL injection vulnerabilities
- Cross-site scripting (XSS)
- Hardcoded passwords/secrets
- Insecure cryptographic functions
- Authentication/authorization issues
- Insecure dependencies

**Benefits:**
- Find issues early (before deployment)
- Fast feedback
- Automated scanning
- Integrates with CI/CD

**Limitations:**
- May have false positives
- Can't find runtime issues
- Needs access to source code

### Q8: What is DAST and how does it differ from SAST?

**Answer:**
DAST (Dynamic Application Security Testing) tests running applications to find security vulnerabilities.

**How it works:**
- Tests application while it's running
- Sends requests and analyzes responses
- Simulates attacks (SQL injection, XSS)
- Finds runtime vulnerabilities

**Tools:**
- **OWASP ZAP**: Free, open-source DAST tool
- **Burp Suite**: Popular commercial tool
- **Acunetix**: Web vulnerability scanner
- **Azure Security Center**: Can perform DAST scans

**Implementation:**
```yaml
# CI/CD example
dast:
  stage: test
  script:
    - docker-compose up -d  # Start application
    - zap-baseline.py -t http://localhost:8080
  only:
    - main  # Run on main branch
```

**What it finds:**
- Runtime vulnerabilities
- Configuration issues
- Authentication/authorization flaws
- Session management problems
- API security issues

**SAST vs DAST:**

| Aspect | SAST | DAST |
|--------|------|------|
| When | Before deployment | After deployment |
| How | Scans source code | Tests running app |
| Speed | Fast | Slower |
| Finds | Code vulnerabilities | Runtime vulnerabilities |
| False positives | More | Fewer |
| Needs | Source code | Running application |

**Best practice**: Use both SAST and DAST for comprehensive security testing.

## Intrusion Detection Systems

### Q9: What is an IDS/IPS and how does Azure provide this?

**Answer:**
**IDS (Intrusion Detection System)**: Monitors network traffic and alerts on suspicious activity (detects but doesn't block).

**IPS (Intrusion Prevention System)**: Monitors and automatically blocks suspicious activity (detects and blocks).

**Azure provides:**

1. **Azure Security Center**:
   - Detects suspicious activities
   - Alerts on potential attacks
   - Uses machine learning for anomaly detection
   - Provides threat intelligence

2. **Azure Firewall Threat Intelligence**:
   - Blocks traffic from known malicious IPs
   - Uses Microsoft threat intelligence feeds
   - Automatic blocking (IPS-like behavior)

3. **Azure DDoS Protection**:
   - Detects DDoS attacks
   - Automatically mitigates attacks
   - Standard tier provides advanced protection

4. **Network Watcher**:
   - Monitors network traffic
   - Detects anomalies
   - Can be used for intrusion detection

5. **Azure Sentinel** (SIEM):
   - Security information and event management
   - Collects logs from all sources
   - Uses AI for threat detection
   - Automated response

**Example detection:**
- Unusual login patterns
- Traffic from known malicious IPs
- Port scanning attempts
- Brute force attacks
- Data exfiltration attempts

**Best practices:**
- Enable Azure Security Center Standard tier
- Enable Azure Firewall threat intelligence
- Use Azure Sentinel for centralized monitoring
- Set up alerts for suspicious activities
- Review and tune detection rules

### Q10: How do you implement security monitoring and alerting in Azure?

**Answer:**
**1. Azure Security Center:**
- Enable Standard tier
- Configure security alerts
- Set up email notifications
- Integrate with Azure Sentinel

**2. Azure Monitor:**
- Collect logs from all resources
- Create alert rules
- Set up action groups (email, SMS, webhooks)
- Use Log Analytics queries

**Example alert rule:**
```bash
# Alert when failed login attempts exceed threshold
az monitor metrics alert create \
  --name "Multiple Failed Logins" \
  --resource-group myRG \
  --scopes /subscriptions/.../resourceGroups/myRG \
  --condition "count FailedLogins > 5" \
  --window-size 5m
```

**3. Azure Sentinel:**
- Centralized SIEM
- Collects logs from Azure and on-premises
- Uses AI for threat detection
- Automated response playbooks

**4. Log Analytics:**
- Query security logs
- Create custom alerts
- Generate reports

**Example query:**
```kql
SecurityEvent
| where EventID == 4625  // Failed login
| where TimeGenerated > ago(1h)
| summarize count() by Account
| where count_ > 5
```

**5. Action Groups:**
- Configure who gets notified
- Multiple channels (email, SMS, webhook)
- Escalation paths

**Best practices:**
- Enable logging on all resources
- Centralize logs in Log Analytics
- Set up alerts for critical events
- Test alerting regularly
- Review and tune alerts
- Use Azure Sentinel for advanced threat detection

## Security Scenarios

### Q11: How would you secure an Azure environment for a healthcare application (HIPAA compliance)?

**Answer:**
**1. Network Security:**
- Use VNets with subnets (isolate tiers)
- NSGs to restrict traffic
- Azure Firewall for outbound control
- Private Endpoints for Azure services (no public internet)
- VPN Gateway for secure on-premises connection

**2. Identity & Access:**
- Azure AD with MFA required
- Role-Based Access Control (RBAC) - least privilege
- Managed Identities (no passwords)
- Regular access reviews
- Audit logs for all access

**3. Data Protection:**
- Encryption at rest (all storage, databases)
- Encryption in transit (HTTPS, TLS 1.2+)
- Azure SQL Database with TDE
- Azure Disk Encryption for VMs
- Customer-managed keys in Key Vault

**4. Monitoring & Compliance:**
- Azure Security Center Standard tier
- Enable HIPAA compliance policies
- Continuous security monitoring
- Regular security assessments
- Audit logs retention (HIPAA requires 6 years)

**5. Application Security:**
- WAF on Application Gateway
- SAST/DAST scanning in CI/CD
- Regular vulnerability scanning
- Secure coding practices
- Input validation

**6. Backup & Recovery:**
- Encrypted backups
- Geo-redundant storage
- Regular backup testing
- Documented recovery procedures

**7. Administrative Controls:**
- Separate admin accounts
- Just-in-time access for VMs
- Azure Bastion for secure access
- No direct RDP/SSH from internet

**8. Compliance:**
- Azure Policy for HIPAA compliance
- Regular compliance assessments
- Documentation of security controls
- Incident response plan

### Q12: Explain how you would implement defense in depth for Azure resources.

**Answer:**
Defense in depth means multiple layers of security (if one fails, others protect).

**Layer 1: Network Perimeter:**
- DDoS Protection
- Azure Firewall
- WAF on Application Gateway
- NSGs

**Layer 2: Identity & Access:**
- Azure AD with MFA
- RBAC (least privilege)
- Managed Identities
- Conditional Access policies

**Layer 3: Compute:**
- Secure VM configurations
- Regular patching
- Antimalware (Microsoft Defender)
- Just-in-time VM access
- Azure Bastion (no public IPs)

**Layer 4: Application:**
- Secure coding practices
- SAST/DAST scanning
- Input validation
- Authentication/authorization
- Session management

**Layer 5: Data:**
- Encryption at rest
- Encryption in transit
- Key management (Key Vault)
- Data classification
- Access controls

**Layer 6: Monitoring:**
- Azure Security Center
- Azure Monitor
- Log Analytics
- Security alerts
- Threat detection

**Example implementation:**
```
Internet → DDoS Protection → WAF → Application Gateway → NSG → VM
                                                              ↓
                                                      Azure Security Center
                                                              ↓
                                                      Log Analytics
                                                              ↓
                                                      Security Alerts
```

**Benefits:**
- Multiple protection layers
- If one layer fails, others protect
- Comprehensive security coverage
- Meets compliance requirements

### Q13: How would you respond to a security breach in Azure?

**Answer:**
**Immediate Response (First Hour):**

1. **Contain the breach:**
   - Isolate affected resources
   - Block malicious IPs in Azure Firewall
   - Disable compromised accounts
   - Change compromised credentials

2. **Assess the impact:**
   - What resources are affected?
   - What data was accessed?
   - How did attacker get in?
   - Is breach ongoing?

3. **Preserve evidence:**
   - Enable logging if not already
   - Export logs immediately
   - Take snapshots of affected resources
   - Don't delete anything yet

**Short-term (First Day):**

4. **Notify stakeholders:**
   - Security team
   - Management
   - Legal/compliance (if required)
   - Customers (if data breach)

5. **Investigate:**
   - Review Azure Activity Logs
   - Check Security Center alerts
   - Analyze network traffic
   - Review access logs

6. **Remediate:**
   - Patch vulnerabilities
   - Remove malicious code
   - Reset all credentials
   - Restore from clean backups if needed

**Long-term (First Week):**

7. **Root cause analysis:**
   - How did breach occur?
   - What vulnerabilities were exploited?
   - What controls failed?

8. **Implement fixes:**
   - Patch all vulnerabilities
   - Strengthen security controls
   - Update security policies
   - Improve monitoring

9. **Document:**
   - Incident report
   - Lessons learned
   - Action items
   - Update runbooks

**Prevention:**
- Regular security assessments
- Penetration testing
- Security training
- Continuous monitoring
- Incident response plan (test regularly)

**Azure tools for response:**
- Azure Security Center: Threat detection
- Azure Monitor: Log analysis
- Azure Sentinel: SIEM and automated response
- Network Watcher: Network forensics
- Key Vault: Rotate compromised keys

