# Monitoring & SDLC - Interview Q&A

## Application Health Monitoring

### Q1: What is application health monitoring and why is it important?

**Answer:**
Application health monitoring tracks whether your application is working correctly and performing well. It's important because:
- **Detect issues early**: Find problems before users notice
- **Minimize downtime**: Quick response to failures
- **Performance optimization**: Identify slow operations
- **User experience**: Ensure users have good experience
- **Business impact**: Downtime costs money and reputation

**What to monitor:**
- **Availability**: Is application up and responding?
- **Response time**: How fast does it respond?
- **Error rates**: How many requests fail?
- **Resource usage**: CPU, memory, disk, network
- **Business metrics**: User signups, transactions, revenue

**Monitoring layers:**
1. **Infrastructure**: Servers, networks, databases
2. **Application**: Code performance, errors, logs
3. **User experience**: Page load times, user actions
4. **Business**: Key business metrics

### Q2: Explain Azure Application Insights and its features.

**Answer:**
Azure Application Insights is an application performance management (APM) service that monitors live applications.

**Key features:**
- **Application performance**: Response times, request rates, failure rates
- **Dependency tracking**: Database calls, external API calls, their performance
- **Exception tracking**: Automatic exception logging with stack traces
- **Custom metrics**: Track business-specific metrics
- **Live metrics**: Real-time monitoring
- **Smart detection**: AI-powered anomaly detection
- **Availability tests**: Monitor website availability from multiple locations
- **User analytics**: Understand how users interact with application

**How it works:**
- SDK installed in application
- Sends telemetry to Application Insights
- Data stored in Azure
- Visualized in Azure Portal

**Example integration (Node.js):**
```javascript
const appInsights = require("applicationinsights");
appInsights.setup("your-instrumentation-key");
appInsights.start();

// Automatic tracking of HTTP requests, exceptions
// Custom tracking
appInsights.defaultClient.trackEvent({
  name: "UserSignup",
  properties: { userId: "123" }
});
```

**Use cases:**
- Monitor application performance
- Debug production issues
- Track user behavior
- Set up alerts
- Analyze trends

### Q3: How do you set up health checks for applications?

**Answer:**
**1. Create health endpoint:**
```javascript
// Node.js example
app.get('/health', (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    checks: {
      database: checkDatabase(),
      cache: checkCache(),
      externalAPI: checkExternalAPI()
    }
  };
  
  const isHealthy = Object.values(health.checks).every(check => check.status === 'ok');
  res.status(isHealthy ? 200 : 503).json(health);
});
```

**2. Azure App Service health checks:**
```bash
# Enable health check
az webapp config set \
  --resource-group myRG \
  --name myapp \
  --generic-configurations '{"healthCheckPath": "/health"}'
```

**3. Load balancer health probes:**
```bash
# Azure Load Balancer
az network lb probe create \
  --resource-group myRG \
  --lb-name myLB \
  --name healthProbe \
  --protocol Http \
  --port 80 \
  --path /health
```

**4. Application Gateway health probe:**
```bash
az network application-gateway http-settings update \
  --resource-group myRG \
  --gateway-name myAG \
  --name appGatewayBackendHttpSettings \
  --probe healthProbe
```

**5. Kubernetes liveness/readiness probes:**
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 5
```

**Health check should verify:**
- Application is running
- Database connectivity
- External dependencies
- Critical services available

### Q4: Explain different types of monitoring metrics.

**Answer:**
**1. Availability Metrics:**
- Uptime percentage
- Response codes (200, 404, 500)
- Health check status
- Example: 99.9% uptime means 8.76 hours downtime per year

**2. Performance Metrics:**
- Response time (p50, p95, p99 percentiles)
- Throughput (requests per second)
- Latency
- Example: p95 response time = 95% of requests faster than this

**3. Error Metrics:**
- Error rate (errors per total requests)
- Exception count
- Failed requests
- Example: 0.1% error rate = 1 error per 1000 requests

**4. Resource Metrics:**
- CPU usage
- Memory usage
- Disk I/O
- Network bandwidth
- Example: CPU at 80% = high utilization, might need scaling

**5. Business Metrics:**
- User signups
- Transactions
- Revenue
- Conversion rates
- Example: Track signups per day

**6. Custom Metrics:**
- Application-specific
- Business logic metrics
- Example: Orders processed, emails sent

**Key metrics to track:**
- **Four Golden Signals**: Latency, Traffic, Errors, Saturation
- **RED method**: Rate, Errors, Duration
- **USE method**: Utilization, Saturation, Errors

## Security Monitoring

### Q5: How do you monitor security in Azure?

**Answer:**
**1. Azure Security Center:**
- Continuous security monitoring
- Threat detection
- Security recommendations
- Compliance monitoring

**2. Azure Monitor:**
- Collect security logs
- Create security alerts
- Track authentication events
- Monitor access patterns

**3. Azure Sentinel (SIEM):**
- Centralized security monitoring
- Threat detection using AI
- Security incident management
- Automated response

**4. Log Analytics:**
- Query security logs
- Create custom alerts
- Generate security reports

**Example security queries:**
```kql
// Failed login attempts
SigninLogs
| where ResultType != "0"
| summarize count() by UserPrincipalName, bin(TimeGenerated, 1h)
| where count_ > 5

