# Azure Networking: Complete Guide for DevOps Engineers

## Table of Contents
1. [Core Concepts](#core-concepts)
2. [Virtual Network (VNet) Architecture](#virtual-network-vnet-architecture)
3. [Subnets and Routing](#subnets-and-routing)
4. [Network Security Groups (NSGs)](#network-security-groups-nsgs)
5. [Load Balancers](#load-balancers)
6. [Application Gateway](#application-gateway)
7. [Traffic Flow Scenarios](#traffic-flow-scenarios)
8. [Azure Kubernetes Service (AKS) Networking](#azure-kubernetes-service-aks-networking)
9. [Common Interview Scenarios](#common-interview-scenarios)

---

## Core Concepts

### What is a Virtual Network (VNet)?

A **Virtual Network (VNet)** is your isolated network in Azure. Think of it as your private network in the cloud, similar to AWS VPC.

**Key Points:**
- Each VNet has an address space (CIDR block)
- You can create multiple VNets in different regions
- VNets are region-specific
- By default, VNets cannot communicate with each other (unless you set up peering/VPN)

### IP Addressing in Azure

- **Private IP ranges:**
  - 10.0.0.0/8 (10.0.0.0 to 10.255.255.255)
  - 172.16.0.0/12 (172.16.0.0 to 172.31.255.255)
  - 192.168.0.0/16 (192.168.0.0 to 192.168.255.255)
- **Public IP:** Assigned by Azure from Azure's pool (not your VNet address space)

### Azure Networking Components

- **Virtual Network (VNet):** Isolated network
- **Subnets:** Logical divisions of VNet
- **Network Security Groups (NSGs):** Firewall rules
- **Route Tables:** Control traffic routing
- **Load Balancer:** Distribute traffic
- **Application Gateway:** Layer 7 load balancer
- **Virtual Network Gateway:** VPN/ExpressRoute connectivity

---

## Virtual Network (VNet) Architecture

### Basic VNet Components

```
┌─────────────────────────────────────────────────┐
│              Virtual Network (VNet)             │
│              (10.0.0.0/16)                      │
│                                                  │
│  ┌──────────────┐      ┌──────────────┐        │
│  │ Public Subnet│      │ Private Subnet│       │
│  │ 10.0.1.0/24  │      │ 10.0.2.0/24  │        │
│  │              │      │              │        │
│  │ VM (Public IP)│     │ VM (Private IP)│      │
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
    │Internet │          │   NAT   │
    │Gateway  │          │ Gateway │
    └─────────┘          └─────────┘
```

### Components Explained

1. **Virtual Network (VNet)**
   - Your isolated network
   - Defined by address space (CIDR)
   - Spans an entire Azure region

2. **Subnets**
   - Logical division of VNet address space
   - Must be within a single VNet
   - Can span multiple Availability Zones (optional)

3. **Public IP Address**
   - Assigned to resources that need internet access
   - Can be static or dynamic
   - Not part of VNet address space

4. **Network Security Groups (NSGs)**
   - Firewall rules for subnets/network interfaces
   - Stateful (return traffic automatically allowed)
   - Default deny inbound, allow outbound

---

## Subnets and Routing

### Public vs Private Subnets

**Public Subnet:**
- Resources can have public IPs
- Direct internet access (bidirectional)
- Typically hosts load balancers, NAT gateways

**Private Subnet:**
- Resources have only private IPs
- No direct internet access
- Can access internet via NAT Gateway (outbound only)

### Route Tables

**System Routes (Default):**
Azure creates default routes automatically:
- Local VNet traffic: Stays within VNet
- Internet traffic: Routes to internet
- Virtual network peering: Routes to peered VNets

**User-Defined Routes (UDRs):**
You can create custom routes:
- Route to Network Virtual Appliance (NVA)
- Route to Virtual Network Gateway
- Route to specific next hop

**Route Table Example:**
```
Name                Address Prefix    Next Hop Type    Next Hop IP
Local              10.0.0.0/16       VNetLocal        -
Internet           0.0.0.0/0         Internet         -
Peering            10.1.0.0/16       VNetPeering      -
```

### How Routing Works

1. **Traffic originates from VM**
2. **Azure checks route table** (associated with subnet)
3. **Route table matches destination** to determine next hop
4. **Traffic is forwarded** to the next hop

**Example Flow:**
- VM in public subnet wants to reach `google.com`
- Route table matches `0.0.0.0/0` → routes to Internet
- Traffic goes to internet via public IP

---

## Network Security Groups (NSGs)

### What are NSGs?

**Network Security Groups (NSGs)** are firewall rules that control traffic to/from Azure resources. They are stateful firewalls.

**Key Characteristics:**
- **Stateful:** Return traffic is automatically allowed
- **Default deny inbound:** All inbound denied by default
- **Default allow outbound:** All outbound allowed by default
- **Can be applied to:** Subnet or Network Interface (NIC)

### NSG Rules

**Priority:** Rules are evaluated in priority order (lower number = higher priority)

**Rule Components:**
- **Name:** Rule identifier
- **Priority:** 100-4096 (lower = higher priority)
- **Source/Destination:** IP address, service tag, or application security group
- **Protocol:** TCP, UDP, or Any
- **Port:** Single port or range
- **Action:** Allow or Deny

**Example NSG Rules:**
```
Priority  Name           Source      Destination  Protocol  Port  Action
1000      AllowHTTP      Internet    Any          TCP       80    Allow
1100      AllowHTTPS     Internet    Any          TCP       443   Allow
1200      AllowSSH       MyIP        Any          TCP       22    Allow
4096      DenyAllInbound Any         Any          Any       Any   Deny
```

### NSG Application Security Groups (ASGs)

**Application Security Groups** allow you to group VMs and apply NSG rules to the group.

**Example:**
```
ASG: WebServers
- Contains: web-vm-1, web-vm-2

NSG Rule:
- Source: Internet
- Destination: WebServers (ASG)
- Port: 80
- Action: Allow
```

**Benefits:**
- Dynamic grouping (VM automatically inherits rules when added to ASG)
- Easier management
- More flexible than IP-based rules

### NSG Flow Logs

**NSG Flow Logs** capture information about IP traffic flowing through NSGs.

**Use Cases:**
- Network monitoring
- Security analysis
- Troubleshooting
- Compliance

**Information Captured:**
- Source/destination IPs
- Ports
- Protocol
- Action (allowed/denied)
- Bytes sent/received

---

## Load Balancers

### Types of Load Balancers

#### 1. Azure Load Balancer (Layer 4)

**Features:**
- TCP/UDP load balancing
- Internal or public
- Health probes
- Port forwarding
- High availability

**Use Cases:**
- Non-HTTP traffic
- High-performance scenarios
- Port-based routing

**How it works:**
```
Internet → Public Load Balancer → Backend Pool → VMs
```

#### 2. Application Gateway (Layer 7)

**Features:**
- HTTP/HTTPS load balancing
- URL-based routing
- Host-based routing
- SSL termination
- Web Application Firewall (WAF)
- Cookie-based session affinity

**Use Cases:**
- Web applications
- Microservices
- SSL termination
- WAF protection

**How it works:**
```
Internet → Application Gateway → Backend Pool → VMs
         (Layer 7 - HTTP/HTTPS)
```

### Load Balancer Architecture

```
┌─────────────┐
│   Internet  │
└──────┬──────┘
       │
       │
┌──────▼──────────────────────────────────┐
│      Azure Load Balancer                │
│  ┌──────────┐  ┌──────────┐            │
│  │ Frontend │  │ Frontend │            │
│  │  Public  │  │ Internal │            │
│  └────┬─────┘  └────┬─────┘            │
│       │             │                  │
│       └──────┬──────┘                  │
│              │                         │
│       ┌──────▼──────┐                  │
│       │ Load Balance│                  │
│       │   Rules     │                  │
│       └──────┬──────┘                  │
│              │                         │
│       ┌──────▼──────┐                  │
│       │ Health      │                  │
│       │   Probes    │                  │
│       └──────┬──────┘                  │
│              │                         │
│       ┌──────▼──────┐                  │
│       │ Backend Pool│                  │
│       └──────┬──────┘                  │
└──────────────┼─────────────────────────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼───┐  ┌───▼───┐  ┌───▼───┐
│  VM-1 │  │  VM-2 │  │  VM-3 │
└───────┘  └───────┘  └───────┘
```

### Load Balancer Components

1. **Frontend IP Configuration**
   - Public IP (for internet-facing)
   - Private IP (for internal)
   - Defines entry point

2. **Backend Pool**
   - Group of VMs or VM scale sets
   - Health probes determine availability
   - Load balancing algorithm (5-tuple hash)

3. **Load Balancing Rules**
   - Maps frontend to backend pool
   - Defines port and protocol
   - Health probe association

4. **Health Probes**
   - Monitors backend health
   - HTTP, TCP, or HTTPS
   - Unhealthy instances removed from rotation

---

## Application Gateway

### What is Application Gateway?

**Application Gateway** is Azure's Layer 7 (HTTP/HTTPS) load balancer with advanced features.

### Key Features

1. **URL Path-Based Routing**
   - Route based on URL path
   - Example: `/api/*` → API backend, `/web/*` → Web backend

2. **Host-Based Routing**
   - Route based on host header
   - Example: `api.example.com` → API backend, `web.example.com` → Web backend

3. **SSL Termination**
   - Decrypts SSL at gateway
   - Reduces load on backend
   - Supports multiple certificates

4. **Web Application Firewall (WAF)**
   - Protects against common web vulnerabilities
   - OWASP Top 10 protection
   - Custom rules

5. **Session Affinity**
   - Cookie-based session affinity
   - Ensures user stays on same backend

### Application Gateway Architecture

```
┌─────────────┐
│   Internet  │
└──────┬──────┘
       │
       │
┌──────▼──────────────────────────────────┐
│      Application Gateway                │
│  ┌──────────────────────────────────┐  │
│  │         WAF (Optional)           │  │
│  └──────────────┬───────────────────┘  │
│                 │                       │
│  ┌──────────────▼───────────────────┐  │
│  │      HTTP Listener               │  │
│  │  - Host: myapp.com               │  │
│  │  - Port: 443 (HTTPS)             │  │
│  └──────────────┬───────────────────┘  │
│                 │                       │
│  ┌──────────────▼───────────────────┐  │
│  │      Routing Rules               │  │
│  │  - Path: /api/* → API Backend    │  │
│  │  - Path: /* → Web Backend        │  │
│  └──────────────┬───────────────────┘  │
│                 │                       │
│  ┌──────────────▼───────────────────┐  │
│  │      Backend Pools               │  │
│  │  - API Pool: api-vm-1, api-vm-2  │  │
│  │  - Web Pool: web-vm-1, web-vm-2  │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
         │                    │
         │                    │
    ┌────▼────┐          ┌────▼────┐
    │ API VMs │          │ Web VMs │
    └─────────┘          └─────────┘
```

### Application Gateway Traffic Flow

```
1. User accesses myapp.com/api/users
   ↓
2. DNS resolves to Application Gateway public IP
   ↓
3. Application Gateway receives request
   ↓
4. WAF checks request (if enabled)
   ↓
5. HTTP Listener matches host (myapp.com) and port (443)
   ↓
6. Routing rule matches path (/api/*)
   ↓
7. Routes to API backend pool
   ↓
8. Application Gateway selects healthy backend (api-vm-1)
   ↓
9. Request forwarded to api-vm-1
   ↓
10. Response flows back through Application Gateway
    ↓
11. Application Gateway returns response to user
```

---

## Traffic Flow Scenarios

### Scenario 1: User Accesses Web Application

**Setup:**
- VNet: 10.0.0.0/16
- Public Subnet: 10.0.1.0/24 (Application Gateway)
- Private Subnet: 10.0.2.0/24 (Web VMs)
- Private Subnet: 10.0.3.0/24 (Database)

**Traffic Flow:**

```
1. User (Internet) 
   ↓
2. DNS resolves to Application Gateway public IP
   ↓
3. Application Gateway receives request (Layer 7)
   ↓
4. NSG on Application Gateway subnet allows HTTP/HTTPS from internet
   ↓
5. Application Gateway checks routing rules
   ↓
6. Routes to backend pool (web VMs)
   ↓
7. Application Gateway forwards to web VM (10.0.2.10)
   ↓
8. VNet routing delivers to private subnet (10.0.2.0/24)
   ↓
9. NSG on web VM allows inbound from Application Gateway
   ↓
10. Web VM processes request
    ↓
11. Web VM queries database (10.0.3.20)
    ↓
12. VNet routing delivers to database subnet (10.0.3.0/24)
    ↓
13. NSG on database allows inbound from web VM subnet
    ↓
14. Database responds
    ↓
15. Response flows back through same path
    ↓
16. Application Gateway returns response to user
```

**Key Points:**
- Application Gateway handles SSL termination
- NSGs control access at subnet/VM level
- Database never directly accessible from internet

### Scenario 2: Private VM Accesses Internet

**Setup:**
- Private Subnet: 10.0.2.0/24
- NAT Gateway in Public Subnet: 10.0.1.0/24

**Traffic Flow:**

```
1. VM in private subnet (10.0.2.10) wants to download package
   ↓
2. VM checks route table
   - 10.0.0.0/16 → local (stays in VNet)
   - 0.0.0.0/0 → NAT Gateway (10.0.1.50)
   ↓
3. Traffic routed to NAT Gateway
   ↓
4. NAT Gateway translates source IP (10.0.2.10 → NAT's public IP)
   ↓
5. NAT Gateway forwards to internet
   ↓
6. Response comes back to NAT Gateway's public IP
   ↓
7. NAT Gateway remembers connection (stateful)
   ↓
8. NAT Gateway forwards to original VM (10.0.2.10)
   ↓
9. VM receives response
```

**Key Points:**
- NAT Gateway is stateful (remembers connections)
- Private VM never has public IP
- Internet cannot initiate connection to private VM

### Scenario 3: VNet Peering

**Setup:**
- VNet-A: 10.0.0.0/16 (Production)
- VNet-B: 10.1.0.0/16 (Development)
- Peering connection established

**Traffic Flow:**

```
1. VM in VNet-A (10.0.1.10) wants to access VM in VNet-B (10.1.2.20)
   ↓
2. Route table in VNet-A has entry:
   10.1.0.0/16 → VNet Peering
   ↓
3. Traffic routed to peering connection
   ↓
4. Route table in VNet-B has entry:
   10.0.0.0/16 → VNet Peering
   ↓
5. Traffic reaches VM in VNet-B
   ↓
6. NSGs allow traffic (must allow in both VNets)
   ↓
7. Response flows back through peering connection
```

**Key Points:**
- Address spaces must not overlap
- Route tables must be configured (usually automatic)
- NSGs must allow traffic in both directions
- No transitive routing (must have direct peering)

---

## Azure Kubernetes Service (AKS) Networking

### AKS Networking Models

#### 1. kubenet (Basic Networking)

**How it works:**
- Pods get IPs from a separate CIDR (not VNet)
- Nodes get VNet IPs
- Uses overlay network (bridge)
- NAT for pod internet access

**Characteristics:**
- Simpler setup
- Limited to 400 nodes per cluster
- Pods not directly accessible from VNet
- Requires user-defined routes (UDRs)

#### 2. Azure CNI (Advanced Networking)

**How it works:**
- Pods get IPs directly from VNet subnets
- No overlay network
- Direct VNet integration
- Pods appear as VNet resources

**Characteristics:**
- Better performance (no overlay)
- Pods directly accessible from VNet
- Must plan IP addresses carefully
- More complex setup

### AKS Network Architecture (Azure CNI)

```
┌─────────────────────────────────────────────────┐
│              Virtual Network (VNet)              │
│              (10.0.0.0/16)                      │
│                                                  │
│  ┌──────────────────────────────────────────┐  │
│  │     AKS Cluster Subnet                   │  │
│  │     10.0.1.0/24                         │  │
│  │                                          │  │
│  │  ┌────────────┐      ┌────────────┐    │  │
│  │  │ Node Pool  │      │ Node Pool  │    │  │
│  │  │            │      │            │    │  │
│  │  │ Pod:       │      │ Pod:       │    │  │
│  │  │ 10.0.1.50  │      │ 10.0.1.60  │    │  │
│  │  │            │      │            │    │  │
│  │  │ Pod:       │      │ Pod:       │    │  │
│  │  │ 10.0.1.51  │      │ 10.0.1.61  │    │  │
│  │  └────────────┘      └────────────┘    │  │
│  └──────────────────────────────────────────┘  │
│                                                  │
│  ┌──────────────────────────────────────────┐  │
│  │     Application Gateway Subnet            │  │
│  │     10.0.2.0/24                         │  │
│  │                                          │  │
│  │  Application Gateway                     │  │
│  └──────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

### AKS Service Types

#### 1. LoadBalancer Service

**How it works:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

**Traffic Flow:**
```
1. Service created with type: LoadBalancer
   ↓
2. Azure creates Azure Load Balancer
   ↓
3. Load Balancer gets public IP
   ↓
4. Load Balancer routes to node IPs (NodePort)
   ↓
5. kube-proxy routes to Service ClusterIP
   ↓
6. Service routes to pod IPs
   ↓
7. Traffic: Internet → Load Balancer → Nodes → Pods
```

#### 2. Ingress with Application Gateway

**How it works:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**Traffic Flow:**
```
1. Ingress resource created
   ↓
2. Application Gateway Ingress Controller (AGIC) watches API server
   ↓
3. AGIC configures Application Gateway
   ↓
4. Application Gateway routes to pod IPs (if Azure CNI)
   ↓
5. Traffic: Internet → Application Gateway → Pod IPs (direct)
```

### AKS Pod-to-Pod Communication

**Azure CNI Mode:**
```
Pod A (10.0.1.50) → VNet Routing → Pod B (10.0.1.60)
- Direct VNet IP communication
- No overlay network
- VNet handles routing
```

**kubenet Mode:**
```
Pod A (172.17.1.5) → Bridge → Node → VNet → Node → Bridge → Pod B (172.17.2.5)
- Overlay network
- NAT for internet access
- Requires UDRs
```

---

## Common Interview Scenarios

### Scenario 1: "How would you design a highly available web application in Azure?"

**Answer:**

```
Architecture:
- Multi-region deployment (primary + secondary)
- Availability Zones in each region
- Application Gateway (multi-instance, zone-redundant)
- Virtual Machine Scale Sets for VMs
- Azure SQL Database (geo-replication)
- Traffic Manager for global load balancing

Traffic Flow:
Internet → Traffic Manager → Application Gateway (Multi-AZ) → VM Scale Sets
                                                                    ↓
                                                           Azure SQL (Multi-AZ)

Key Components:
- Application Gateway: Layer 7 load balancing, SSL termination
- VM Scale Sets: Auto-scaling, high availability
- Availability Zones: Protection against datacenter failures
- Traffic Manager: Global load balancing, failover
- Azure SQL: Multi-AZ, geo-replication

High Availability Features:
- Application Gateway: Zone-redundant, automatic failover
- VM Scale Sets: Distribute across zones
- Azure SQL: Always On, automatic failover
- Traffic Manager: Health checks, automatic failover
```

### Scenario 2: "Explain the difference between NSGs and Application Security Groups"

**Answer:**

**Network Security Groups (NSGs):**
- Firewall rules for subnets/network interfaces
- Can use IP addresses, service tags, or ASGs
- Applied to subnet or NIC
- Stateful (return traffic automatically allowed)

**Application Security Groups (ASGs):**
- Logical grouping of VMs
- Used in NSG rules (as source/destination)
- Dynamic membership (VM automatically inherits rules)
- Makes NSG rules more manageable

**Example:**
```
ASG: WebServers
- Contains: web-vm-1, web-vm-2, web-vm-3

NSG Rule:
- Source: Internet
- Destination: WebServers (ASG)
- Port: 80
- Action: Allow

When you add web-vm-4 to WebServers ASG:
- web-vm-4 automatically gets the NSG rules
- No need to update NSG rules
```

**Benefits of ASGs:**
- Easier management (group-based rules)
- Dynamic (add/remove VMs without changing rules)
- More flexible than IP-based rules
- Better for auto-scaling scenarios

### Scenario 3: "How does Application Gateway differ from Azure Load Balancer?"

**Answer:**

| Feature | Application Gateway | Azure Load Balancer |
|---------|---------------------|---------------------|
| Layer | Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP) |
| Routing | URL path, host header | Port-based |
| SSL | Built-in termination | Manual setup |
| WAF | Built-in (optional) | Not available |
| Session Affinity | Cookie-based | Source IP-based |
| Use Case | Web applications | Any TCP/UDP service |
| Cost | Higher | Lower |

**When to use Application Gateway:**
- Web applications (HTTP/HTTPS)
- Need URL-based routing
- Need SSL termination
- Need WAF protection
- Need cookie-based session affinity

**When to use Azure Load Balancer:**
- Non-HTTP traffic (TCP/UDP)
- High-performance scenarios
- Port-based routing
- Lower cost requirement

### Scenario 4: "How does AKS networking work with Azure CNI?"

**Answer:**

**Azure CNI Mode:**
- Pods get IPs directly from VNet subnets
- No overlay network
- Direct VNet integration
- Pods appear as VNet resources

**How it works:**
```
1. Pod is scheduled to node
   ↓
2. Azure CNI plugin allocates IP from VNet subnet
   ↓
3. IP is assigned from subnet's address space
   ↓
4. Pod gets real VNet IP (e.g., 10.0.1.50)
   ↓
5. No encapsulation needed (direct VNet routing)
   ↓
6. Pod can communicate directly with VNet resources
```

**Pod-to-Pod Communication:**
```
Same Node:
- Pod A and Pod B on same node
- Direct communication via node's network

Different Nodes:
- Pod A (10.0.1.50) → VNet Routing → Pod B (10.0.1.60)
- VNet handles routing between nodes
- No overlay network overhead
```

**Benefits:**
- Better performance (no overlay)
- Direct VNet integration
- Pods accessible from VNet
- Can use NSGs on pod IPs

**Limitations:**
- Must plan IP addresses carefully
- Limited by subnet size
- More complex setup

### Scenario 5: "Troubleshoot: User can't access application"

**Troubleshooting Steps:**

1. **Check DNS Resolution**
   - Is DNS resolving correctly?
   - Is domain pointing to correct Application Gateway?

2. **Check Application Gateway**
   - Is Application Gateway running?
   - Are backend pools healthy?
   - Check Application Gateway logs
   - Check WAF logs (if enabled)

3. **Check NSGs**
   - Are NSGs allowing traffic?
   - Check NSG flow logs
   - Verify rules are correct

4. **Check VMs**
   - Are VMs running?
   - Are VMs healthy?
   - Check VM logs
   - Check application logs

5. **Check Network**
   - Are route tables correct?
   - Is VNet peering working?
   - Check VNet connectivity

6. **Check Application**
   - Is application responding?
   - Database connectivity?
   - Application errors in logs?

### Scenario 6: "How do you secure a three-tier application in Azure?"

**Answer:**

```
Architecture:
┌─────────────────────────────────────────┐
│  Public Subnet (10.0.1.0/24)            │
│  - Application Gateway                  │
│  - NAT Gateway                          │
└─────────────────────────────────────────┘
         │
         │ (NSG: Allow from Internet)
         │
┌─────────────────────────────────────────┐
│  Private Subnet - Web (10.0.2.0/24)     │
│  - Web VMs                              │
│  - VM Scale Set                         │
└─────────────────────────────────────────┘
         │
         │ (NSG: Allow from App Gateway)
         │
┌─────────────────────────────────────────┐
│  Private Subnet - App (10.0.3.0/24)     │
│  - Application VMs                        │
│  - VM Scale Set                          │
└─────────────────────────────────────────┘
         │
         │ (NSG: Allow from Web Subnet)
         │
┌─────────────────────────────────────────┐
│  Private Subnet - DB (10.0.4.0/24)      │
│  - Azure SQL Database                    │
│  - Private Endpoint                      │
└─────────────────────────────────────────┘

Security Measures:
1. Web tier: Only accepts traffic from Application Gateway
2. App tier: Only accepts traffic from Web tier
3. DB tier: Only accepts traffic from App tier
4. No direct internet access to any tier except Application Gateway
5. NSGs provide subnet-level security
6. Database uses Private Endpoint (no public access)
7. Use Managed Identity for authentication
8. Enable WAF on Application Gateway
```

---

## Best Practices

1. **Use Availability Zones** for high availability
2. **Separate public and private subnets**
3. **Use NSGs** for network security
4. **Use Application Security Groups** for easier management
5. **Use Application Gateway** for web applications
6. **Use Azure CNI** for AKS (better performance)
7. **Plan IP addresses** carefully (especially for AKS)
8. **Enable NSG Flow Logs** for monitoring
9. **Use Private Endpoints** for PaaS services
10. **Use Managed Identity** for authentication

---

## Summary

**Key Takeaways:**
- VNet is your isolated network in Azure
- Subnets divide VNet into smaller networks
- NSGs are stateful firewalls (subnet/NIC level)
- Application Gateway is Layer 7 load balancer
- Azure Load Balancer is Layer 4 load balancer
- AKS can use Azure CNI (direct VNet integration)
- Application Security Groups simplify NSG management

**Remember:**
- NSGs are stateful (return traffic automatically allowed)
- Application Gateway for HTTP/HTTPS, Load Balancer for TCP/UDP
- Azure CNI gives pods real VNet IPs (no overlay)
- Plan IP addresses carefully (especially for AKS)
- Use Availability Zones for high availability

