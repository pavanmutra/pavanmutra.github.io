# Networking Guide: AWS, Kubernetes, and Azure

## Complete Networking Documentation for DevOps Engineers

This comprehensive guide covers networking fundamentals, traffic flows, and architecture patterns for AWS, Kubernetes, and Azure. Designed to help you understand end-to-end networking scenarios and ace your DevOps interviews.

---

## 📚 Documentation Index

### 1. [AWS Networking Guide](./01-aws-networking.md)
**Complete guide to AWS networking fundamentals**

**Topics Covered:**
- VPC (Virtual Private Cloud) architecture
- Subnets and routing (public vs private)
- Internet Gateway and NAT Gateway
- Security Groups and NACLs (Network ACLs)
- Load Balancers (ALB, NLB, CLB)
- Traffic flow scenarios
- Common interview scenarios with detailed explanations

**Key Concepts:**
- How VPCs isolate your network
- Public vs private subnet routing
- Stateful vs stateless firewalls
- Layer 4 vs Layer 7 load balancing
- End-to-end traffic flows from internet to applications

---

### 2. [Kubernetes Networking Guide](./02-kubernetes-networking.md)
**Complete guide to Kubernetes networking**

**Topics Covered:**
- Pod networking and IP assignment
- Services (ClusterIP, NodePort, LoadBalancer, ExternalName)
- Ingress Controllers
- CNI Plugins (Calico, Flannel, Weave, AWS VPC CNI)
- Network Policies
- DNS in Kubernetes (CoreDNS)
- Traffic flow scenarios
- Common interview scenarios

**Key Concepts:**
- How pods get IP addresses
- Service discovery and DNS
- kube-proxy and iptables/IPVS
- Ingress vs LoadBalancer
- Network Policies for pod-level security
- Pod-to-pod communication (same node vs different nodes)

---

### 3. [AWS + Kubernetes Integration](./03-aws-kubernetes-integration.md)
**End-to-end guide for EKS networking**

**Topics Covered:**
- EKS architecture and VPC integration
- AWS VPC CNI plugin (how pods get VPC IPs)
- Load Balancer integration (ALB, NLB)
- End-to-end traffic flows
- Common scenarios and troubleshooting
- Interview scenarios

**Key Concepts:**
- How EKS integrates with AWS VPC
- Direct VPC IP assignment (no overlay network)
- ALB IP mode vs NodePort mode
- Pod-to-pod communication in EKS
- Troubleshooting pod networking issues

---

### 4. [Azure Networking Guide](./04-azure-networking.md)
**Complete guide to Azure networking**

**Topics Covered:**
- Virtual Network (VNet) architecture
- Subnets and routing
- Network Security Groups (NSGs)
- Load Balancers (Azure Load Balancer, Application Gateway)
- AKS (Azure Kubernetes Service) networking
- Traffic flow scenarios
- Common interview scenarios

**Key Concepts:**
- VNet vs AWS VPC comparison
- NSGs vs Security Groups
- Application Gateway vs Load Balancer
- AKS networking models (kubenet vs Azure CNI)
- Application Security Groups (ASGs)

---

## 🎯 How to Use This Guide

### For Learning
1. Start with **AWS Networking** to understand cloud networking fundamentals
2. Move to **Kubernetes Networking** to understand container networking
3. Study **AWS + Kubernetes Integration** to see how they work together
4. Review **Azure Networking** for comparison and Azure-specific concepts

### For Interview Preparation
1. Read each guide thoroughly
2. Focus on **Traffic Flow Scenarios** sections
3. Study **Common Interview Scenarios** in each guide
4. Practice explaining traffic flows out loud
5. Draw diagrams while explaining (helps with visualization)

### For Troubleshooting
1. Use **Traffic Flow Scenarios** to understand expected behavior
2. Check **Troubleshooting Steps** in interview scenarios
3. Review **Best Practices** sections
4. Understand each component's role in the flow

---

## 🔍 Quick Reference