// Unusual access patterns
AzureActivity
| where OperationName == "Microsoft.Compute/virtualMachines/write"
| summarize count() by Caller, bin(TimeGenerated, 1h)
```

**5. Network Watcher:**
- Monitor network traffic
- Detect anomalies
- Network security group flow logs

**6. Application Insights:**
- Track authentication events
- Monitor API access
- Detect unusual patterns

**Best practices:**
- Enable logging on all resources
- Centralize logs in Log Analytics
- Set up alerts for suspicious activities
- Regular security reviews
- Use Azure Sentinel for advanced threat detection

### Q6: Explain how to set up security alerts in Azure.

**Answer:**
**1. Azure Security Center alerts:**
- Automatically enabled in Standard tier
- Alerts on threats and suspicious activities
- Configure email notifications

**2. Azure Monitor alert rules:**
```bash
# Create alert for failed logins
az monitor metrics alert create \
  --name "Multiple Failed Logins" \
  --resource-group myRG \
  --scopes /subscriptions/.../resourceGroups/myRG \
  --condition "count FailedLogins > 5" \
  --window-size 5m \
  --evaluation-frequency 1m
```

**3. Log Analytics alerts:**
```kql
// Query for alert
SigninLogs
| where ResultType != "0"
| where TimeGenerated > ago(1h)
| summarize FailedAttempts = count() by UserPrincipalName
| where FailedAttempts > 5
```

**4. Action Groups (who gets notified):**
```bash
az monitor action-group create \
  --resource-group myRG \
  --name security-alerts \
  --short-name sec-alert \
  --email-receivers name=security-team email=security@company.com
```

**5. Azure Sentinel:**
- Automated security playbooks
- AI-powered threat detection
- Incident management

**Common security alerts:**
- Multiple failed login attempts
- Unusual access from new location
- Privileged account changes
- Resource creation/deletion
- Network anomalies
- Malware detection

**Best practices:**
- Set appropriate thresholds (not too sensitive)
- Use action groups for notifications
- Create runbooks for automated response
- Review and tune alerts regularly
- Test alerting system

## SDLC Process Monitoring

### Q7: How do you monitor the Software Development Lifecycle (SDLC)?

**Answer:**
**1. Source Control Metrics:**
- Commit frequency
- Code review time
- Branch lifecycle
- Merge frequency

**2. Build Metrics:**
- Build success rate
- Build duration
- Build frequency
- Failed build reasons

**3. Test Metrics:**
- Test coverage
- Test pass rate
- Test execution time
- Defect detection rate

**4. Deployment Metrics:**
- Deployment frequency
- Deployment success rate
- Deployment duration
- Rollback rate
- Time to production

**5. Quality Metrics:**
- Code quality scores
- Technical debt
- Security vulnerabilities
- Performance benchmarks

**Tools:**
- **Azure DevOps**: Built-in dashboards and reports
- **GitLab**: Analytics and insights
- **SonarQube**: Code quality metrics
- **Application Insights**: Application performance
- **Custom dashboards**: Power BI, Grafana

**Example dashboard:**
```
SDLC Metrics Dashboard:
- Commits this week: 150
- Build success rate: 98%
- Average build time: 5 minutes
- Test coverage: 85%
- Deployments this month: 45
- Mean time to recovery: 15 minutes
```

**Benefits:**
- Identify bottlenecks
- Improve processes
- Track team performance
- Make data-driven decisions

### Q8: Explain how to track deployment metrics.

**Answer:**
**Key deployment metrics:**

**1. Deployment Frequency:**
- How often you deploy
- Target: Multiple times per day (continuous deployment)
- Track: Number of deployments per day/week/month

**2. Lead Time:**
- Time from code commit to production
- Target: Hours to days
- Track: Average time per deployment

**3. Change Failure Rate:**
- Percentage of deployments that fail
- Target: < 5%
- Track: Failed deployments / Total deployments

**4. Mean Time to Recovery (MTTR):**
- Time to recover from failure
- Target: < 1 hour
- Track: Average recovery time

**Tracking in Azure DevOps:**
```yaml
# Pipeline with metrics
- task: PublishPipelineArtifact@1
  inputs:
    artifactName: 'deployment-metrics'
    path: 'metrics.json'
