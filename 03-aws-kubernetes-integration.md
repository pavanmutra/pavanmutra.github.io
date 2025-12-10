# AWS + Kubernetes Integration: End-to-End Networking Guide

## Table of Contents
1. [Overview](#overview)
2. [EKS Architecture](#eks-architecture)
3. [VPC Integration](#vpc-integration)
4. [CNI Plugin (AWS VPC CNI)](#cni-plugin-aws-vpc-cni)
5. [Load Balancer Integration](#load-balancer-integration)
6. [End-to-End Traffic Flows](#end-to-end-traffic-flows)
7. [Common Scenarios](#common-scenarios)
8. [Interview Scenarios](#interview-scenarios)

---

## Overview

### What is EKS?

**Amazon EKS (Elastic Kubernetes Service)** is a managed Kubernetes service. It runs Kubernetes control plane on AWS infrastructure, while you manage worker nodes.

### Key Integration Points

1. **VPC Integration:** EKS cluster runs in your VPC
2. **CNI Plugin:** AWS VPC CNI assigns VPC IPs to pods
3. **Load Balancers:** Kubernetes services integrate with AWS ELB
4. **IAM Integration:** Kubernetes RBAC integrates with AWS IAM
5. **Networking:** Pods get real VPC IPs (no overlay)

---

## EKS Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      AWS VPC                            │
│                   (10.0.0.0/16)                        │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │         EKS Control Plane (Managed)              │  │
│  │  - API Server                                    │  │
│  │  - etcd                                          │  │
│  │  - Scheduler                                     │  │
│  │  - Controller Manager                            │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
│  ┌──────────────────┐      ┌──────────────────┐      │
│  │   Public Subnet   │      │   Public Subnet   │      │
│  │   (AZ-1)          │      │   (AZ-2)          │      │
│  │   10.0.1.0/24     │      │   10.0.2.0/24     │      │
│  │                   │      │                   │      │
│  │  ┌─────────────┐  │      │  ┌─────────────┐  │      │
│  │  │ Worker Node │  │      │  │ Worker Node │  │      │
│  │  │             │  │      │  │             │  │      │
│  │  │  Pods       │  │      │  │  Pods       │  │      │
│  │  └─────────────┘  │      │  └─────────────┘  │      │
│  └──────────────────┘      └──────────────────┘      │
│                                                         │
│  ┌──────────────────┐      ┌──────────────────┐      │
│  │  Private Subnet   │      │  Private Subnet   │      │
│  │  (AZ-1)           │      │  (AZ-2)           │      │
│  │  10.0.3.0/24      │      │  10.0.4.0/24      │      │
│  │                   │      │                   │      │
│  │  ┌─────────────┐  │      │  ┌─────────────┐  │      │
│  │  │ Worker Node │  │      │  │ Worker Node │  │      │
│  │  │             │  │      │  │             │  │      │
│  │  │  Pods       │  │      │  │  Pods       │  │      │
│  │  └─────────────┘  │      │  └─────────────┘  │      │
│  └──────────────────┘      └──────────────────┘      │
│                                                         │
│  ┌──────────────┐                                      │
│  │ NAT Gateway  │                                      │
│  │ (AZ-1)       │                                      │
│  └──────────────┘                                      │
│                                                         │
│  ┌──────────────┐                                      │
│  │ NAT Gateway  │                                      │
│  │ (AZ-2)       │                                      │
│  └──────────────┘                                      │
│                                                         │
│  ┌──────────────┐                                      │
│  │ Internet     │                                      │
│  │ Gateway      │                                      │
│  └──────────────┘                                      │
└─────────────────────────────────────────────────────────┘
```

### EKS Components

1. **Control Plane:**
   - Managed by AWS
   - Highly available (spans multiple AZs)
   - Endpoint: `https://<cluster-id>.eks.<region>.amazonaws.com`

2. **Worker Nodes:**
   - EC2 instances (or Fargate)
   - Run in your VPC subnets
   - Managed by you (or node groups)

3. **VPC CNI:**
   - Assigns VPC IPs to pods
   - No overlay network
   - Direct VPC integration

---

## VPC Integration

### VPC Requirements for EKS

**Minimum Requirements:**
- At least 2 subnets in different AZs
- Subnets must have sufficient IP addresses
- Internet access (for pulling images, API access)
- Route tables configured correctly

### Subnet Configuration

**Public Subnets:**
- For load balancers
- For NAT Gateways
- Route to Internet Gateway

**Private Subnets:**
- For worker nodes (recommended)
- Route to NAT Gateway (for outbound internet)

### IP Address Planning

**Important:** Pods get VPC IPs, not overlay IPs!

**Calculation:**
```
Total IPs needed = 
  (Number of nodes × IPs per node) + 
  (Number of pods × 1 IP per pod) +
  (Load balancers) +
  (NAT Gateways) +
  (Reserved IPs)

Example:
- 10 nodes × 1 IP = 10 IPs
- 100 pods × 1 IP = 100 IPs
- 5 load balancers = 5 IPs
- 2 NAT Gateways = 2 IPs
- Reserved = 5 IPs
Total = 122 IPs needed

Subnet size: /24 = 256 IPs (sufficient)
```

**Best Practice:**
- Use /24 subnets (256 IPs) minimum
- Plan for growth
- Consider using /22 or /20 for large clusters

---

## CNI Plugin (AWS VPC CNI)

### What is AWS VPC CNI?

**AWS VPC CNI** is the default CNI plugin for EKS. It assigns VPC IP addresses directly to pods (no overlay network).

### How AWS VPC CNI Works

```
1. Pod is scheduled to node
   ↓
2. VPC CNI plugin (daemonset) on node receives request
   ↓
3. VPC CNI allocates IP from VPC subnet
   ↓
4. IP is assigned from secondary IP range (ENI)
   ↓
5. Pod gets real VPC IP (e.g., 10.0.3.50)
   ↓
6. No encapsulation needed (direct VPC routing)
```

### Key Concepts

**Elastic Network Interface (ENI):**
- Each EC2 instance has primary ENI
- VPC CNI attaches secondary IPs to ENI
- Each ENI can have multiple IPs

**Secondary IP Ranges:**
- VPC CNI uses secondary IPs on ENI
- Each node can have multiple secondary IPs
- Limited by instance type (e.g., m5.large = 10 IPs per ENI)

**Warm Pool:**
- VPC CNI pre-allocates IPs
- Reduces pod startup time
- Configurable via environment variables

### VPC CNI Configuration

**Environment Variables:**
```yaml
env:
  - name: WARM_ENI_TARGET
    value: "1"  # Pre-allocate 1 ENI
  - name: WARM_IP_TARGET
    value: "2"  # Pre-allocate 2 IPs
  - name: MINIMUM_IP_TARGET
    value: "2"  # Minimum IPs to keep
```

**Benefits:**
- No overlay network overhead
- Better performance
- Direct VPC integration
- Pods appear as VPC resources

**Limitations:**
- Limited by ENI/IP capacity per instance
- Must plan IP addresses carefully
- Cannot exceed VPC subnet size

---

## Load Balancer Integration

### AWS Load Balancer Controller

**AWS Load Balancer Controller** is a Kubernetes controller that provisions AWS load balancers for Kubernetes services.

### Service Types in EKS

#### 1. LoadBalancer Service

**How it works:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
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
2. AWS Load Balancer Controller watches API server
   ↓
3. Controller creates AWS NLB/ALB
   ↓
4. Controller creates target groups
   ↓
5. Controller registers nodes as targets
   ↓
6. Load balancer gets DNS name
   ↓
7. Traffic: Internet → NLB → Nodes → Pods
```

#### 2. Ingress with ALB

**How it works:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
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
2. AWS Load Balancer Controller creates ALB
   ↓
3. ALB target type: IP (direct to pods)
   ↓
4. Controller registers pod IPs as targets
   ↓
5. ALB health checks pods directly
   ↓
6. Traffic: Internet → ALB → Pod IPs (direct)
```

**Key Difference:**
- **NodePort mode:** ALB → Nodes → kube-proxy → Pods
- **IP mode:** ALB → Pod IPs (direct, better performance)

### Load Balancer Types

**Network Load Balancer (NLB):**
- Layer 4 (TCP/UDP)
- Best for high performance
- Preserves source IP
- Static IP addresses

**Application Load Balancer (ALB):**
- Layer 7 (HTTP/HTTPS)
- Best for web applications
- Path/host-based routing
- SSL termination

---

## End-to-End Traffic Flows

### Scenario 1: User Accesses Web Application via ALB Ingress

**Architecture:**
- VPC: 10.0.0.0/16
- Public Subnet: 10.0.1.0/24 (ALB)
- Private Subnet: 10.0.3.0/24 (Worker Nodes)
- Pods: 10.0.3.50, 10.0.3.51

**Traffic Flow:**

```
1. User accesses myapp.com
   ↓
2. DNS resolves to ALB DNS name
   ↓
3. Route 53 / External DNS returns ALB IP
   ↓
4. User's browser sends HTTP request to ALB IP
   ↓
5. Traffic enters AWS VPC via Internet Gateway
   ↓
6. VPC route table routes to ALB (in public subnet)
   ↓
7. ALB receives request
   ↓
8. ALB checks Ingress rules (host: myapp.com, path: /)
   ↓
9. ALB routes to target group (pod IPs: 10.0.3.50, 10.0.3.51)
   ↓
10. ALB sends request directly to pod IP (10.0.3.50)
    ↓
11. VPC route table routes to private subnet (10.0.3.0/24 → local)
    ↓
12. Traffic reaches worker node's ENI
    ↓
13. VPC CNI routes to pod (secondary IP on ENI)
    ↓
14. Pod receives request
    ↓
15. Pod processes request
    ↓
16. Response flows back:
    Pod → ENI → VPC Route Table → ALB → IGW → Internet → User
```

**Key Points:**
- ALB targets pod IPs directly (IP mode)
- No kube-proxy involved (direct routing)
- Pods have real VPC IPs
- Traffic stays within VPC until IGW

### Scenario 2: Pod-to-Pod Communication (Same Node)

**Architecture:**
- Node: 10.0.3.10
- Pod A: 10.0.3.50 (secondary IP on ENI)
- Pod B: 10.0.3.51 (secondary IP on ENI)

**Traffic Flow:**

```
1. Pod A (10.0.3.50) wants to reach Pod B (10.0.3.51)
   ↓
2. Pod A sends packet with destination 10.0.3.51
   ↓
3. Linux kernel on node receives packet
   ↓
4. Kernel routes to ENI (same interface)
   ↓
5. ENI delivers to Pod B (secondary IP)
   ↓
6. Pod B receives packet
   ↓
7. Response flows back through same path
```

**Key Points:**
- No network hop (same node)
- Direct ENI routing
- Very low latency

### Scenario 3: Pod-to-Pod Communication (Different Nodes)

**Architecture:**
- Node 1: 10.0.3.10 (Pod A: 10.0.3.50)
- Node 2: 10.0.4.10 (Pod B: 10.0.4.50)

**Traffic Flow:**

```
1. Pod A (10.0.3.50) wants to reach Pod B (10.0.4.50)
   ↓
2. Pod A sends packet with destination 10.0.4.50
   ↓
3. Linux kernel on Node 1 routes packet
   ↓
4. VPC route table on Node 1:
   10.0.4.0/24 → local (same VPC)
   ↓
5. Packet goes to Node 1's ENI
   ↓
6. VPC routing forwards to Node 2's subnet (10.0.4.0/24)
   ↓
7. VPC route table routes to Node 2 (10.0.4.10)
   ↓
8. Packet reaches Node 2's ENI
   ↓
9. ENI delivers to Pod B (secondary IP 10.0.4.50)
   ↓
10. Pod B receives packet
    ↓
11. Response flows back:
    Pod B → Node 2 ENI → VPC Route → Node 1 ENI → Pod A
```

**Key Points:**
- Uses VPC routing (no overlay)
- Direct VPC IP communication
- VPC handles routing between subnets

### Scenario 4: Pod Accesses Internet

**Architecture:**
- Pod: 10.0.3.50 (private subnet)
- NAT Gateway: 10.0.1.50 (public subnet)

**Traffic Flow:**

```
1. Pod (10.0.3.50) wants to download package from internet
   ↓
2. Pod sends packet with destination: external IP
   ↓
3. Node's route table checks:
   - 10.0.0.0/16 → local (stays in VPC)
   - 0.0.0.0/0 → NAT Gateway (10.0.1.50)
   ↓
4. Packet routed to NAT Gateway
   ↓
5. NAT Gateway translates source IP:
   10.0.3.50 → NAT Gateway's Elastic IP
   ↓
6. NAT Gateway forwards to Internet Gateway
   ↓
7. Internet Gateway sends to internet
   ↓
8. Response comes back to NAT Gateway's Elastic IP
   ↓
9. NAT Gateway remembers connection (stateful)
   ↓
10. NAT Gateway forwards to original pod (10.0.3.50)
    ↓
11. VPC routing delivers to pod
    ↓
12. Pod receives response
```

**Key Points:**
- Pod uses NAT Gateway for outbound internet
- NAT Gateway is stateful
- Internet cannot initiate connection to pod

### Scenario 5: Pod Accesses RDS Database

**Architecture:**
- Pod: 10.0.3.50 (private subnet)
- RDS: 10.0.5.20 (private subnet, different subnet)

**Traffic Flow:**

```
1. Pod (10.0.3.50) wants to connect to RDS (10.0.5.20:5432)
   ↓
2. Pod sends packet with destination 10.0.5.20:5432
   ↓
3. Node's route table:
   10.0.0.0/16 → local (stays in VPC)
   ↓
4. VPC route table routes to RDS subnet (10.0.5.0/24)
   ↓
5. Traffic reaches RDS security group
   ↓
6. Security Group checks:
   - Allows inbound from pod's security group? ✅
   ↓
7. RDS receives connection
   ↓
8. Response flows back through same path
```

**Key Points:**
- Direct VPC communication
- Security Groups control access
- No internet involved
- Low latency

### Scenario 6: External Service Accesses Pod via NLB

**Architecture:**
- NLB in public subnet
- Pods in private subnet
- Service type: LoadBalancer (NLB)

**Traffic Flow:**

```
1. External user accesses NLB DNS
   ↓
2. DNS resolves to NLB IP
   ↓
3. User sends request to NLB IP
   ↓
4. Traffic enters VPC via Internet Gateway
   ↓
5. VPC routes to NLB (public subnet)
   ↓
6. NLB receives request
   ↓
7. NLB routes to target group (node IPs)
   ↓
8. Traffic reaches node (NodePort)
   ↓
9. kube-proxy intercepts NodePort traffic
   ↓
10. kube-proxy routes to Service ClusterIP
    ↓
11. iptables routes to pod IP
    ↓
12. VPC CNI routes to pod
    ↓
13. Pod receives request
    ↓
14. Response flows back:
    Pod → kube-proxy → NodePort → NLB → IGW → Internet
```

**Key Points:**
- NLB uses NodePort mode (targets nodes)
- kube-proxy handles service routing
- More hops than ALB IP mode

---

## Common Scenarios

### Scenario 1: High Availability Setup

**Architecture:**
```
VPC: 10.0.0.0/16
├── Public Subnet AZ-1: 10.0.1.0/24 (ALB, NAT)
├── Public Subnet AZ-2: 10.0.2.0/24 (ALB, NAT)
├── Private Subnet AZ-1: 10.0.3.0/24 (Nodes)
└── Private Subnet AZ-2: 10.0.4.0/24 (Nodes)
```

**Key Points:**
- Multi-AZ deployment
- ALB spans multiple AZs
- NAT Gateway in each AZ
- Worker nodes in multiple AZs
- Pods distributed across AZs

### Scenario 2: IP Address Exhaustion

**Problem:** Running out of IP addresses for pods

**Solutions:**
1. **Increase subnet size:**
   - Use larger CIDR (e.g., /22 instead of /24)

2. **Use multiple subnets:**
   - Configure VPC CNI to use multiple subnets
   - Pods can get IPs from any configured subnet

3. **Use Fargate:**
   - Fargate manages IP allocation
   - No need to manage ENI limits

4. **Optimize IP usage:**
   - Reduce WARM_IP_TARGET
   - Use smaller instance types with fewer pods

### Scenario 3: Network Policy with VPC CNI

**How it works:**
- Network Policies are enforced by VPC CNI
- Uses iptables rules on nodes
- Works with VPC IPs (no overlay)

**Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

**Enforcement:**
- VPC CNI creates iptables rules
- Rules filter based on pod IPs
- Works seamlessly with VPC networking

---

## Interview Scenarios

### Scenario 1: "Explain how a user request reaches a pod in EKS"

**Answer:**

```
Complete Flow:

1. User accesses myapp.com
   ↓
2. DNS (Route 53) resolves to ALB DNS name
   ↓
3. User's browser sends HTTP request to ALB
   ↓
4. Traffic enters AWS VPC via Internet Gateway
   ↓
5. VPC route table routes to ALB (in public subnet)
   ↓
6. ALB receives request (Layer 7)
   ↓
7. ALB checks Ingress rules (host, path)
   ↓
8. ALB routes to target group (pod IPs directly)
   ↓
9. ALB sends request to pod IP (e.g., 10.0.3.50)
   ↓
10. VPC routing delivers to private subnet (10.0.3.0/24)
    ↓
11. Traffic reaches worker node's ENI
    ↓
12. VPC CNI routes to pod (secondary IP on ENI)
    ↓
13. Pod receives request and processes it
    ↓
14. Response flows back: Pod → ENI → VPC → ALB → IGW → User

Key Components:
- Internet Gateway: Entry point to VPC
- ALB: Layer 7 load balancer
- VPC Routing: Routes between subnets
- VPC CNI: Assigns VPC IPs to pods
- ENI: Network interface with secondary IPs
```

### Scenario 2: "How does pod-to-pod communication work in EKS?"

**Answer:**

```
EKS uses AWS VPC CNI, which assigns real VPC IPs to pods.

Same Node:
- Pod A and Pod B on same node
- Both have secondary IPs on same ENI
- Linux kernel routes directly (no network hop)
- Very low latency

Different Nodes:
- Pod A on Node 1 (10.0.3.50)
- Pod B on Node 2 (10.0.4.50)
- Pod A sends packet to 10.0.4.50
- VPC route table routes to Node 2's subnet
- VPC delivers to Node 2's ENI
- ENI delivers to Pod B (secondary IP)

Key Points:
- No overlay network (unlike Calico/Flannel)
- Direct VPC IP communication
- VPC handles routing
- Better performance (no encapsulation)
- Limited by ENI/IP capacity
```

### Scenario 3: "What's the difference between ALB IP mode and NodePort mode?"

**Answer:**

**ALB IP Mode (Recommended):**
```
Internet → ALB → Pod IPs (direct)
- ALB targets pod IPs directly
- No kube-proxy involved
- Better performance
- Lower latency
- Direct routing
```

**NodePort Mode:**
```
Internet → ALB → Node IPs (NodePort) → kube-proxy → Pod IPs
- ALB targets node IPs
- kube-proxy handles routing
- More hops
- Higher latency
- Uses NodePort (30000-32767)
```

**When to use:**
- IP mode: Production, better performance
- NodePort mode: Legacy, or when IP mode not available

### Scenario 4: "How do you troubleshoot pod networking issues in EKS?"

**Troubleshooting Steps:**

1. **Check Pod Status:**
   ```bash
   kubectl get pods -o wide
   # Verify pod has IP address
   ```

2. **Check VPC CNI:**
   ```bash
   kubectl get pods -n kube-system | grep vpc-cni
   kubectl logs -n kube-system <vpc-cni-pod>
   ```

3. **Check ENI/IP Allocation:**
   ```bash
   # On node, check ENI
   aws ec2 describe-network-interfaces --filters "Name=attachment.instance-id,Values=<instance-id>"
   ```

4. **Check VPC Routing:**
   ```bash
   # On node
   ip route
   # Verify routes to other subnets
   ```

5. **Check Security Groups:**
   ```bash
   # Verify security groups allow traffic
   aws ec2 describe-security-groups
   ```

6. **Check Network Policies:**
   ```bash
   kubectl get networkpolicies
   # Network policies might be blocking
   ```

7. **Test Connectivity:**
   ```bash
   kubectl exec -it <pod> -- ping <other-pod-ip>
   kubectl exec -it <pod> -- curl <service-name>
   ```

8. **Check IP Address Availability:**
   ```bash
   # Check subnet IP usage
   aws ec2 describe-subnets --subnet-ids <subnet-id>
   ```

### Scenario 5: "How does EKS integrate with AWS services?"

**Answer:**

**VPC Integration:**
- EKS cluster runs in your VPC
- Pods get VPC IPs
- Direct VPC communication

**Load Balancer Integration:**
- Kubernetes services create AWS ELB
- ALB/NLB provisioned automatically
- Integrated with target groups

**IAM Integration:**
- IAM roles for service accounts (IRSA)
- Pods can assume IAM roles
- No need for access keys

**Security Groups:**
- Pods use EC2 security groups
- Can reference security groups in Network Policies
- Integrated with VPC security

**Route 53:**
- External DNS controller
- Automatically creates DNS records
- Integrates with Ingress

**CloudWatch:**
- Container Insights
- Logs aggregation
- Metrics collection

---

## Best Practices

1. **Use private subnets** for worker nodes
2. **Use ALB IP mode** for Ingress (better performance)
3. **Plan IP addresses** carefully (pods use VPC IPs)
4. **Use multiple AZs** for high availability
5. **Use NAT Gateway** in each AZ
6. **Enable VPC Flow Logs** for monitoring
7. **Use Network Policies** for pod-level security
8. **Use IAM roles for service accounts** (IRSA)
9. **Monitor ENI/IP usage** to avoid exhaustion
10. **Use Fargate** if IP management is concern

---

## Summary

**Key Takeaways:**
- EKS runs in your VPC
- Pods get real VPC IPs (no overlay)
- AWS VPC CNI assigns IPs from VPC subnets
- ALB can target pod IPs directly (IP mode)
- VPC routing handles pod-to-pod communication
- Security Groups and Network Policies provide security
- Plan IP addresses carefully (pods use VPC IPs)

**Remember:**
- No overlay network (unlike other CNIs)
- Direct VPC integration
- Better performance
- Limited by ENI/IP capacity
- Must plan IP addresses