### AWS Networking Components
- **VPC:** Isolated network (like your private data center)
- **Subnet:** Logical division of VPC (one per AZ)
- **IGW:** Internet Gateway (two-way internet access)
- **NAT Gateway:** Outbound-only internet access for private subnets
- **Security Group:** Stateful firewall (instance level)
- **NACL:** Stateless firewall (subnet level)
- **ALB:** Application Load Balancer (Layer 7)
- **NLB:** Network Load Balancer (Layer 4)

### Kubernetes Networking Components
- **Pod:** Smallest unit (gets its own IP)
- **Service:** Stable endpoint for pods (ClusterIP, NodePort, LoadBalancer)
- **Ingress:** HTTP/HTTPS routing (Layer 7)
- **CNI:** Container Network Interface (assigns pod IPs)
- **kube-proxy:** Routes service traffic (iptables/IPVS)
- **Network Policy:** Firewall rules for pods
- **CoreDNS:** DNS for service discovery

### Azure Networking Components
- **VNet:** Virtual Network (like AWS VPC)
- **Subnet:** Logical division of VNet
- **NSG:** Network Security Group (stateful firewall)
- **Application Gateway:** Layer 7 load balancer
- **Azure Load Balancer:** Layer 4 load balancer
- **ASG:** Application Security Group (logical grouping)

---

## 📊 Comparison Tables

### AWS vs Azure Networking

| Component | AWS | Azure |
|-----------|-----|-------|
| Isolated Network | VPC | VNet |
| Firewall (Instance) | Security Group | NSG (on NIC) |
| Firewall (Subnet) | NACL | NSG (on Subnet) |
| Layer 7 LB | ALB | Application Gateway |
| Layer 4 LB | NLB | Azure Load Balancer |
| Outbound NAT | NAT Gateway | NAT Gateway |
| Internet Access | IGW | Public IP / IGW |

### Kubernetes Service Types

| Type | Use Case | Access |
|------|----------|--------|
| ClusterIP | Internal services | Within cluster only |
| NodePort | Development/testing | NodeIP:Port |
| LoadBalancer | Production | Cloud LB IP/DNS |
| Ingress | HTTP/HTTPS apps | URL-based routing |

---

## 🎓 Interview Preparation Tips

### Common Question Types

1. **"Explain how traffic flows from X to Y"**
   - Start from the source
   - Follow each hop
   - Mention each component (DNS, Load Balancer, Route Tables, etc.)
   - Explain what happens at each step

2. **"What's the difference between X and Y?"**
   - Compare features side-by-side
   - Give use cases for each
   - Mention when to use which

3. **"How would you design X?"**
   - Start with high-level architecture
   - Break down into components
   - Explain traffic flow
   - Mention high availability, security, scalability

4. **"Troubleshoot: X is not working"**
   - Start with the most common issues
   - Check each component in the flow
   - Use systematic approach
   - Mention tools/commands to use

### Key Concepts to Master

1. **Stateful vs Stateless Firewalls**
   - Security Groups (AWS) / NSGs (Azure) = Stateful
   - NACLs (AWS) = Stateless
   - Understand return traffic handling

2. **Public vs Private Subnets**
   - Public: Route to IGW (bidirectional)
   - Private: Route to NAT (outbound only)
   - Know the routing differences

3. **Layer 4 vs Layer 7 Load Balancing**
   - Layer 4: TCP/UDP, port-based
   - Layer 7: HTTP/HTTPS, content-based
   - Know when to use which

4. **Pod Networking**
   - How pods get IPs (CNI)
   - Pod-to-pod communication
   - Service discovery (DNS)
   - Network Policies

---

## 🚀 Real-World Scenarios

### Scenario 1: Three-Tier Web Application
**Architecture:**
- Load Balancer (public subnet)
- Web Tier (private subnet)
- App Tier (private subnet)
- Database (private subnet)

**Traffic Flow:**
Internet → Load Balancer → Web → App → Database

**Security:**
- Each tier isolated in separate subnet
- Security Groups/NSGs control access
- Database never directly accessible