```

**Custom tracking:**
```javascript
// Track deployment in Application Insights
appInsights.defaultClient.trackEvent({
  name: "Deployment",
  properties: {
    environment: "production",
    version: "1.2.3",
    duration: 300, // seconds
    success: true
  }
});
```

**Dashboard example:**
- Deployments today: 12
- Average deployment time: 8 minutes
- Success rate: 98%
- Average recovery time: 12 minutes

**Best practices:**
- Automate metric collection
- Create dashboards
- Set targets and track progress
- Review metrics regularly
- Use metrics to improve processes

## Azure Monitoring Tools

### Q9: Explain Azure Monitor and its components.

**Answer:**
Azure Monitor is a comprehensive monitoring solution for Azure resources and applications.

**Components:**

**1. Metrics:**
- Numerical values over time
- Example: CPU usage, request count
- Stored for 93 days
- Real-time, low latency

**2. Logs:**
- Text data with timestamps
- Example: Application logs, audit logs
- Stored in Log Analytics workspace
- Queryable with KQL (Kusto Query Language)

**3. Alerts:**
- Notifications when conditions met
- Based on metrics or logs
- Can trigger actions (email, webhook, runbook)

**4. Dashboards:**
- Visual representation of metrics/logs
- Customizable views
- Share with team

**5. Workbooks:**
- Interactive reports
- Combine metrics, logs, visualizations
- Template-based

**6. Application Insights:**
- APM for applications
- Performance monitoring
- User analytics

**Example log query:**
```kql
// Failed requests in last hour
requests
| where timestamp > ago(1h)
| where success == false
| summarize count() by bin(timestamp, 5m), resultCode
| render timechart
```

**Use cases:**
- Monitor resource health
- Debug issues
- Performance optimization
- Capacity planning
- Compliance reporting

### Q10: How do you create effective monitoring dashboards?

**Answer:**
**Dashboard design principles:**

**1. Know your audience:**
- Executives: High-level business metrics
- Operations: Technical metrics, alerts
- Developers: Application performance, errors

**2. Key metrics first:**
- Most important metrics at top
- Use visual hierarchy
- Group related metrics

**3. Use appropriate visualizations:**
- Time series: Line charts for trends
- Current status: Gauges, single stat
- Comparisons: Bar charts
- Distributions: Pie charts, histograms

**4. Set thresholds:**
- Green: Healthy
- Yellow: Warning
- Red: Critical
- Use color coding

**5. Real-time vs historical:**
- Real-time: Current status, alerts
- Historical: Trends, patterns

**Example dashboard layout:**
```
┌─────────────────────────────────────┐
│  System Health (Overall)             │
│  [Gauge: 98%]                        │
├─────────────────────────────────────┤
│  Response Time (Last 24h)            │
│  [Line Chart: p50, p95, p99]        │
├─────────────────────────────────────┤
│  Error Rate                          │
│  [Bar Chart: Errors by hour]        │
├─────────────────────────────────────┤
│  Resource Usage                      │
│  [Line Chart: CPU, Memory, Disk]    │
└─────────────────────────────────────┘
```

**Azure Monitor Dashboard:**
```bash
# Create dashboard
az portal dashboard create \
  --resource-group myRG \
  --name "Production Dashboard" \
  --input-path dashboard.json
```

**Best practices:**
- Keep it simple (don't overload)
- Update regularly
- Make it actionable
- Use consistent colors
- Include time ranges
- Add context (what's normal?)

