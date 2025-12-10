# AWS & Kubernetes Networking: Complete Guide for DevOps Engineers

## Table of Contents
1. [AWS Networking Fundamentals](#aws-networking-fundamentals)
2. [Kubernetes Networking Basics](#kubernetes-networking-basics)
3. [AWS EKS Integration](#aws-eks-integration)
4. [End-to-End Traffic Flow Scenarios](#end-to-end-traffic-flow-scenarios)
5. [Common Interview Scenarios](#common-interview-scenarios)
6. [Troubleshooting Guide](#troubleshooting-guide)

---

## AWS Networking Fundamentals

### 1. VPC (Virtual Private Cloud)

**What is it?**
Think of a VPC as your own private section of AWS cloud. It's like renting a piece of land where you can build your own network infrastructure.

**Key Concepts:**
- **CIDR Block**: Defines the IP address range (e.g., 10.0.0.0/16 means 65,536 IP addresses from 10.0.0.0 to 10.0.255.255)
- **Region-specific**: Each VPC exists in one AWS region
- **Isolated by default**: Resources in one VPC cannot communicate with another VPC unless explicitly configured

**Example:**
```
VPC: 10.0.0.0/16
- This gives you all IPs from 10.0.0.0 to 10.0.255.255
- /16 means first 16 bits are fixed (10.0), last 16 bits are variable
```

### 2. Subnets

**What is it?**
Subnets are smaller networks within your VPC. Think of them as rooms in your house (VPC).

**Types:**
- **Public Subnet**: Has a route to the Internet Gateway (IGW). Resources here can directly access the internet.
- **Private Subnet**: No direct internet access. Resources here are more secure.

**Key Points:**
- Subnets are Availability Zone (AZ) specific
- Each subnet has its own CIDR block (subset of VPC CIDR)
- Route table determines if subnet is public or private

**Example Setup:**
```
VPC: 10.0.0.0/16

Public Subnets:
- 10.0.1.0/24 (us-east-1a) - 256 IPs
- 10.0.2.0/24 (us-east-1b) - 256 IPs

Private Subnets:
- 10.0.10.0/24 (us-east-1a) - 256 IPs
- 10.0.11.0/24 (us-east-1b) - 256 IPs
```

### 3. Internet Gateway (IGW)

**What is it?**
The door that connects your VPC to the internet. It's a one-to-one mapping with your VPC.

**How it works:**
- Attached to VPC
- Provides public IP addresses (via Elastic IPs)
- Enables bidirectional internet access
- Only one IGW per VPC

**Traffic Flow:**
```
Internet → IGW → Public Subnet → Your Resource
Your Resource → Public Subnet → IGW → Internet
```

### 4. NAT Gateway / NAT Instance

**What is it?**
Allows resources in private subnets to access the internet (outbound only) while keeping them hidden from the internet.

**Key Differences:**
- **NAT Gateway**: Managed by AWS, highly available, scales automatically
- **NAT Instance**: EC2 instance you manage, more control but more work

**Why use it?**
- Private subnets need to download updates, access APIs, etc.
- But you don't want them directly accessible from the internet

**Traffic Flow:**
```
Private Subnet Resource → NAT Gateway → IGW → Internet
Internet → (Cannot reach private subnet directly)
```

### 5. Route Tables

**What is it?**
Like a GPS system for your network. Tells traffic where to go.

**Key Rules:**
- Each subnet must have a route table (explicit or default)
- Routes are evaluated in order (most specific first)
- Default route: 0.0.0.0/0 means "everything else"

**Public Subnet Route Table:**
```
Destination        Target
10.0.0.0/16        local (VPC internal)
0.0.0.0/0          igw-xxxxx (Internet Gateway)
```

**Private Subnet Route Table:**
```
Destination        Target
10.0.0.0/16        local (VPC internal)
0.0.0.0/0          nat-xxxxx (NAT Gateway)
```

### 6. Security Groups

**What is it?**
A virtual firewall that controls inbound and outbound traffic at the instance/ENI level.

**Key Characteristics:**
- **Stateful**: If you allow inbound, outbound is automatically allowed
- **Default deny**: Everything is blocked unless explicitly allowed
- **Can attach multiple**: One resource can have multiple security groups
- **Region-specific**: Security groups are tied to a region

**Example:**
```
Security Group: web-servers
Inbound Rules:
- Port 80 (HTTP) from 0.0.0.0/0 (anywhere)
- Port 443 (HTTPS) from 0.0.0.0/0
- Port 22 (SSH) from 10.0.0.0/16 (only from VPC)

Outbound Rules:
- All traffic to 0.0.0.0/0 (default)
```

### 7. Network ACLs (NACLs)

**What is it?**
Another layer of security at the subnet level (not instance level).

**Key Differences from Security Groups:**
- **Stateless**: Must allow both inbound and outbound
- **Subnet-level**: Applies to entire subnet
- **Order matters**: Rules are evaluated in order (lowest number first)
- **Default allow**: Everything is allowed unless explicitly denied

**Example:**
```
NACL Rules (evaluated in order):
Rule #100: Allow inbound HTTP (80) from 0.0.0.0/0
Rule #200: Allow inbound HTTPS (443) from 0.0.0.0/0
Rule #300: Deny all inbound
```

### 8. VPC Peering

**What is it?**
Direct connection between two VPCs, allowing them to communicate as if they're on the same network.

**Key Points:**
- **Non-transitive**: If VPC A peers with B, and B peers with C, A cannot talk to C
- **Same or different accounts/regions**: Can peer across accounts and regions
- **CIDR blocks must not overlap**: VPCs cannot have overlapping IP ranges

**Use Case:**
```
Production VPC (10.0.0.0/16) ←→ Peering Connection ←→ Development VPC (10.1.0.0/16)
```

### 9. VPC Endpoints

**What is it?**
Private connection to AWS services without going through the internet.

**Types:**
- **Gateway Endpoint**: For S3 and DynamoDB (free, uses route table)
- **Interface Endpoint**: For other services (costs money, uses ENI)

**Why use it?**
- Security: Traffic never leaves AWS network
- Cost: No data transfer charges
- Performance: Lower latency

**Example:**
```
EC2 in Private Subnet → VPC Endpoint → S3 (no internet, no NAT needed)
```

### 10. Load Balancers

**Application Load Balancer (ALB):**
- **Layer 7** (HTTP/HTTPS)
- Content-based routing (path, host, headers)
- SSL termination
- Health checks
- Target groups (EC2, IPs, Lambda)

**Network Load Balancer (NLB):**
- **Layer 4** (TCP/UDP)
- Ultra-low latency
- Static IP addresses
- Preserves source IP
- Best for high performance

**Classic Load Balancer (CLB):**
- Legacy (avoid for new deployments)
- Layer 4 and Layer 7

**Traffic Flow:**
```
Internet → ALB/NLB → Target Group → EC2 Instances
```

---

## Kubernetes Networking Basics

### 1. Pod Networking

**What is a Pod?**
A pod is the smallest deployable unit in Kubernetes. It's like a container wrapper that can contain one or more containers.

**Pod IP Address:**
- Each pod gets its own IP address
- Pods can communicate directly using these IPs
- IPs are assigned by the Container Network Interface (CNI)

**Key Principle:**
> **"Every pod should be able to communicate with every other pod without NAT"**

**Example:**
```
Pod A: 10.244.1.5
Pod B: 10.244.2.10
Pod A can directly ping Pod B using 10.244.2.10
```

### 2. Container Network Interface (CNI)

**What is it?**
A plugin that Kubernetes uses to configure networking for pods.

**Popular CNI Plugins:**
- **AWS VPC CNI**: Uses AWS VPC IP addresses directly (EKS default)
- **Calico**: Policy-based networking
- **Flannel**: Simple overlay network
- **Cilium**: eBPF-based networking

**How it works:**
1. Pod is created
2. CNI plugin assigns IP address
3. CNI sets up routes and network interfaces
4. Pod can now communicate

### 3. Services

**What is it?**
A stable network endpoint to access pods. Pods come and go, but services provide a consistent way to reach them.

**Types of Services:**

#### ClusterIP (Default)
- **Internal only**: Only accessible within the cluster
- **Virtual IP**: Gets a virtual IP in the service CIDR range
- **Load balancing**: Distributes traffic to pod endpoints

**Example:**
```
Service: my-app (ClusterIP: 10.100.0.5)
  ↓
Pods: 10.244.1.5, 10.244.2.10, 10.244.3.15
```

#### NodePort
- **External access**: Exposes service on each node's IP at a specific port (30000-32767)
- **Access pattern**: `<NodeIP>:<NodePort>`

**Example:**
```
Node 1 (IP: 192.168.1.10): Port 30080 → Service → Pods
Node 2 (IP: 192.168.1.11): Port 30080 → Service → Pods
```

#### LoadBalancer
- **Cloud integration**: Creates a cloud load balancer (ALB/NLB in AWS)
- **External IP**: Gets a public IP address
- **Most common in production**

**Example:**
```
Internet → AWS Load Balancer → Service → Pods
```

#### ExternalName
- **DNS alias**: Maps service to external DNS name
- **No proxying**: Just DNS resolution

### 4. Service Discovery

**How pods find services:**
- **DNS**: Kubernetes has a built-in DNS server (CoreDNS)
- **Service name**: `<service-name>.<namespace>.svc.cluster.local`
- **Short form**: Just `<service-name>` in the same namespace

**Example:**
```
Pod in namespace "default" wants to reach "my-app" service:
- Full: my-app.default.svc.cluster.local
- Short: my-app (if in same namespace)
```

### 5. Ingress

**What is it?**
An API object that manages external HTTP/HTTPS access to services. Think of it as a smart router.

**How it works:**
- **Ingress Controller**: The actual implementation (like nginx, ALB, etc.)
- **Ingress Resource**: The configuration (rules, paths, hosts)

**Example:**
```
Internet → Ingress Controller → Ingress Rules → Services → Pods

Ingress Rule:
- Host: example.com
- Path: /api → service: api-service
- Path: /web → service: web-service
```

**AWS ALB Ingress Controller:**
- Creates AWS ALB automatically
- Handles SSL certificates
- Integrates with AWS Certificate Manager (ACM)

### 6. Network Policies

**What is it?**
Firewall rules for pods. Controls which pods can talk to which pods.

**Key Concepts:**
- **Default allow**: Without policies, all pods can communicate
- **Once applied**: Only explicitly allowed traffic is permitted
- **Namespace-scoped**: Policies apply within a namespace

**Example:**
```
NetworkPolicy: Allow frontend to backend
- Pods with label "app=frontend" can connect to
- Pods with label "app=backend" on port 8080
```

### 7. DNS in Kubernetes

**CoreDNS:**
- Default DNS server in Kubernetes
- Resolves service names to IPs
- Handles pod DNS queries

**DNS Records:**
- **Service A record**: `<service>.<namespace>.svc.cluster.local` → Service IP
- **Pod A record**: `<pod-ip>.<namespace>.pod.cluster.local` → Pod IP

---

## AWS EKS Integration

### 1. EKS Architecture Overview

**What is EKS?**
Amazon Elastic Kubernetes Service - managed Kubernetes on AWS.

**Components:**
```
EKS Control Plane (Managed by AWS)
  ↓
Worker Nodes (EC2 instances or Fargate)
  ↓
Pods running your applications
```

### 2. VPC CNI Plugin (AWS)

**How it works:**
- Pods get **real VPC IP addresses** (not overlay network)
- Each pod gets an ENI (Elastic Network Interface) or uses secondary IPs
- No NAT between pods and VPC resources

**Benefits:**
- Direct VPC integration
- Can use VPC security groups on pods
- No performance overhead of overlay networks

**Limitations:**
- Limited by ENI/IP limits per instance
- Must plan IP address allocation carefully

### 3. EKS Networking Components

#### Control Plane Networking
- **API Server**: Accessible via public or private endpoint
- **Private endpoint**: Only accessible from VPC
- **Public endpoint**: Accessible from internet (with authentication)

#### Worker Node Networking
- Nodes in **private subnets** (best practice)
- Nodes need access to:
  - EKS API endpoint
  - Container registry (ECR)
  - Internet (via NAT) for pulling images

#### Pod Networking
- Pods get IPs from VPC subnet CIDR
- Can communicate with:
  - Other pods (same or different nodes)
  - VPC resources (RDS, ElastiCache, etc.)
  - Internet (if node has NAT access)

### 4. Load Balancer Integration

#### ALB Ingress Controller
**How it works:**
1. You create an Ingress resource
2. ALB Ingress Controller watches for Ingress resources
3. Creates AWS ALB automatically
4. Configures target groups pointing to nodes
5. Updates ALB as pods change

**Traffic Flow:**
```
Internet → ALB → Target Group → NodePort → Service → Pod
```

#### NLB for Services
**Type: LoadBalancer with NLB:**
```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
```

**Traffic Flow:**
```
Internet → NLB → Node IP:NodePort → Service → Pod
```

### 5. Security Groups in EKS

**Node Security Group:**
- Applied to worker nodes
- Allows traffic from ALB/NLB
- Allows inter-node communication
- Allows access to EKS API

**Pod Security Groups (VPC CNI):**
- Can assign security groups directly to pods
- More granular control
- Requires VPC CNI with security group support

### 6. EKS Networking Best Practices

1. **Use private subnets for nodes**: More secure
2. **Use NAT Gateway**: For outbound internet access
3. **Plan IP addresses**: VPC CNI uses real VPC IPs
4. **Use ALB Ingress Controller**: For HTTP/HTTPS traffic
5. **Use NLB**: For TCP/UDP or high-performance needs
6. **Enable VPC Flow Logs**: For network monitoring
7. **Use Network Policies**: For pod-to-pod security

---

## End-to-End Traffic Flow Scenarios

### Scenario 1: Internet to Kubernetes Pod (HTTP/HTTPS)

**Setup:**
- EKS cluster with ALB Ingress Controller
- Application pods in private subnets
- Ingress resource configured

**Traffic Flow:**
```
1. User → Internet
   User types: https://myapp.example.com

2. Internet → Route 53 (DNS)
   DNS resolves to ALB IP address

3. Route 53 → ALB
   ALB receives HTTPS request on port 443

4. ALB → SSL Termination
   ALB terminates SSL using ACM certificate

5. ALB → Target Group
   ALB forwards HTTP request to target group
   Target group contains: Node IPs on NodePort (e.g., 30080)

6. Target Group → Worker Node
   Traffic hits node's NodePort (e.g., 192.168.1.10:30080)

7. Worker Node → kube-proxy
   kube-proxy (running on node) receives traffic
   Looks up service endpoints

8. kube-proxy → Service
   Service selects pods based on labels
   Load balances to one of the pod IPs

9. Service → Pod
   Traffic forwarded to pod (e.g., 10.0.10.5:8080)

10. Pod → Application
    Application processes request and responds

11. Response follows reverse path back to user
```

**Key Components:**
- **ALB**: Layer 7 load balancer, SSL termination
- **Target Group**: Health checks, routing rules
- **NodePort**: Exposes service on each node
- **Service**: Stable endpoint, load balancing
- **Pod**: Actual application container

### Scenario 2: Pod to Pod Communication (Same Node)

**Setup:**
- Two pods on the same worker node
- Both pods in same VPC subnet

**Traffic Flow:**
```
1. Pod A (10.0.10.5) wants to reach Pod B (10.0.10.6)

2. Pod A → VPC CNI
   Pod A's network namespace sends packet
   Source: 10.0.10.5, Destination: 10.0.10.6

3. VPC CNI → Node's Network Stack
   Packet goes through node's network interface
   Linux routing determines it's local

4. Node's Network Stack → Pod B
   Since both pods on same node, traffic stays local
   Directly forwarded to Pod B's network namespace

5. Pod B receives packet and processes
```

**Key Points:**
- No NAT involved
- Traffic stays on the node
- Very low latency
- Uses Linux network namespaces

### Scenario 3: Pod to Pod Communication (Different Nodes)

**Setup:**
- Pod A on Node 1 (subnet 10.0.10.0/24)
- Pod B on Node 2 (subnet 10.0.11.0/24)
- Both in same VPC

**Traffic Flow:**
```
1. Pod A (10.0.10.5) wants to reach Pod B (10.0.11.10)

2. Pod A → Node 1's Network Stack
   Packet: Source 10.0.10.5, Dest 10.0.11.10

3. Node 1 → VPC Routing
   Node 1's route table says:
   - 10.0.11.0/24 is in different subnet
   - Route via VPC routing (local route)

4. VPC Routing → Node 2
   VPC routing table forwards packet to Node 2
   (Both nodes in same VPC, can route directly)

5. Node 2 → Pod B
   Node 2 receives packet
   VPC CNI forwards to Pod B's network namespace

6. Pod B receives and processes
```

**Key Points:**
- Uses VPC routing (not overlay network)
- Traffic goes through VPC network
- Still no NAT (direct pod-to-pod)
- Works because pods have real VPC IPs

### Scenario 4: Pod to AWS Service (RDS)

**Setup:**
- Pod in EKS cluster
- RDS database in same VPC
- Security groups configured

**Traffic Flow:**
```
1. Pod (10.0.10.5) wants to connect to RDS (10.0.20.50:5432)

2. Pod → Node's Network Stack
   Packet: Source 10.0.10.5, Dest 10.0.20.50:5432

3. Node → VPC Routing
   Route table determines 10.0.20.50 is in different subnet
   Routes via VPC (local route)

4. VPC Routing → RDS Subnet
   Packet forwarded to RDS subnet

5. Security Group Check (RDS)
   RDS security group checks:
   - Inbound rule allows port 5432 from pod's security group
   - ✅ Allowed

6. RDS receives connection
   Database processes query

7. Response follows reverse path
```

**Key Points:**
- Direct VPC communication (no internet)
- Security groups control access
- Low latency (same VPC)
- No data transfer charges

### Scenario 5: Pod to Internet (Outbound)

**Setup:**
- Pod in private subnet
- Node has NAT Gateway access

**Traffic Flow:**
```
1. Pod (10.0.10.5) wants to reach api.example.com

2. Pod → Node's Network Stack
   DNS query first (if needed)
   Then HTTP request

3. Node → NAT Gateway
   Node's route table says:
   - 0.0.0.0/0 → NAT Gateway
   - Source IP: Node's private IP (10.0.10.100)
   - Destination: api.example.com IP

4. NAT Gateway → Internet Gateway
   NAT Gateway performs Source NAT (SNAT)
   Changes source IP to NAT Gateway's public IP
   Forwards to Internet Gateway

5. Internet Gateway → Internet
   Packet goes to public internet
   Reaches api.example.com

6. Response comes back:
   Internet → IGW → NAT Gateway → Node → Pod
   NAT Gateway remembers connection (stateful)
   Forwards response to correct node/pod
```

**Key Points:**
- NAT Gateway performs Source NAT
- Pod's IP is hidden (uses NAT's public IP)
- Stateful connection tracking
- One-way: Pod can initiate, but internet can't reach pod directly

### Scenario 6: Internet to Pod via NLB (TCP/UDP)

**Setup:**
- Service type: LoadBalancer with NLB
- TCP-based application

**Traffic Flow:**
```
1. Internet → NLB
   Client connects to NLB's public IP:Port

2. NLB → Target Group
   NLB forwards to target group
   Targets: Node IPs on NodePort

3. Target Group → Worker Node
   Health checks ensure node is healthy
   Forwards to node (e.g., 192.168.1.10:30080)

4. Worker Node → kube-proxy
   kube-proxy receives on NodePort
   Looks up service endpoints

5. kube-proxy → Service
   Service selects pod
   Load balances to pod IP

6. Service → Pod
   Traffic forwarded to pod

7. Pod processes and responds
```

**Key Differences from ALB:**
- **Layer 4**: Works with TCP/UDP (not just HTTP)
- **Preserves source IP**: Can see original client IP
- **Lower latency**: No HTTP processing overhead
- **Static IPs**: NLB provides static IP addresses

### Scenario 7: Pod to Pod via Service

**Setup:**
- Frontend pod wants to reach backend service
- Service type: ClusterIP

**Traffic Flow:**
```
1. Frontend Pod (10.0.10.5) wants backend-service

2. Frontend Pod → DNS Query
   Queries CoreDNS: backend-service.default.svc.cluster.local
   CoreDNS responds: 10.100.0.50 (Service ClusterIP)

3. Frontend Pod → Service IP
   Sends packet to 10.100.0.50:8080

4. Service IP → kube-proxy
   kube-proxy intercepts (using iptables or IPVS)
   Looks up endpoints for backend-service

5. kube-proxy → Endpoint Selection
   Service has 3 endpoints (pods):
   - 10.0.10.10:8080
   - 10.0.11.5:8080
   - 10.0.11.15:8080
   Selects one (round-robin or random)

6. kube-proxy → Destination NAT (DNAT)
   Changes destination IP from 10.100.0.50 to 10.0.10.10
   Forwards packet

7. Packet → Backend Pod
   Backend pod (10.0.10.10) receives request

8. Response:
   Backend pod responds directly to frontend pod (10.0.10.5)
   (kube-proxy is transparent for return traffic)
```

**Key Points:**
- Service IP is virtual (doesn't exist on wire)
- kube-proxy does DNAT (Destination NAT)
- Load balancing happens at kube-proxy level
- Return traffic bypasses service (direct pod-to-pod)

---

## Common Interview Scenarios

### Scenario 1: "How would you design networking for a 3-tier application on EKS?"

**Answer Structure:**

**Architecture:**
```
Internet → ALB → Ingress → Services → Pods

Tiers:
1. Web Tier (Frontend)
   - Public-facing
   - ALB Ingress
   - Public subnets (optional) or private with ALB

2. App Tier (Backend API)
   - Internal only
   - ClusterIP service
   - Private subnets

3. Data Tier (Database)
   - RDS in private subnet
   - No Kubernetes, separate AWS service
```

**Detailed Design:**

1. **VPC Setup:**
   ```
   VPC: 10.0.0.0/16
   
   Public Subnets (for ALB):
   - 10.0.1.0/24 (us-east-1a)
   - 10.0.2.0/24 (us-east-1b)
   
   Private Subnets (for EKS nodes):
   - 10.0.10.0/24 (us-east-1a)
   - 10.0.11.0/24 (us-east-1b)
   
   Database Subnets:
   - 10.0.20.0/24 (us-east-1a)
   - 10.0.21.0/24 (us-east-1b)
   ```

2. **Networking Components:**
   - **Internet Gateway**: For ALB public access
   - **NAT Gateway**: For nodes to pull images, access ECR
   - **Route Tables**: 
     - Public: Route to IGW
     - Private: Route to NAT Gateway
   - **Security Groups**:
     - ALB SG: Allow 80, 443 from internet
     - Node SG: Allow from ALB SG, inter-node communication
     - RDS SG: Allow 3306 from node SG

3. **Kubernetes Setup:**
   - **EKS Cluster**: In private subnets
   - **Worker Nodes**: Managed node groups in private subnets
   - **VPC CNI**: For pod networking
   - **ALB Ingress Controller**: For web tier
   - **Services**:
     - Web: LoadBalancer or Ingress
     - App: ClusterIP (internal)
   - **Network Policies**: Restrict app tier to only accept from web tier

4. **Traffic Flow:**
   ```
   User → Internet → ALB → Ingress → Web Service → Web Pods
   Web Pod → App Service (ClusterIP) → App Pods
   App Pod → RDS (direct VPC) → Database
   ```

### Scenario 2: "A pod cannot reach the internet. How do you troubleshoot?"

**Troubleshooting Steps:**

1. **Check Pod's Network Configuration:**
   ```bash
   kubectl exec -it <pod-name> -- ip addr
   kubectl exec -it <pod-name> -- route -n
   ```
   - Verify pod has IP address
   - Check routing table

2. **Check Node's Network:**
   ```bash
   # SSH to node
   ip route show
   ```
   - Verify route to NAT Gateway (0.0.0.0/0 → nat-xxx)
   - Check if node can reach internet

3. **Check Security Groups:**
   - Node security group: Allow outbound to 0.0.0.0/0
   - Pod security group (if using VPC CNI with SG): Allow outbound

4. **Check Route Tables:**
   - Private subnet route table should have: 0.0.0.0/0 → NAT Gateway
   - Verify NAT Gateway is in correct subnet

5. **Test Connectivity:**
   ```bash
   # From pod
   kubectl exec -it <pod-name> -- curl -v https://www.google.com
   
   # From node
   curl -v https://www.google.com
   ```

6. **Check NAT Gateway:**
   - Verify NAT Gateway is running
   - Check NAT Gateway route table association
   - Verify NAT Gateway has Elastic IP

7. **Check VPC CNI:**
   ```bash
   kubectl get pods -n kube-system | grep vpc-cni
   kubectl logs -n kube-system <vpc-cni-pod>
   ```

**Common Issues:**
- Missing NAT Gateway route
- Security group blocking outbound
- NAT Gateway in wrong subnet
- VPC CNI not configured properly

### Scenario 3: "How do you ensure pods in different namespaces cannot communicate?"

**Solution: Network Policies**

1. **Enable Network Policy Support:**
   - EKS with Calico CNI, or
   - VPC CNI with network policy support

2. **Create Default Deny Policy:**
   ```yaml
   # namespace-default-deny.yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: default-deny-all
     namespace: production
   spec:
     podSelector: {}
     policyTypes:
     - Ingress
     - Egress
   ```

3. **Create Allow Policies:**
   ```yaml
   # Allow frontend to backend
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: allow-frontend-to-backend
     namespace: production
   spec:
     podSelector:
       matchLabels:
         app: backend
     policyTypes:
     - Ingress
     ingress:
     - from:
       - namespaceSelector:
           matchLabels:
             name: production
       - podSelector:
           matchLabels:
             app: frontend
       ports:
       - protocol: TCP
         port: 8080
   ```

4. **Namespace Isolation:**
   - Apply default deny to each namespace
   - Explicitly allow only required communication
   - Use namespace selectors to control cross-namespace access

### Scenario 4: "How would you expose a gRPC service externally?"

**Options:**

**Option 1: NLB (Recommended for gRPC)**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: grpc-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
  ports:
  - port: 50051
    targetPort: 50051
    protocol: TCP
  selector:
    app: grpc-app
```

**Why NLB?**
- gRPC uses HTTP/2 over TCP
- NLB is Layer 4, works with any TCP protocol
- Lower latency than ALB
- Preserves source IP

**Option 2: ALB with gRPC support**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grpc-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/backend-protocol: GRPC
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
spec:
  rules:
  - host: grpc.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grpc-service
            port:
              number: 50051
```

**Traffic Flow:**
```
Client → NLB/ALB → NodePort → Service → gRPC Pods
```

### Scenario 5: "How do you handle IP address exhaustion in EKS?"

**Problem:**
- VPC CNI uses real VPC IPs
- Limited by ENI and IP limits per instance
- Running out of IPs as cluster grows

**Solutions:**

1. **Use Secondary IP Ranges:**
   - Assign secondary CIDR blocks to VPC
   - Configure VPC CNI to use secondary IPs
   - Increases available IP pool

2. **Use Fargate:**
   - Pods run on AWS-managed infrastructure
   - No need to manage IP allocation
   - Scales automatically

3. **Optimize IP Usage:**
   - Use host networking for specific pods (if appropriate)
   - Reduce number of pods per node
   - Use larger instance types (more ENIs/IPs)

4. **Switch CNI:**
   - Use overlay network CNI (Calico, Flannel)
   - Pods get overlay IPs, not VPC IPs
   - Trade-off: Lose some VPC integration features

5. **VPC CNI Configuration:**
   ```yaml
   # vpc-cni config
   env:
   - name: WARM_IP_TARGET
     value: "2"  # Reduce warm IP pool
   - name: MINIMUM_IP_TARGET
     value: "2"  # Minimum IPs per ENI
   ```

### Scenario 6: "Design networking for multi-region EKS deployment"

**Architecture:**

1. **Per-Region Setup:**
   ```
   Region 1 (us-east-1):
   - VPC: 10.0.0.0/16
   - EKS Cluster
   - ALB
   
   Region 2 (us-west-2):
   - VPC: 10.1.0.0/16
   - EKS Cluster
   - ALB
   ```

2. **Global Traffic Management:**
   - **Route 53**: Health checks, failover, geolocation
   - **CloudFront**: CDN in front (optional)
   - **Global Accelerator**: For lower latency (optional)

3. **Cross-Region Communication:**
   - **VPC Peering**: Direct connection (if needed)
   - **Transit Gateway**: Hub-and-spoke model
   - **PrivateLink**: For AWS services
   - **VPN**: For on-premises connectivity

4. **Data Replication:**
   - RDS Cross-Region Read Replicas
   - DynamoDB Global Tables
   - S3 Cross-Region Replication

5. **Traffic Flow:**
   ```
   User → Route 53 → Region 1 ALB → EKS → App
                    ↓ (if Region 1 down)
                    Region 2 ALB → EKS → App
   ```

### Scenario 7: "How do you secure pod-to-pod communication?"

**Multi-Layer Security:**

1. **Network Policies:**
   - Define allowed ingress/egress
   - Label-based selection
   - Namespace isolation

2. **Security Groups (VPC CNI):**
   - Assign security groups to pods
   - Traditional AWS security model
   - Works with VPC CNI

3. **Service Mesh (Istio/Linkerd):**
   - mTLS between pods
   - Fine-grained policies
   - Observability

4. **Pod Security Standards:**
   - Run pods as non-root
   - Read-only root filesystem
   - Drop unnecessary capabilities

**Example Network Policy:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: secure-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to: []  # Allow DNS
    ports:
    - protocol: UDP
      port: 53
```

---

## Troubleshooting Guide

### Common Issues and Solutions

#### Issue 1: Pods cannot reach each other

**Symptoms:**
- Pod A cannot ping Pod B
- Connection timeouts

**Checks:**
1. Verify pods have IP addresses: `kubectl get pods -o wide`
2. Check if pods are in same namespace (if using network policies)
3. Verify network policies allow communication
4. Check security groups (if using VPC CNI with SG)
5. Verify VPC CNI is running: `kubectl get pods -n kube-system | grep vpc-cni`

**Solutions:**
- Fix network policies
- Check security group rules
- Restart VPC CNI pods
- Verify route tables

#### Issue 2: Service not accessible

**Symptoms:**
- Cannot reach service from pod
- DNS resolution fails

**Checks:**
1. Verify service exists: `kubectl get svc`
2. Check service endpoints: `kubectl get endpoints <service-name>`
3. Test DNS: `kubectl run -it --rm debug --image=busybox -- nslookup <service-name>`
4. Check kube-proxy: `kubectl get pods -n kube-system | grep kube-proxy`
5. Verify CoreDNS: `kubectl get pods -n kube-system | grep coredns`

**Solutions:**
- Ensure pods have correct labels (matching service selector)
- Restart kube-proxy if needed
- Check CoreDNS logs
- Verify service selector matches pod labels

#### Issue 3: Ingress not working

**Symptoms:**
- Cannot access application via Ingress URL
- 404 or connection refused

**Checks:**
1. Verify Ingress resource: `kubectl get ingress`
2. Check Ingress Controller: `kubectl get pods -n kube-system | grep ingress`
3. Verify ALB was created: Check AWS Console
4. Check target group health: AWS Console → EC2 → Target Groups
5. Verify service exists and has endpoints
6. Check Ingress Controller logs

**Solutions:**
- Ensure Ingress Controller is installed and running
- Verify Ingress annotations are correct
- Check ALB security group allows traffic
- Verify service is correctly referenced in Ingress

#### Issue 4: High latency between pods

**Symptoms:**
- Slow communication between pods
- High network latency

**Checks:**
1. Check if pods are on same node: `kubectl get pods -o wide`
2. Verify VPC CNI configuration
3. Check for network policies (may add overhead)
4. Monitor network metrics
5. Check instance types (network performance)

**Solutions:**
- Use placement rules to co-locate related pods
- Optimize network policies
- Use enhanced networking on instances
- Consider service mesh for better routing

#### Issue 5: IP address exhaustion

**Symptoms:**
- Pods stuck in Pending state
- "Insufficient IP addresses" errors

**Checks:**
1. Check available IPs in subnets: AWS Console → VPC → Subnets
2. Verify ENI limits per instance type
3. Check VPC CNI configuration
4. Count pods per node

**Solutions:**
- Add secondary CIDR to VPC
- Use larger instance types (more ENIs)
- Reduce pods per node
- Consider Fargate
- Switch to overlay CNI

---

## Key Takeaways for Interviews

### AWS Networking Essentials:
1. **VPC** = Your private network in AWS
2. **Subnets** = Smaller networks within VPC (public/private)
3. **IGW** = Door to internet (for public subnets)
4. **NAT Gateway** = One-way door to internet (for private subnets)
5. **Route Tables** = GPS for network traffic
6. **Security Groups** = Firewall at instance level (stateful)
7. **NACLs** = Firewall at subnet level (stateless)
8. **Load Balancers** = Distribute traffic (ALB Layer 7, NLB Layer 4)

### Kubernetes Networking Essentials:
1. **Pods** = Smallest unit, each gets an IP
2. **Services** = Stable endpoint to reach pods
3. **Ingress** = Smart router for HTTP/HTTPS
4. **CNI** = Plugin that gives pods IP addresses
5. **Network Policies** = Firewall rules for pods
6. **DNS** = Service discovery (CoreDNS)

### EKS Integration:
1. **VPC CNI** = Pods get real VPC IPs
2. **ALB Ingress Controller** = Creates ALB from Ingress
3. **NLB** = For TCP/UDP or high performance
4. **Security Groups** = Can apply to pods (with VPC CNI)

### Traffic Flow Patterns:
1. **Internet → Pod**: Internet → ALB/NLB → NodePort → Service → Pod
2. **Pod → Pod (same node)**: Direct, no routing
3. **Pod → Pod (different node)**: Via VPC routing
4. **Pod → AWS Service**: Direct VPC, security groups
5. **Pod → Internet**: Pod → Node → NAT Gateway → IGW → Internet

### Interview Tips:
1. **Always start with the user/request**: "A user makes a request..."
2. **Follow the packet**: Trace the packet through each component
3. **Mention security**: Security groups, network policies, encryption
4. **Consider failure scenarios**: What if a component fails?
5. **Think about scale**: How does this work with 1000 pods?
6. **Use diagrams**: Draw it out if possible (mentally or on paper)

---

## Additional Resources

### AWS Services to Know:
- VPC, Subnets, IGW, NAT Gateway
- Route Tables, Security Groups, NACLs
- ALB, NLB, CLB
- VPC Peering, VPC Endpoints
- Route 53, CloudFront
- EKS, ECR

### Kubernetes Concepts:
- Pods, Services, Ingress
- Network Policies
- CNI plugins
- DNS (CoreDNS)
- kube-proxy

### Tools for Troubleshooting:
- `kubectl`: Get pods, services, endpoints, logs
- `ip`, `route`, `netstat`: Network debugging on nodes
- `curl`, `wget`, `nslookup`: Connectivity testing
- AWS Console: VPC, EC2, Load Balancers
- VPC Flow Logs: Network traffic analysis

---

**Remember**: Networking is about understanding how data flows from point A to point B. Always think about:
1. **Source**: Where is the request coming from?
2. **Destination**: Where does it need to go?
3. **Path**: What components does it pass through?
4. **Security**: What checks happen along the way?
5. **Failure**: What if something breaks?

Good luck with your interviews! 🚀

