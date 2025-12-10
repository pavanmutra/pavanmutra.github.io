# Kubernetes Networking: Complete Guide for DevOps Engineers

## Table of Contents
1. [Core Concepts](#core-concepts)
2. [Pod Networking](#pod-networking)
3. [Service Networking](#service-networking)
4. [Ingress Controllers](#ingress-controllers)
5. [CNI Plugins](#cni-plugins)
6. [Network Policies](#network-policies)
7. [DNS in Kubernetes](#dns-in-kubernetes)
8. [Traffic Flow Scenarios](#traffic-flow-scenarios)
9. [Common Interview Scenarios](#common-interview-scenarios)

---

## Core Concepts

### What is Kubernetes Networking?

Kubernetes networking is how containers (pods) communicate with each other and with external services. Unlike traditional networking, Kubernetes has specific requirements:

**Kubernetes Networking Model:**
1. **Every pod gets its own IP address** (no NAT between pods)
2. **Pods can communicate without NAT** (flat network)
3. **Nodes can communicate with pods** without NAT
4. **Pods see their own IP** as others see it

### Key Components

- **Pods:** Smallest deployable unit (one or more containers)
- **Services:** Stable network endpoint for pods
- **Ingress:** HTTP/HTTPS routing from outside
- **CNI:** Container Network Interface (plugin for networking)
- **Network Policies:** Firewall rules for pods

---

## Pod Networking

### What is a Pod?

A **Pod** is the smallest unit in Kubernetes. It contains:
- One or more containers
- Shared network namespace (containers share IP)
- Shared storage volumes

### Pod IP Address

**Key Points:**
- Each pod gets a unique IP from the pod CIDR
- IP is assigned by CNI plugin
- Pods can communicate directly using pod IPs
- IP is ephemeral (changes when pod restarts)

**Example:**
```
Pod: web-app-abc123
IP: 10.244.1.5
Namespace: default
```

### How Pods Get IPs

```
1. Pod is scheduled to a node
   ↓
2. Kubelet (on node) calls CNI plugin
   ↓
3. CNI plugin assigns IP from pod CIDR
   ↓
4. IP is configured on pod's network interface
   ↓
5. Routes are added to node's routing table
   ↓
6. Pod can now communicate
```

### Pod-to-Pod Communication

**Same Node:**
```
Pod A (10.244.1.5) → Bridge Network → Pod B (10.244.1.6)
```

**Different Nodes:**
```
Pod A (Node 1, 10.244.1.5)
   ↓
Route Table (10.244.1.0/24 → Node 1)
   ↓
Overlay Network / VPC Routing
   ↓
Route Table (10.244.2.0/24 → Node 2)
   ↓
Pod B (Node 2, 10.244.2.5)
```

**Key Point:** Kubernetes ensures pods can communicate regardless of which node they're on.

---

## Service Networking

### What is a Service?

A **Service** is a stable network endpoint that abstracts pods. It provides:
- **Stable IP address** (doesn't change)
- **Stable DNS name** (service-name.namespace.svc.cluster.local)
- **Load balancing** across pod endpoints

### Why Services are Needed

**Problem:** Pods are ephemeral
- Pod IPs change when pods restart
- Pods are created/destroyed dynamically
- How do you find pods?

**Solution:** Services provide stable endpoint

### Service Types

#### 1. ClusterIP (Default)

**Purpose:** Internal access only (within cluster)

**How it works:**
```
Pod → Service ClusterIP → Endpoints (Pods)
```

**Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

**Traffic Flow:**
```
1. Pod wants to access web-service
   ↓
2. DNS resolves web-service to ClusterIP (e.g., 10.96.1.5)
   ↓
3. Traffic goes to ClusterIP
   ↓
4. kube-proxy (on each node) intercepts
   ↓
5. kube-proxy uses iptables/ipvs to route to pod IP
   ↓
6. Traffic reaches actual pod
```

#### 2. NodePort

**Purpose:** Expose service on each node's IP at a static port

**How it works:**
```
Internet → NodeIP:NodePort → Service → Pods
```

**Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080  # Optional (30000-32767)
```

**Traffic Flow:**
```
1. User accesses node-ip:30080
   ↓
2. Traffic hits node's network interface
   ↓
3. kube-proxy intercepts on port 30080
   ↓
4. Routes to Service ClusterIP
   ↓
5. Service routes to pod endpoints
```

**Key Points:**
- Port range: 30000-32767
- Accessible from outside cluster
- Not recommended for production (use LoadBalancer/Ingress)

#### 3. LoadBalancer

**Purpose:** Expose service using cloud provider's load balancer

**How it works:**
```
Internet → Cloud Load Balancer → Service → Pods
```

**Example:**
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

**Traffic Flow (AWS):**
```
1. User accesses LoadBalancer DNS/IP
   ↓
2. AWS ELB receives traffic
   ↓
3. ELB routes to healthy nodes
   ↓
4. Node receives traffic on NodePort
   ↓
5. kube-proxy routes to Service
   ↓
6. Service routes to pods
```

**Key Points:**
- Integrates with cloud provider (AWS ELB, GCP LB, Azure LB)
- Automatically provisions load balancer
- Best for production

#### 4. ExternalName

**Purpose:** Maps service to external DNS name

**Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
```

**Use Case:** Access external services as if they were internal

### Service Discovery

**DNS-based:**
- Service name: `web-service`
- Full DNS: `web-service.default.svc.cluster.local`
- Short form: `web-service` (same namespace)
- Cross-namespace: `web-service.production.svc.cluster.local`

**Environment Variables:**
- Kubernetes injects service environment variables into pods
- Format: `SERVICE_NAME_SERVICE_HOST`, `SERVICE_NAME_SERVICE_PORT`

### kube-proxy Modes

#### iptables Mode (Default)

**How it works:**
1. kube-proxy watches API server for services/endpoints
2. Creates iptables rules on each node
3. Traffic to ClusterIP is DNAT'd to pod IP

**Example iptables rule:**
```
-A KUBE-SERVICES -d 10.96.1.5/32 -p tcp -m tcp --dport 80 -j KUBE-SVC-XXXXX
-A KUBE-SVC-XXXXX -m statistic --mode random --probability 0.5 -j KUBE-SEP-AAAAA
-A KUBE-SVC-XXXXX -j KUBE-SEP-BBBBB
```

**Pros:**
- Fast (kernel-level)
- No userspace overhead
- Better performance

**Cons:**
- More iptables rules (can be thousands)
- Harder to debug

#### IPVS Mode

**How it works:**
1. Uses Linux IPVS (IP Virtual Server)
2. More efficient load balancing algorithms
3. Better for large clusters

**Pros:**
- Better performance at scale
- More load balancing algorithms
- Lower latency

**Cons:**
- Requires IPVS kernel modules
- Slightly more complex

---

## Ingress Controllers

### What is Ingress?

**Ingress** is an API object that manages external HTTP/HTTPS access to services. It provides:
- **URL-based routing** (path-based, host-based)
- **SSL/TLS termination**
- **Load balancing**

### Ingress vs LoadBalancer

| Feature | Ingress | LoadBalancer |
|---------|---------|--------------|
| Layer | Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP) |
| Routing | Path/host-based | Port-based |
| SSL | Built-in termination | Manual setup |
| Cost | One LB for many services | One LB per service |
| Use Case | Web applications | Any TCP/UDP service |

### Ingress Controller

**Ingress Controller** is the actual implementation that processes Ingress resources. Popular options:
- **NGINX Ingress Controller** (most popular)
- **Traefik**
- **AWS ALB Ingress Controller**
- **Istio Gateway**

### How Ingress Works

```
Internet → Ingress Controller (Pod) → Service → Pods
```

**Traffic Flow:**
```
1. User accesses myapp.com/api
   ↓
2. DNS resolves to Ingress Controller's LoadBalancer IP
   ↓
3. Ingress Controller (NGINX) receives request
   ↓
4. Ingress Controller checks Ingress rules
   ↓
5. Matches path /api to backend service
   ↓
6. Routes to Service ClusterIP
   ↓
7. Service routes to pod endpoints
```

### Ingress Example

**Ingress Resource:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
  tls:
  - hosts:
    - myapp.com
    secretName: tls-secret
```

**Key Points:**
- One Ingress can route to multiple services
- Path-based routing
- Host-based routing
- SSL/TLS termination

---

## CNI Plugins

### What is CNI?

**CNI (Container Network Interface)** is a specification for configuring network interfaces for containers. Kubernetes uses CNI plugins to:
- Assign IP addresses to pods
- Configure network interfaces
- Set up routing

### Popular CNI Plugins

#### 1. Calico

**Features:**
- Network policies
- BGP routing
- IP-in-IP or VXLAN encapsulation
- Works on any platform

**How it works:**
- Uses BGP to advertise routes
- Each node knows routes to all pods
- Direct pod-to-pod communication

#### 2. Flannel

**Features:**
- Simple overlay network
- VXLAN or host-gw mode
- Easy to set up

**How it works:**
- Creates overlay network
- Allocates subnet per node
- Encapsulates traffic in VXLAN

#### 3. Weave Net

**Features:**
- Automatic network discovery
- Encryption support
- Network policies

#### 4. AWS VPC CNI (for EKS)

**Features:**
- Uses AWS VPC directly (no overlay)
- Pods get real VPC IPs
- Best performance on AWS

**How it works:**
- Allocates IPs from VPC subnet
- No encapsulation overhead
- Integrates with AWS networking

### CNI Plugin Selection

**Choose based on:**
- **Cloud provider:** Use cloud-specific CNI (AWS VPC CNI, Azure CNI)
- **On-premises:** Calico, Flannel, Weave
- **Network policies needed:** Calico, Weave
- **Performance:** AWS VPC CNI (no overlay)

---

## Network Policies

### What are Network Policies?

**Network Policies** are Kubernetes firewall rules. They control:
- Which pods can communicate
- Which ports are allowed
- Ingress and egress rules

### Why Network Policies?

**Default:** All pods can communicate with all pods (no isolation)

**With Network Policies:** Fine-grained control over pod communication

### Network Policy Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow-frontend
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

**What this does:**
- Applies to pods with label `app: api`
- Allows ingress only from pods with label `app: frontend`
- Only on port 8080
- All other traffic is denied

### Network Policy Rules

**Ingress Rules:**
- Control incoming traffic to pods
- Can specify: podSelector, namespaceSelector, ipBlock

**Egress Rules:**
- Control outgoing traffic from pods
- Can specify: podSelector, namespaceSelector, ipBlock

**Default Deny:**
- If NetworkPolicy exists, default is deny all
- Must explicitly allow traffic

### Network Policy Scenarios

**Scenario 1: Isolate Namespaces**
```yaml
# Deny all ingress to production namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  # No ingress rules = deny all
```

**Scenario 2: Allow Specific Service**
```yaml
# Allow only database access from app pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-allow-app
spec:
  podSelector:
    matchLabels:
      app: database
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: app
    ports:
    - protocol: TCP
      port: 5432
```

---

## DNS in Kubernetes

### CoreDNS

**CoreDNS** is the default DNS server in Kubernetes. It provides:
- Service discovery
- Pod DNS resolution
- External DNS resolution

### DNS Records

**Service Records:**
- `web-service.default.svc.cluster.local` → Service ClusterIP
- `web-service` → Service ClusterIP (short form)

**Pod Records:**
- `10-244-1-5.default.pod.cluster.local` → Pod IP
- Format: `<pod-ip-with-dashes>.<namespace>.pod.cluster.local`

### DNS Resolution Flow

```
1. Pod wants to access web-service
   ↓
2. Pod queries CoreDNS (10.96.0.10)
   ↓
3. CoreDNS checks service records
   ↓
4. Returns ClusterIP (10.96.1.5)
   ↓
5. Pod uses ClusterIP to access service
```

### DNS Configuration

**CoreDNS ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
        }
        forward . /etc/resolv.conf
        cache 30
    }
```

---

## Traffic Flow Scenarios

### Scenario 1: User Accesses Web Application

**Architecture:**
- Ingress Controller (NGINX)
- Web Service (ClusterIP)
- Web Pods

**Traffic Flow:**

```
1. User accesses myapp.com
   ↓
2. DNS resolves to Ingress Controller LoadBalancer IP
   ↓
3. Ingress Controller receives request
   ↓
4. Ingress Controller checks Ingress rules
   ↓
5. Routes to web-service (ClusterIP: 10.96.1.5)
   ↓
6. kube-proxy (iptables) intercepts ClusterIP traffic
   ↓
7. iptables DNAT to pod IP (10.244.1.5)
   ↓
8. Traffic reaches web pod
   ↓
9. Web pod processes request
   ↓
10. Response flows back through same path
```

### Scenario 2: Pod-to-Pod Communication

**Architecture:**
- Frontend Pod (10.244.1.5)
- Backend Service (ClusterIP: 10.96.2.5)
- Backend Pods (10.244.2.5, 10.244.3.5)

**Traffic Flow:**

```
1. Frontend pod wants to call backend-service
   ↓
2. Frontend pod queries CoreDNS for backend-service
   ↓
3. CoreDNS returns ClusterIP (10.96.2.5)
   ↓
4. Frontend pod sends request to 10.96.2.5:8080
   ↓
5. kube-proxy intercepts (iptables rule)
   ↓
6. iptables load balances to backend pod (10.244.2.5)
   ↓
7. Traffic routed through CNI (Calico/Flannel)
   ↓
8. If different node: overlay network (VXLAN)
   ↓
9. If same node: bridge network
   ↓
10. Backend pod receives request
    ↓
11. Response flows back through same path
```

### Scenario 3: Pod Accesses External Service

**Architecture:**
- Pod in cluster
- External API (api.example.com)

**Traffic Flow:**

```
1. Pod wants to access api.example.com
   ↓
2. Pod queries CoreDNS
   ↓
3. CoreDNS doesn't find service, forwards to upstream DNS
   ↓
4. Upstream DNS resolves to external IP
   ↓
5. Pod sends request to external IP
   ↓
6. Traffic goes through node's network interface
   ↓
7. If in cloud: through VPC routing
   ↓
8. If on-prem: through physical network
   ↓
9. External service receives request
   ↓
10. Response flows back
```

### Scenario 4: External Service Accesses Pod

**Architecture:**
- External user
- LoadBalancer Service
- Pods

**Traffic Flow (AWS EKS):**

```
1. External user accesses LoadBalancer DNS
   ↓
2. AWS ELB receives traffic
   ↓
3. ELB health checks nodes
   ↓
4. ELB routes to healthy node (NodePort)
   ↓
5. Node receives traffic on NodePort (e.g., 30080)
   ↓
6. kube-proxy intercepts NodePort traffic
   ↓
7. Routes to Service ClusterIP
   ↓
8. iptables routes to pod IP
   ↓
9. CNI routes to pod (if different node)
   ↓
10. Pod receives request
```

---

## Common Interview Scenarios

### Scenario 1: "How does a pod get its IP address?"

**Answer:**

```
1. Pod is scheduled to a node by scheduler
   ↓
2. Kubelet (on node) receives pod spec
   ↓
3. Kubelet calls CNI plugin (configured in /etc/cni/net.d)
   ↓
4. CNI plugin:
   - Allocates IP from pod CIDR (e.g., 10.244.1.0/24)
   - Creates network interface (veth pair)
   - Attaches one end to pod, other to bridge
   - Configures IP on pod's interface
   - Adds routes to node's routing table
   ↓
5. Pod now has IP and can communicate

Example:
- Pod CIDR: 10.244.1.0/24
- Pod gets: 10.244.1.5
- CNI: Calico/Flannel/AWS VPC CNI
```

### Scenario 2: "Explain the difference between ClusterIP, NodePort, and LoadBalancer"

**Answer:**

**ClusterIP:**
- Internal only (within cluster)
- Stable IP (10.96.x.x range)
- Accessed via DNS or ClusterIP
- Use case: Internal services

**NodePort:**
- Exposes service on each node's IP
- Port range: 30000-32767
- Accessible from outside cluster
- Use case: Development, testing
- Not recommended for production

**LoadBalancer:**
- Cloud provider load balancer
- External IP/DNS provided
- Best for production
- Use case: Public-facing services
- Integrates with AWS ELB, GCP LB, Azure LB

**Comparison:**
```
ClusterIP:    Pod → Service (internal)
NodePort:     Internet → NodeIP:Port → Service → Pod
LoadBalancer: Internet → Cloud LB → Service → Pod
```

### Scenario 3: "How does service discovery work in Kubernetes?"

**Answer:**

**DNS-based (Primary method):**
1. Every service gets DNS name
2. Format: `service-name.namespace.svc.cluster.local`
3. Short form: `service-name` (same namespace)
4. CoreDNS resolves service names to ClusterIPs

**Example:**
```
Pod queries: web-service
CoreDNS returns: 10.96.1.5 (ClusterIP)
Pod uses ClusterIP to access service
```

**Environment Variables (Legacy):**
- Kubernetes injects service env vars
- Format: `SERVICE_NAME_SERVICE_HOST`, `SERVICE_NAME_SERVICE_PORT`
- Not recommended (DNS is better)

**How it works:**
```
1. Service created with name "web-service"
2. CoreDNS watches API server
3. CoreDNS creates DNS record: web-service → 10.96.1.5
4. Pod queries CoreDNS (10.96.0.10)
5. CoreDNS returns ClusterIP
6. Pod uses ClusterIP
```

### Scenario 4: "How does Ingress work? Explain the flow"

**Answer:**

**Components:**
1. Ingress Resource (YAML definition)
2. Ingress Controller (actual implementation, e.g., NGINX)

**Traffic Flow:**
```
1. User accesses myapp.com/api
   ↓
2. DNS resolves to Ingress Controller's LoadBalancer IP
   ↓
3. Ingress Controller (NGINX pod) receives request
   ↓
4. Ingress Controller reads Ingress resources from API server
   ↓
5. Matches host (myapp.com) and path (/api)
   ↓
6. Routes to backend service (api-service)
   ↓
7. Service routes to pod endpoints
   ↓
8. Pod processes request
```

**Key Points:**
- Ingress Controller is a pod (needs to be deployed)
- Ingress resource defines routing rules
- One Ingress Controller can handle multiple Ingress resources
- SSL termination happens at Ingress Controller

### Scenario 5: "How do Network Policies work? Give an example"

**Answer:**

**What they do:**
- Firewall rules for pods
- Control ingress and egress traffic
- Default: allow all (if no NetworkPolicy)
- If NetworkPolicy exists: default deny

**Example:**
```yaml
# Allow only frontend to access API
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

**How it works:**
1. NetworkPolicy applies to pods with label `app: api`
2. Allows ingress only from pods with label `app: frontend`
3. Only on port 8080
4. All other traffic denied

**Evaluation:**
- Network Policies are evaluated by CNI plugin
- If no NetworkPolicy: allow all
- If NetworkPolicy exists: deny by default, allow only if rule matches

### Scenario 6: "Troubleshoot: Pod can't reach another pod"

**Troubleshooting Steps:**

1. **Check Pod Status**
   ```bash
   kubectl get pods
   kubectl describe pod <pod-name>
   ```

2. **Check Pod IPs**
   ```bash
   kubectl get pods -o wide
   # Verify both pods have IPs
   ```

3. **Check Service (if using service)**
   ```bash
   kubectl get svc
   kubectl describe svc <service-name>
   kubectl get endpoints <service-name>
   ```

4. **Check Network Policies**
   ```bash
   kubectl get networkpolicies
   # NetworkPolicy might be blocking traffic
   ```

5. **Check DNS**
   ```bash
   kubectl exec -it <pod> -- nslookup <service-name>
   ```

6. **Check CNI Plugin**
   ```bash
   # Check CNI plugin logs
   kubectl logs -n kube-system <cni-pod>
   ```

7. **Check Node Networking**
   ```bash
   # On node, check routes
   ip route
   # Check iptables (if using iptables mode)
   iptables -t nat -L
   ```

8. **Test Connectivity**
   ```bash
   kubectl exec -it <pod1> -- ping <pod2-ip>
   kubectl exec -it <pod1> -- curl <service-name>
   ```

---

## Best Practices

1. **Use Services** instead of pod IPs (pods are ephemeral)
2. **Use Ingress** for HTTP/HTTPS traffic (not NodePort)
3. **Use Network Policies** for security (defense in depth)
4. **Choose right CNI** for your environment
5. **Use DNS** for service discovery (not IPs)
6. **Monitor network policies** (can break connectivity)
7. **Use LoadBalancer** for production (not NodePort)
8. **Plan pod CIDR** carefully (avoid overlaps)
9. **Enable network policies** gradually (test thoroughly)
10. **Use labels** consistently (for selectors)

---

## Summary

**Key Takeaways:**
- Every pod gets its own IP address
- Services provide stable endpoints for pods
- Ingress provides HTTP/HTTPS routing
- CNI plugins handle pod networking
- Network Policies provide firewall rules
- CoreDNS provides service discovery
- kube-proxy handles service routing (iptables/IPVS)

**Remember:**
- Pod IPs are ephemeral → use Services
- Services provide stable ClusterIPs
- Ingress is for HTTP/HTTPS routing
- Network Policies are deny-by-default (if exists)
- CNI plugin assigns pod IPs
- DNS is primary service discovery method

