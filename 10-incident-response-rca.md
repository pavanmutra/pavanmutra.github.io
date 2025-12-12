# Incident Response & RCA - Interview Q&A

## Incident Response Procedures

### Q1: What is incident response and what are the key phases?

**Answer:**
Incident response is the process of handling security breaches or system failures to minimize damage and restore normal operations quickly.

**Key phases (NIST framework):**

**1. Preparation:**
- Create incident response plan
- Train team
- Set up tools and monitoring
- Define roles and responsibilities
- Test procedures

**2. Detection & Analysis:**
- Identify incident (monitoring, alerts, user reports)
- Classify severity (critical, high, medium, low)
- Gather information
- Determine scope

**3. Containment:**
- Short-term: Immediate actions to stop damage
- Long-term: Isolate affected systems
- Prevent further spread

**4. Eradication:**
- Remove threat/vulnerability
- Patch systems
- Remove malicious code
- Clean infected systems

**5. Recovery:**
- Restore systems to normal operation
- Verify systems are clean
- Monitor for recurrence
- Gradually restore services

**6. Post-Incident Activity:**
- Document incident
- Root cause analysis
- Lessons learned
- Update procedures
- Improve security

**Time is critical**: Faster response = less damage.

### Q2: Explain the incident severity classification.

**Answer:**
**Severity levels:**

**Critical (P1):**
- Complete system outage
- Data breach
- Security compromise
- Affects all users
- **Response time**: Immediate (within 15 minutes)
- **Example**: Production database down, ransomware attack

**High (P2):**
- Major feature unavailable
- Significant performance degradation
- Affects many users
- **Response time**: Within 1 hour
- **Example**: Payment processing down, 50% of users affected

**Medium (P3):**
- Minor feature unavailable
- Some users affected
- Workaround available
- **Response time**: Within 4 hours
- **Example**: Report generation slow, specific feature broken

**Low (P4):**
- Cosmetic issues
- Minor bugs
- Few users affected
- **Response time**: Within 24 hours
- **Example**: UI typo, minor display issue

**Classification factors:**
- Number of users affected
- Business impact
- Data exposure
- System availability
- Workaround availability

**Best practice**: Have clear definitions and examples for each severity level.

### Q3: How do you set up an incident response team?

**Answer:**
**Team roles:**

**1. Incident Commander:**
- Overall coordination
- Makes decisions
- Communicates with stakeholders
- Manages timeline

**2. Technical Lead:**
- Technical investigation
- Coordinates technical team
- Determines root cause
- Implements fixes

**3. Communications Lead:**
- Internal communications
- External communications (if needed)
- Status updates
- Documentation

**4. Security Specialist:**
- Security investigation
- Threat analysis
- Forensics
- Security remediation

**5. Operations Engineer:**
- System recovery
- Infrastructure changes
- Monitoring
- Deployment

**6. Developer:**
- Code fixes
- Application changes
- Testing
- Deployment

**On-call rotation:**
- 24/7 coverage for critical incidents
- Escalation path defined
- Backup on-call engineer
- Clear handoff procedures

**Tools needed:**
- Incident management system (PagerDuty, Opsgenie)
- Communication (Slack, Teams)
- Monitoring (Azure Monitor, Application Insights)
- Documentation (Confluence, Wiki)

**Best practices:**
- Regular training
- Practice drills
- Clear escalation procedures
- Documented runbooks
- Post-incident reviews

## Root Cause Analysis

### Q4: What is Root Cause Analysis (RCA) and why is it important?

**Answer:**
Root Cause Analysis is a method to identify the underlying cause of an incident, not just the symptoms.

**Why important:**
- **Prevent recurrence**: Fix root cause, not just symptoms
- **Systemic improvements**: Improve processes and systems
- **Learning**: Understand what went wrong
- **Documentation**: Record for future reference
- **Continuous improvement**: Make systems more resilient

**Common mistake**: Fixing symptoms instead of root cause.

