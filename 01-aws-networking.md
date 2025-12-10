# AWS Networking: Complete Guide for DevOps Engineers

## Table of Contents
1. [Core Concepts](#core-concepts)
2. [VPC Architecture](#vpc-architecture)
3. [Subnets and Routing](#subnets-and-routing)
4. [Internet Gateway and NAT](#internet-gateway-and-nat)
5. [Security Groups and NACLs](#security-groups-and-nacls)
6. [Load Balancers](#load-balancers)
7. [Traffic Flow Scenarios](#traffic-flow-scenarios)
8. [Common Interview Scenarios](#common-interview-scenarios)

---

## Core Concepts

### What is a VPC?
A **Virtual Private Cloud (VPC)** is your own isolated network in AWS. Think of it as your private data center in the cloud. It's completely isolated from other customers' VPCs.

**Key Points:**
- Each VPC has a CIDR block (e.g., 10.0.0.0/16)
- You can create multiple VPCs in different regions
- VPCs are region-specific
- By default, VPCs cannot communicate with each other (unless you set up peering/VPN)

### IP Addressing in AWS
- **Private IP ranges:**
  - 10.0.0.0/8 (10.0.0.0 to 10.255.255.255)
  - 172.16.0.0/12 (172.16.0.0 to 172.31.255.255)
  - 192.168.0.0/16 (192.168.0.0 to 192.168.255.255)
- **Public IP:** Assigned by AWS from AWS's pool (not your VPC CIDR)

---

## VPC Architecture

### Basic VPC Components

```
┌─────────────────────────────────────────────────┐
│                    VPC                           │
│              (10.0.0.0/16)                      │
│                                                  │
│  ┌──────────────┐      ┌──────────────┐        │
│  │ Public Subnet│      │ Private Subnet│       │
│  │ 10.0.1.0/24  │      │ 10.0.2.0/24  │        │
│  │              │      │              │        │
│  │ EC2 Instance │      │ EC2 Instance │        │
│  │ (Public IP)  │      │ (Private IP) │        │
│  └──────────────┘      └──────────────┘        │
│         │                      │                │
│         └──────────┬───────────┘                │
│                   │                             │
│            ┌──────▼──────┐                      │
│            │ Route Table │                      │
│            └─────────────┘                      │
│                                                  │
└─────────────────────────────────────────────────┘
         │                    │
         │                    │
    ┌────▼────┐          ┌────▼────┐
    │   IGW   │          │   NAT   │
    └─────────┘          └─────────┘
         │                    │
         └────────┬───────────┘
                  │
            ┌─────▼─────┐
            │ Internet  │
            └──────────┘
```

### Components Explained

1. **VPC (Virtual Private Cloud)**
   - Your isolated network
   - Defined by CIDR block
   - Spans an entire AWS region

2. **Subnets**
   - Logical division of VPC IP address range
   - Must be in a single Availability Zone (AZ)
   - Can be public or private

3. **Internet Gateway (IGW)**
   - Allows resources in public subnets to access the internet
   - Provides public IP addresses
   - One per VPC

4. **NAT Gateway**
   - Allows private subnet resources to access internet
   - Prevents internet from initiating connections to private resources
   - Requires public subnet to host it

5. **Route Tables**
   - Control traffic routing within VPC
   - Each subnet must have a route table
   - Default route table created automatically

---

## Subnets and Routing

### Public vs Private Subnets

**Public Subnet:**
- Has a route to Internet Gateway (0.0.0.0/0 → IGW)
- Resources can have public IPs
- Direct internet access (bidirectional)

**Private Subnet:**
- No direct route to Internet Gateway
- Resources have only private IPs
- Can access internet via NAT Gateway (outbound only)

### Route Table Example

**Public Subnet Route Table:**
```
Destination        Target
10.0.0.0/16        local
0.0.0.0/0          igw-xxxxx (Internet Gateway)
```

**Private Subnet Route Table:**
```
Destination        Target
10.0.0.0/16        local
0.0.0.0/0          nat-xxxxx (NAT Gateway)
```

### How Routing Works

1. **Traffic originates from EC2 instance**
2. **EC2 checks its route table** (associated with subnet)
3. **Route table matches destination** to determine target
4. **Traffic is forwarded** to the target (IGW, NAT, VPC Peering, etc.)

**Example Flow:**
- EC2 in public subnet wants to reach `google.com`
- Route table matches `0.0.0.0/0` → sends to IGW
- IGW translates private IP to public IP
- Traffic goes to internet

---

## Internet Gateway and NAT

### Internet Gateway (IGW)

**Purpose:** Two-way communication between VPC and internet

**How it works:**
1. Attached to VPC
2. Provides public IP addresses
3. Performs Network Address Translation (NAT)
4. Routes traffic between VPC and internet

**Key Characteristics:**
- Highly available (managed by AWS)
- No bandwidth charges
- One per VPC
- Must be in public subnet route table

### NAT Gateway

**Purpose:** Allow private subnet resources to access internet (outbound only)

**How it works:**
1. Created in public subnet
2. Has its own Elastic IP
3. Private subnet route table points to NAT Gateway
4. NAT Gateway forwards traffic to IGW

**Traffic Flow:**
```
Private EC2 → Route Table → NAT Gateway → IGW → Internet
```

**Key Characteristics:**
- Managed service (highly available)
- Charges per hour and data transfer
- One NAT Gateway per AZ (for high availability)
- Internet cannot initiate connections to private resources

### NAT Instance vs NAT Gateway

**NAT Gateway (Recommended):**
- ✅ Managed by AWS
- ✅ Highly available
- ✅ Better performance
- ✅ No maintenance
- ❌ More expensive

**NAT Instance:**
- ✅ More control
- ✅ Cheaper for low traffic
- ❌ You manage it
- ❌ Single point of failure
- ❌ Manual scaling

---

## Security Groups and NACLs

### Security Groups (Stateful Firewall)

**What it is:** Virtual firewall at the instance level

**Key Characteristics:**
- **Stateful:** Return traffic is automatically allowed
- **Default deny:** All inbound denied, all outbound allowed by default
- **Instance-level:** Applied to EC2 instances, RDS, etc.
- **Allow rules only:** Cannot create deny rules

**Example:**
```
Inbound Rules:
- Type: HTTP, Port: 80, Source: 0.0.0.0/0
- Type: SSH, Port: 22, Source: 10.0.1.0/24

Outbound Rules:
- Type: All traffic, Port: All, Destination: 0.0.0.0/0
```

**How it works:**
1. Traffic arrives at instance
2. Security Group checks rules
3. If allowed, traffic proceeds
4. Return traffic automatically allowed (stateful)

### Network ACLs (NACLs) - Stateless Firewall

**What it is:** Virtual firewall at the subnet level

**Key Characteristics:**
- **Stateless:** Must explicitly allow return traffic
- **Subnet-level:** Applied to entire subnet
- **Allow and deny rules:** Can create both
- **Default allow:** All traffic allowed by default (can be changed)

**Example:**
```
Inbound Rules:
Rule #100: Allow HTTP (80) from 0.0.0.0/0
Rule #200: Allow SSH (22) from 10.0.1.0/24
Rule #32767: Deny all

Outbound Rules:
Rule #100: Allow all to 0.0.0.0/0
Rule #32767: Deny all
```

**Evaluation Order:**
1. Rules evaluated in order (lowest number first)
2. First match wins
3. Default rule (32767) is evaluated last

### Security Group vs NACL

| Feature | Security Group | NACL |
|---------|---------------|------|
| Level | Instance | Subnet |
| Stateful | Yes | No |
| Default | Deny inbound, Allow outbound | Allow all |
| Rules | Allow only | Allow and Deny |
| Use Case | Fine-grained control | Subnet-wide rules |

**Best Practice:** Use Security Groups for most cases, NACLs for compliance/extra layer

---

## Load Balancers

### Types of Load Balancers

#### 1. Application Load Balancer (ALB) - Layer 7

**Features:**
- HTTP/HTTPS traffic
- Content-based routing
- Host-based routing
- Path-based routing
- SSL termination
- WebSocket support

**Use Cases:**
- Web applications
- Microservices
- Container-based applications

**How it works:**
```
Internet → ALB → Target Groups → EC2 Instances
         (Layer 7)
```

#### 2. Network Load Balancer (NLB) - Layer 4

**Features:**
- TCP/UDP traffic
- Ultra-low latency
- High performance
- Static IP addresses
- Preserves source IP

**Use Cases:**
- High-performance applications
- Gaming
- Real-time applications

**How it works:**
```
Internet → NLB → Target Groups → EC2 Instances
         (Layer 4)
```

#### 3. Classic Load Balancer (Legacy)

**Features:**
- Layer 4 and Layer 7
- Basic load balancing
- Being phased out

### Load Balancer Architecture

```
┌─────────────┐
│   Internet  │
└──────┬──────┘
       │
       │
┌──────▼──────────────────────────────────┐
│         Application Load Balancer       │
│  ┌──────────┐  ┌──────────┐            │
│  │ Listener │  │ Listener │            │
│  │  HTTP:80 │  │ HTTPS:443│            │
│  └────┬─────┘  └────┬─────┘            │
│       │             │                  │
│       └──────┬──────┘                  │
│              │                         │
│       ┌──────▼──────┐                  │
│       │ Target Group│                  │
│       └──────┬──────┘                  │
└──────────────┼─────────────────────────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼───┐  ┌───▼───┐  ┌───▼───┐
│ EC2-1 │  │ EC2-2 │  │ EC2-3 │
└───────┘  └───────┘  └───────┘
```

### Load Balancer Components

1. **Listeners**
   - Check for connection requests
   - Configured with protocol and port
   - Routes to target groups based on rules

2. **Target Groups**
   - Group of targets (EC2 instances, IPs, Lambda)
   - Health checks configured here
   - Can have multiple target groups per ALB

3. **Health Checks**
   - Monitors target health
   - Unhealthy targets removed from rotation
   - Configurable interval, timeout, threshold

---

## Traffic Flow Scenarios

### Scenario 1: User Accesses Web Application

**Setup:**
- VPC: 10.0.0.0/16
- Public Subnet: 10.0.1.0/24 (ALB + Web Servers)
- Private Subnet: 10.0.2.0/24 (Database)

**Traffic Flow:**

```
1. User (Internet) 
   ↓
2. DNS resolves to ALB public IP
   ↓
3. ALB receives request (Layer 7)
   ↓
4. ALB checks Security Group (allow HTTP/HTTPS from 0.0.0.0/0)
   ↓
5. ALB routes to Target Group (healthy EC2 instances)
   ↓
6. Request reaches EC2 in public subnet
   ↓
7. EC2 Security Group allows inbound from ALB Security Group
   ↓
8. Web server processes request
   ↓
9. Web server queries database in private subnet
   ↓
10. Route table routes to private subnet (10.0.0.0/16 → local)
    ↓
11. Database Security Group allows inbound from Web Server Security Group
    ↓
12. Database responds
    ↓
13. Response flows back through same path
    ↓
14. ALB returns response to user
```

**Key Points:**
- ALB handles SSL termination
- Security Groups reference each other (best practice)
- Database never directly accessible from internet

### Scenario 2: Private EC2 Accesses Internet

**Setup:**
- Private Subnet: 10.0.2.0/24
- NAT Gateway in Public Subnet: 10.0.1.0/24

**Traffic Flow:**

```
1. EC2 in private subnet (10.0.2.10) wants to download package
   ↓
2. EC2 checks route table
   - 10.0.0.0/16 → local (stays in VPC)
   - 0.0.0.0/0 → nat-xxxxx (NAT Gateway)
   ↓
3. Traffic routed to NAT Gateway (10.0.1.50)
   ↓
4. NAT Gateway has route to IGW (public subnet route table)
   ↓
5. NAT Gateway translates source IP (10.0.2.10 → NAT's Elastic IP)
   ↓
6. IGW forwards to internet
   ↓
7. Response comes back to NAT Gateway's Elastic IP
   ↓
8. NAT Gateway remembers connection (stateful)
   ↓
9. NAT Gateway forwards to original EC2 (10.0.2.10)
   ↓
10. EC2 receives response
```

**Key Points:**
- NAT Gateway is stateful (remembers connections)
- Private EC2 never has public IP
- Internet cannot initiate connection to private EC2

### Scenario 3: VPC Peering

**Setup:**
- VPC-A: 10.0.0.0/16 (Production)
- VPC-B: 10.1.0.0/16 (Development)
- Peering connection established

**Traffic Flow:**

```
1. EC2 in VPC-A (10.0.1.10) wants to access RDS in VPC-B (10.1.2.20)
   ↓
2. Route table in VPC-A has entry:
   10.1.0.0/16 → pcx-xxxxx (Peering Connection)
   ↓
3. Traffic routed to peering connection
   ↓
4. Route table in VPC-B has entry:
   10.0.0.0/16 → pcx-xxxxx (Peering Connection)
   ↓
5. Traffic reaches RDS in VPC-B
   ↓
6. Security Groups allow traffic (must allow in both VPCs)
   ↓
7. Response flows back through peering connection
```

**Key Points:**
- CIDR blocks must not overlap
- Route tables must be configured in both VPCs
- Security Groups must allow traffic in both directions
- No transitive routing (must have direct peering)

---

## Common Interview Scenarios

### Scenario 1: "How would you design a highly available web application?"

**Answer:**

```
Architecture:
- Multi-AZ deployment (at least 2 AZs)
- Public subnets in each AZ (for ALB)
- Private subnets in each AZ (for EC2 instances)
- Application Load Balancer (ALB) across multiple AZs
- Auto Scaling Group for EC2 instances
- RDS Multi-AZ for database
- NAT Gateway in each AZ (for high availability)

Traffic Flow:
Internet → Route 53 → ALB (Multi-AZ) → EC2 Instances (Auto Scaling)
                                              ↓
                                         RDS Multi-AZ

Key Points:
- ALB automatically distributes across AZs
- If one AZ fails, traffic routes to other AZs
- NAT Gateway in each AZ prevents single point of failure
- Auto Scaling ensures capacity
```

### Scenario 2: "A user can't access your application. How do you troubleshoot?"

**Troubleshooting Steps:**

1. **Check DNS Resolution**
   - Is Route 53 resolving correctly?
   - Is the domain pointing to correct ALB?

2. **Check Load Balancer**
   - Are targets healthy?
   - Are Security Groups allowing traffic?
   - Check ALB access logs

3. **Check EC2 Instances**
   - Are instances running?
   - Are Security Groups allowing traffic from ALB?
   - Check application logs
   - Check instance health checks

4. **Check Network**
   - Route tables correct?
   - NACLs allowing traffic?
   - VPC endpoints working (if using private services)?

5. **Check Application**
   - Is application responding?
   - Database connectivity?
   - Application errors in logs?

### Scenario 3: "How do you secure a three-tier application?"

**Answer:**

```
Architecture:
┌─────────────────────────────────────────┐
│  Public Subnet (10.0.1.0/24)            │
│  - Application Load Balancer            │
│  - NAT Gateway                          │
└─────────────────────────────────────────┘
         │
         │ (Security Group: ALB → Web)
         │
┌─────────────────────────────────────────┐
│  Private Subnet - Web (10.0.2.0/24)     │
│  - EC2 Web Servers                      │
│  - Auto Scaling Group                   │
└─────────────────────────────────────────┘
         │
         │ (Security Group: Web → App)
         │
┌─────────────────────────────────────────┐
│  Private Subnet - App (10.0.3.0/24)     │
│  - EC2 Application Servers               │
│  - Auto Scaling Group                   │
└─────────────────────────────────────────┘
         │
         │ (Security Group: App → DB)
         │
┌─────────────────────────────────────────┐
│  Private Subnet - DB (10.0.4.0/24)      │
│  - RDS Database                         │
│  - Multi-AZ                              │
└─────────────────────────────────────────┘

Security Measures:
1. Web tier: Only accepts traffic from ALB
2. App tier: Only accepts traffic from Web tier
3. DB tier: Only accepts traffic from App tier
4. No direct internet access to any tier except ALB
5. NACLs provide additional subnet-level security
6. Database in separate subnet with restricted access
7. Use VPC endpoints for AWS services (no internet needed)
```

### Scenario 4: "Explain the difference between Security Groups and NACLs with an example"

**Answer:**

**Security Group (Stateful):**
```
Inbound Rule: Allow HTTP (80) from 0.0.0.0/0
Outbound Rule: (Default: Allow all)

When user sends HTTP request:
1. Inbound rule allows request → ✅ Allowed
2. Response automatically allowed (stateful) → ✅ No outbound rule needed

When user sends HTTPS request:
1. No inbound rule for 443 → ❌ Denied
```

**NACL (Stateless):**
```
Inbound Rules:
- Rule 100: Allow HTTP (80) from 0.0.0.0/0
- Rule 200: Allow HTTPS (443) from 0.0.0.0/0
- Rule 32767: Deny all

Outbound Rules:
- Rule 100: Allow ephemeral ports (1024-65535) to 0.0.0.0/0
- Rule 32767: Deny all

When user sends HTTP request:
1. Inbound rule 100 allows request → ✅ Allowed
2. Response needs outbound rule → ✅ Rule 100 allows ephemeral ports
3. If outbound rule missing → ❌ Response blocked (stateless)
```

**Key Difference:**
- Security Group: Allow inbound HTTP, outbound automatically allowed
- NACL: Must explicitly allow both inbound AND outbound

### Scenario 5: "How does traffic flow from internet to a private EC2 instance?"

**Answer:**

```
This is a TRICK question! Private EC2 instances cannot be directly accessed from internet.

Correct Architecture:
Internet → ALB (Public) → EC2 (Private)

Traffic Flow:
1. User accesses ALB public IP/DNS
2. ALB is in public subnet with route to IGW
3. ALB receives request (Security Group allows from internet)
4. ALB routes to EC2 in private subnet
5. EC2 Security Group allows traffic from ALB Security Group
6. EC2 processes request
7. Response goes back through ALB to user

Key Points:
- Private EC2 never directly accessible from internet
- ALB acts as entry point
- Security Groups must allow ALB → EC2 traffic
- EC2 can access internet via NAT Gateway (outbound only)
```

---

## Best Practices

1. **Use multiple AZs** for high availability
2. **Separate public and private subnets**
3. **Use Security Groups** for instance-level security
4. **Use NACLs** for compliance/extra layer (optional)
5. **Reference Security Groups** instead of IPs (more flexible)
6. **Use NAT Gateway** instead of NAT Instance (unless cost is concern)
7. **Use VPC endpoints** for AWS services (no internet needed)
8. **Enable VPC Flow Logs** for monitoring
9. **Use least privilege** in Security Groups
10. **Plan CIDR blocks** carefully (avoid overlaps)

---

## Summary

**Key Takeaways:**
- VPC is your isolated network in AWS
- Subnets divide VPC into smaller networks (one per AZ)
- IGW provides internet access to public subnets
- NAT Gateway provides outbound internet access to private subnets
- Security Groups are stateful, instance-level firewalls
- NACLs are stateless, subnet-level firewalls
- Load Balancers distribute traffic and provide entry points
- Route tables control traffic routing within VPC

**Remember:**
- Public subnet = route to IGW
- Private subnet = route to NAT Gateway (or no internet)
- Security Groups are evaluated first, then NACLs
- Always design for high availability (multiple AZs)