### Scenario 2: Microservices on Kubernetes
**Architecture:**
- Ingress Controller
- Multiple Services (ClusterIP)
- Pods in different namespaces
- Network Policies for isolation

**Traffic Flow:**
Internet → Ingress → Service → Pod

**Key Points:**
- Service discovery via DNS
- Network Policies for security
- Ingress for external access

### Scenario 3: Hybrid Cloud
**Architecture:**
- On-premises datacenter
- AWS/Azure cloud
- VPN or ExpressRoute connection

**Traffic Flow:**
On-prem → VPN Gateway → VPC/VNet → Cloud Resources

**Key Points:**
- Route tables for hybrid routing
- Security Groups/NSGs for access control
- VPN/ExpressRoute for connectivity

---

## 📝 Study Checklist

### AWS Networking
- [ ] Understand VPC and subnet concepts
- [ ] Know difference between IGW and NAT Gateway
- [ ] Understand Security Groups vs NACLs
- [ ] Know ALB vs NLB differences
- [ ] Can explain traffic flow from internet to application
- [ ] Understand VPC peering
- [ ] Know how to troubleshoot networking issues

### Kubernetes Networking
- [ ] Understand how pods get IPs
- [ ] Know all service types and when to use them
- [ ] Understand Ingress vs LoadBalancer
- [ ] Know how kube-proxy works
- [ ] Understand Network Policies
- [ ] Know how service discovery works (DNS)
- [ ] Can explain pod-to-pod communication

### AWS + Kubernetes (EKS)
- [ ] Understand EKS architecture
- [ ] Know how AWS VPC CNI works
- [ ] Understand ALB IP mode vs NodePort mode
- [ ] Know how pods get VPC IPs
- [ ] Understand pod-to-pod communication in EKS
- [ ] Can troubleshoot EKS networking issues

### Azure Networking
- [ ] Understand VNet concepts
- [ ] Know NSGs and ASGs
- [ ] Understand Application Gateway vs Load Balancer
- [ ] Know AKS networking models
- [ ] Understand Azure CNI vs kubenet
- [ ] Can explain traffic flows in Azure

---

## 🔗 Additional Resources

### Official Documentation
- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [Kubernetes Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- [Azure Virtual Network](https://docs.microsoft.com/azure/virtual-network/)

### Practice
- Set up a test VPC/VNet
- Deploy a simple application
- Trace traffic flows
- Practice troubleshooting scenarios

---

## 💡 Tips for Success

1. **Draw Diagrams:** Visualize traffic flows
2. **Explain Out Loud:** Practice explaining concepts
3. **Start Simple:** Understand basics before advanced topics
4. **Follow the Flow:** Always trace traffic from source to destination
5. **Think Security:** Consider security at each layer
6. **Know the Tools:** Understand commands for troubleshooting
7. **Practice Scenarios:** Work through different scenarios
8. **Compare Platforms:** Understand similarities and differences

---

## 📞 Common Interview Questions

1. "Explain how a user request reaches a pod in EKS"
2. "What's the difference between Security Groups and NACLs?"
3. "How does pod-to-pod communication work?"
4. "Explain the difference between ALB and NLB"
5. "How do you troubleshoot networking issues?"
6. "Design a highly available web application"
7. "How does service discovery work in Kubernetes?"
8. "Explain Ingress vs LoadBalancer"
9. "How does VPC peering work?"
10. "What's the difference between Azure CNI and kubenet?"

**All these questions are answered in detail in the respective guides!**

---

## 🎯 Next Steps

1. **Read each guide** thoroughly
2. **Practice explaining** concepts out loud
3. **Draw diagrams** for traffic flows
4. **Set up test environments** to see it in action
5. **Review interview scenarios** multiple times
6. **Practice troubleshooting** scenarios

---

**Good luck with your DevOps journey! 🚀**

*This guide is designed to help you understand networking concepts deeply, not just memorize facts. Focus on understanding the "why" behind each concept, and you'll be able to handle any scenario-based question.*