**Example:**
- **Symptom**: Application crashes
- **Immediate cause**: Out of memory
- **Root cause**: Memory leak in code (not fixed) → will happen again
- **Fix symptom**: Restart application (temporary)
- **Fix root cause**: Fix memory leak (permanent)

### Q5: Explain the 5 Whys technique for RCA.

**Answer:**
The 5 Whys is a simple technique: ask "why" five times to get to root cause.

**Example:**
**Problem**: Production website is down

1. **Why is the website down?**
   - Server crashed

2. **Why did the server crash?**
   - Out of memory

3. **Why did it run out of memory?**
   - Memory leak in application code

4. **Why is there a memory leak?**
   - Code wasn't properly tested for memory leaks

5. **Why wasn't it tested?**
   - No memory testing in CI/CD pipeline

**Root cause**: Missing memory testing in CI/CD pipeline.

**Solution**: Add memory leak detection to CI/CD pipeline.

**Tips:**
- Don't stop at first answer
- Be specific
- Focus on process, not people
- May need more or fewer than 5 whys
- Verify root cause makes sense

**Limitations:**
- Can oversimplify complex issues
- May have multiple root causes
- Need to verify with data

### Q6: Explain other RCA techniques.

**Answer:**
**1. Fishbone Diagram (Ishikawa):**
- Visual diagram showing all possible causes
- Categories: People, Process, Technology, Environment
- Helps identify all contributing factors

**2. Fault Tree Analysis:**
- Tree structure showing how failures lead to incident
- Logical relationships (AND, OR gates)
- Good for complex systems

**3. Timeline Analysis:**
- Chronological sequence of events
- Identify what happened when
- Find critical events

**4. Change Analysis:**
- What changed before incident?
- Recent deployments, config changes, updates
- Often root cause is recent change

**5. Barrier Analysis:**
- What barriers should have prevented incident?
- Why did barriers fail?
- How to strengthen barriers?

**Example timeline:**
```
10:00 AM - Code deployment
10:05 AM - Application restarted
10:10 AM - Memory usage starts increasing
10:30 AM - Memory at 80%
11:00 AM - Memory at 95%
11:15 AM - Server crashes
```

**Root cause**: Code deployment at 10:00 AM introduced memory leak.

**Best practice**: Use multiple techniques, combine findings, verify with data.

### Q7: How do you conduct a post-incident review?

**Answer:**
**Post-incident review (blameless):**

**1. Schedule meeting:**
- Within 48 hours of incident resolution
- Include all involved team members
- Keep it blameless (focus on process, not people)

**2. Prepare:**
- Gather timeline of events
- Collect logs and metrics
- Document what happened
- Prepare questions

**3. Meeting structure:**

**a. What happened?**
- Timeline of events
- What was the incident?
- When did it start/end?
- Who was involved?

**b. Impact assessment:**
- How many users affected?
- Duration of incident?
- Business impact?
- Data loss?

**c. What went well?**
- What worked in response?
- Good decisions made?
- Effective communication?

**d. What went wrong?**
- What didn't work?
- Delays in response?
- Communication issues?
- Missing information?

**e. Root cause:**
- Use RCA techniques
- Identify underlying cause
- Verify with data

**f. Action items:**
- What needs to be fixed?
- Who owns each action?
- Timeline for completion?
- How to prevent recurrence?

**4. Document:**
- Incident report
- Timeline
- Root cause
- Action items
- Lessons learned

**5. Follow-up:**
- Track action items
- Verify fixes implemented
- Test improvements
- Update runbooks

**Example template:**
```
Incident: Production database outage
Date: 2024-01-15
Duration: 2 hours
Impact: All users unable to access application

Timeline:
- 10:00 AM - Database connection pool exhausted
- 10:15 AM - Alert triggered
- 10:30 AM - Team notified
- 11:00 AM - Root cause identified
- 11:30 AM - Fix deployed
- 12:00 PM - Service restored

Root Cause: Connection pool not properly configured after recent deployment

Action Items:
1. Fix connection pool configuration (Owner: Dev Team, Due: 2024-01-17)
2. Add connection pool monitoring (Owner: DevOps, Due: 2024-01-20)
3. Update deployment checklist (Owner: DevOps, Due: 2024-01-18)
```

## Corrective Actions

### Q8: How do you implement corrective actions after an incident?

**Answer:**
**Implementation process:**

**1. Prioritize actions:**
- **Critical**: Prevents recurrence (fix root cause)
- **High**: Significant improvement
- **Medium**: Nice to have
- **Low**: Future consideration

**2. Assign owners:**
- Clear ownership for each action
- Set deadlines
- Regular check-ins

**3. Implement fixes:**

**Immediate (within 24 hours):**
- Fix root cause
- Restore service
- Add temporary monitoring

**Short-term (within 1 week):**
- Permanent fixes
- Improve monitoring
- Update documentation
- Add automated checks

**Long-term (within 1 month):**
- Process improvements
- Training
- Tool improvements
- Architecture changes

**4. Verify fixes:**
- Test that fix works
- Verify monitoring catches issue
- Confirm documentation updated
- Test in staging before production

**5. Track progress:**
- Use tracking system (Jira, Azure DevOps)
- Regular status updates
- Close when complete
- Review at team meetings

**Example action items:**
```
1. [Critical] Fix memory leak in user service
   Owner: Dev Team
   Due: 2024-01-17
   Status: In Progress

2. [High] Add memory monitoring alerts
   Owner: DevOps
   Due: 2024-01-20
   Status: Not Started

3. [Medium] Update deployment checklist
   Owner: DevOps
   Due: 2024-01-18
   Status: Not Started
```

**Best practices:**
- Focus on preventing recurrence
- Don't just fix symptoms
- Improve processes, not just code
- Set realistic deadlines
- Follow up regularly

### Q9: Explain how to prevent future incidents.

**Answer:**
**Prevention strategies:**

**1. Fix root causes:**
- Don't just fix symptoms
- Address underlying issues
- Prevent recurrence

**2. Improve monitoring:**
- Add alerts for issues that caused incident
- Better visibility into system
- Proactive detection

**3. Add automated checks:**
- Pre-deployment validation
- Health checks
- Automated testing
- Configuration validation

**4. Improve processes:**
- Better deployment procedures
- Code review requirements
- Testing requirements
- Change management

**5. Training:**
- Train team on incident response
- Share lessons learned
- Best practices
- Tool training

**6. Documentation:**
- Update runbooks
- Document procedures
- Create playbooks
- Knowledge base

**7. Testing:**
- Regular disaster recovery drills
- Chaos engineering
- Load testing
- Security testing

**8. Architecture improvements:**
- Remove single points of failure
- Add redundancy
- Improve resilience
- Better error handling

**Example prevention measures:**
```
Incident: Database connection pool exhausted

Prevention:
1. Add connection pool monitoring (alert when > 80%)
2. Add pre-deployment validation (check connection pool config)
3. Add automated testing (load test connection pool)
4. Update deployment checklist (verify connection pool settings)
5. Add circuit breaker pattern (prevent cascade failures)
```

**Best practice**: Every incident should result in improvements to prevent similar incidents.

## Real-World Scenarios

### Q10: Describe how you would handle a production database outage.

**Answer:**
**Immediate response (0-15 minutes):**

1. **Acknowledge incident:**
   - Alert received or user report
   - Confirm issue (can't connect to database)
   - Classify as Critical (P1)

2. **Assess impact:**
   - All users affected?
   - Which services down?
   - Data loss risk?

3. **Notify team:**
   - Page on-call engineer
   - Notify incident commander
   - Set up war room (Teams/Slack channel)

4. **Initial investigation:**
   - Check database status in Azure Portal
   - Check connection strings
   - Review recent changes
   - Check monitoring/alerts

**Containment (15-60 minutes):**

5. **Stop the bleeding:**
   - If database server down: Check Azure status, restart if needed
   - If connection issues: Check network, firewall rules
   - If overloaded: Scale up database, add read replicas

6. **Gather information:**
   - Database logs
   - Application logs
   - Recent deployments
   - Configuration changes

**Resolution (1-4 hours):**

7. **Identify root cause:**
   - Use RCA techniques
   - Check recent changes
   - Analyze logs
   - Verify with data

8. **Implement fix:**
   - If configuration issue: Fix and deploy
   - If code issue: Rollback or hotfix
   - If infrastructure: Scale or restart
   - If external: Contact vendor

9. **Verify fix:**
   - Test connectivity
   - Verify application works
   - Check monitoring
   - Confirm with users

**Recovery (4-24 hours):**

10. **Restore service:**
    - Gradually restore services
    - Monitor closely
    - Verify data integrity
    - Confirm all features work

11. **Communication:**
    - Update status page
    - Notify stakeholders
    - Post-mortem scheduled

**Post-incident (1-7 days):**

12. **Post-mortem:**
    - Timeline of events
    - Root cause analysis
    - What went well/wrong
    - Action items

13. **Implement improvements:**
    - Fix root cause
    - Add monitoring
    - Update procedures
    - Improve architecture

**Example timeline:**
```
10:00 AM - Database connection errors start
10:05 AM - Alert triggered, on-call paged
10:10 AM - Team assembled, investigation starts
10:15 AM - Identified: Database server restarted (Azure maintenance)
10:20 AM - Database back online, connections restored
10:25 AM - Application fully operational
10:30 AM - Post-mortem scheduled
```

**Prevention:**
- Add database failover (always-on, geo-replication)
- Better monitoring (alert on database restarts)
- Maintenance window notifications
- Connection retry logic in application

### Q11: How would you handle a security breach incident?

**Answer:**
**Immediate response (0-30 minutes):**

1. **Contain breach:**
   - Isolate affected systems
   - Block malicious IPs
   - Disable compromised accounts
   - Change compromised credentials
   - Preserve evidence (don't delete logs)

2. **Assess impact:**
   - What systems affected?
   - What data accessed?
   - How did attacker get in?
   - Is breach ongoing?

3. **Notify:**
   - Security team
   - Management
   - Legal/compliance (if data breach)
   - Law enforcement (if required)

**Investigation (1-24 hours):**

4. **Gather evidence:**
   - Export all logs
   - Take snapshots of affected systems
   - Preserve network traffic
   - Document everything

5. **Investigate:**
   - Review Azure Activity Logs
   - Check Security Center alerts
   - Analyze network traffic
   - Review access logs
   - Identify attack vector

**Remediation (1-7 days):**

6. **Remove threat:**
   - Patch vulnerabilities
   - Remove malicious code
   - Clean infected systems
   - Reset all credentials
   - Update security controls

7. **Restore systems:**
   - Verify systems clean
   - Restore from clean backups if needed
   - Gradually restore services
   - Monitor for recurrence

**Post-incident (1-4 weeks):**

8. **Root cause analysis:**
   - How did breach occur?
   - What vulnerabilities exploited?
   - What controls failed?

9. **Implement improvements:**
   - Patch all vulnerabilities
   - Strengthen security controls
   - Improve monitoring
   - Update security policies
   - Training

10. **Documentation:**
    - Incident report
    - Lessons learned
    - Action items
    - Compliance reporting (if required)

**Azure tools:**
- Azure Security Center: Threat detection
- Azure Monitor: Log analysis
- Azure Sentinel: SIEM and automated response
- Network Watcher: Network forensics
- Key Vault: Rotate compromised keys

**Example:**
```
Incident: Unauthorized access to storage account
Time: 2024-01-15 2:00 AM
Detection: Unusual access pattern alert

Response:
- 2:05 AM - Isolated storage account
- 2:10 AM - Blocked source IP
- 2:15 AM - Changed storage account keys
- 2:20 AM - Reviewed access logs
- 2:30 AM - Identified: Exposed access key in code repository
- 3:00 AM - Rotated all keys, removed exposed key
- 3:30 AM - Scanned codebase for other exposed secrets
- 4:00 AM - Implemented secret scanning in CI/CD

Root Cause: Access key committed to public repository

Prevention:
- Add secret scanning to CI/CD
- Use Key Vault for secrets
- Implement managed identities
- Regular security audits
```

