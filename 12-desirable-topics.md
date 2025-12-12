# Desirable Topics - Interview Q&A

## Kubernetes Basics

### Q1: What is Kubernetes and why is it important?

**Answer:**
Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

**Key concepts:**
- **Pods**: Smallest deployable unit (one or more containers)
- **Nodes**: Worker machines (VMs or physical servers)
- **Cluster**: Group of nodes
- **Deployment**: Manages pods and replicas
- **Service**: Network access to pods

**Why important:**
- **Auto-scaling**: Automatically scale based on demand
- **High availability**: Restarts failed containers
- **Load balancing**: Distributes traffic
- **Rolling updates**: Update without downtime
- **Self-healing**: Automatically replaces failed pods

**Kubernetes vs Docker:**
- **Docker**: Containerization (packages apps)
- **Kubernetes**: Orchestration (manages containers at scale)

**Azure Kubernetes Service (AKS)**: Managed Kubernetes in Azure.

### Q2: Explain basic Kubernetes components.

**Answer:**
**Pods:**
- Smallest deployable unit
- Contains one or more containers
- Share network and storage
- Ephemeral (can be recreated)

**Deployments:**
- Manages pods
- Ensures desired number of replicas
- Handles rolling updates
- Self-healing

**Services:**
- Provides stable network endpoint
- Load balances traffic to pods
- Types: ClusterIP, NodePort, LoadBalancer

**ConfigMaps:**
- Store configuration data
- Separate config from code
- Can be mounted as files or env vars

**Secrets:**
- Store sensitive data (passwords, keys)
- Encrypted at rest
- Similar to ConfigMaps but for secrets

**Example:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 80
```

## Azure Data Stack

### Q3: What is Azure Data Factory and what does it do?

**Answer:**
Azure Data Factory (ADF) is a cloud-based data integration service that orchestrates and automates data movement and transformation.

**Key features:**
- **Data pipelines**: Orchestrate data workflows
- **Data movement**: Copy data between sources
- **Data transformation**: Transform data using various services
- **Scheduling**: Run pipelines on schedule
- **Monitoring**: Track pipeline execution

**Use cases:**
- ETL (Extract, Transform, Load) processes
- Data migration
- Data integration from multiple sources
- Scheduled data processing
- Big data processing

**Components:**
- **Pipelines**: Logical grouping of activities
- **Activities**: Individual tasks (copy, transform)
- **Datasets**: Data structures
- **Linked services**: Connection information
- **Triggers**: When to run pipelines

**Example**: Copy data from on-premises SQL Server to Azure Data Lake, transform it, load into Azure SQL Database.

### Q4: What is Azure Databricks and when would you use it?

**Answer:**
Azure Databricks is an Apache Spark-based analytics platform optimized for Azure.

**Key features:**
- **Apache Spark**: Big data processing engine
- **Collaborative notebooks**: Jupyter-style notebooks
- **ML libraries**: Machine learning capabilities
- **Delta Lake**: Reliable data lakes
- **Auto-scaling**: Automatically scales clusters

**Use cases:**
- Big data analytics
- Machine learning
- Real-time streaming analytics
- Data engineering
- Data science

**When to use:**
- Large-scale data processing
- Need for Spark capabilities
- Machine learning workloads
- Collaborative data science
- Real-time analytics

**Integration:**
- Works with Azure Data Lake
- Integrates with Azure Data Factory
- Connects to various data sources
- Supports multiple languages (Python, Scala, SQL, R)

## Full Stack Concepts

### Q5: Explain the full stack technology layers.

**Answer:**
Full stack refers to all layers of an application:

**1. Frontend (Presentation Layer):**
- **UI/UX**: User interface design
- **Web technologies**: HTML, CSS, JavaScript
- **Frameworks**: React, Angular, Vue.js
- **Mobile**: iOS, Android apps

**2. Application Layer (Backend):**
- **Application servers**: Node.js, .NET, Java
- **APIs**: REST, GraphQL
- **Business logic**: Application code
- **Authentication**: User management

**3. Data Layer:**
- **Databases**: SQL (Azure SQL), NoSQL (Cosmos DB)
- **Caching**: Redis, Azure Cache
- **File storage**: Azure Blob Storage
- **Data warehouses**: Azure Synapse Analytics

**4. Infrastructure Layer:**
- **Compute**: VMs, App Service, Containers
- **Networking**: VNets, Load Balancers
- **Storage**: Disks, Blobs, Files
- **Security**: Firewalls, Key Vault

**DevOps role**: Understand how all layers work together, deploy and monitor entire stack.

### Q6: How does DevOps relate to full stack development?

**Answer:**
**DevOps responsibilities across stack:**

**Frontend:**
- Deploy static assets to CDN
- CI/CD for frontend builds
- Performance monitoring
- A/B testing infrastructure

**Backend:**
- Deploy application code
- API monitoring
- Database migrations
- Service orchestration

**Infrastructure:**
- Provision and manage infrastructure
- Networking configuration
- Security implementation
- Monitoring and alerting

**Full stack understanding helps:**
- Troubleshoot end-to-end issues
- Optimize performance across layers
- Understand dependencies
- Design scalable architectures
- Communicate with development teams

**Example**: User reports slow page load. DevOps needs to check:
- Frontend: CDN, asset sizes
- Backend: API response times
- Database: Query performance
- Infrastructure: Resource utilization

## Object-Oriented Design

### Q7: What are key OOP principles?

**Answer:**
**Core principles:**

**1. Encapsulation:**
- Bundle data and methods together
- Hide internal implementation
- Expose only necessary interface
- Example: Class with private fields, public methods

**2. Inheritance:**
- Create new classes based on existing
- Reuse code
- Establish "is-a" relationship
- Example: Dog extends Animal

**3. Polymorphism:**
- Same interface, different implementations
- Method overriding
- Example: Shape class, Circle and Rectangle implement differently

**4. Abstraction:**
- Hide complexity
- Show only essential features
- Example: Interface defines what, not how

**Benefits for DevOps:**
- Understand code structure
- Design infrastructure as code
- Create reusable modules
- Better communication with developers

### Q8: How does OOP relate to Infrastructure as Code?

**Answer:**
**Similarities:**

**Classes → Modules:**
```hcl
# Terraform module (like a class)
module "vnet" {
  source = "./modules/vnet"
  # Parameters
}
```

**Objects → Resources:**
```hcl
# Resource instance (like an object)
resource "azurerm_virtual_network" "main" {
  # Properties
}
```

**Inheritance → Module composition:**
- Reuse modules
- Extend functionality
- DRY principle

**Encapsulation → Module boundaries:**
- Hide implementation details
- Expose only inputs/outputs
- Internal resources private

**Polymorphism → Provider abstraction:**
- Same interface (Terraform), different providers (Azure, AWS)

**Benefits:**
- Reusable infrastructure code
- Consistent patterns
- Easier maintenance
- Better organization

## UML Basics

### Q9: What is UML and what are common diagrams?

**Answer:**
UML (Unified Modeling Language) is a visual language for modeling software systems.

**Common diagrams:**

**1. Class Diagram:**
- Shows classes and relationships
- Used for system design
- Shows attributes and methods

**2. Sequence Diagram:**
- Shows interactions over time
- Useful for API flows
- Shows message passing

**3. Component Diagram:**
- Shows system components
- Useful for architecture
- Shows dependencies

**4. Deployment Diagram:**
- Shows infrastructure
- Useful for DevOps
- Shows where components run

**5. Activity Diagram:**
- Shows workflows
- Useful for processes
- Shows decision points

**DevOps use:**
- Document architecture
- Design infrastructure
- Communicate with teams
- Plan deployments
- Troubleshoot systems

### Q10: How do you use UML for infrastructure design?

**Answer:**
**Deployment diagrams for infrastructure:**

```
┌─────────────────────────────────────┐
│         Internet                    │
└──────────────┬──────────────────────┘
               │
        ┌──────▼──────┐
        │  Front Door  │
        └──────┬──────┘
               │
    ┌──────────┴──────────┐
    │                      │
┌───▼────┐          ┌──────▼───┐
│ App    │          │  App     │
│ Gateway│          │ Gateway  │
└───┬────┘          └──────┬───┘
    │                      │
┌───▼────┐          ┌──────▼───┐
│ App    │          │  App     │
│ Service│          │ Service  │
└───┬────┘          └──────┬───┘
    │                      │
┌───▼──────────────────────▼───┐
│      Azure SQL Database      │
└──────────────────────────────┘
```

**Use cases:**
- Document current architecture
- Design new infrastructure
- Plan migrations
- Troubleshoot issues
- Onboard team members

**Tools:**
- Draw.io
- Lucidchart
- PlantUML (code-based)
- Visio

## Databases and SQL

### Q11: What are essential SQL concepts for DevOps?

**Answer:**
**Basic SQL:**

**SELECT:**
```sql
SELECT * FROM users;
SELECT id, name, email FROM users WHERE active = 1;
```

**JOIN:**
```sql
SELECT u.name, o.total
FROM users u
JOIN orders o ON u.id = o.user_id;
```

**Aggregation:**
```sql
SELECT COUNT(*) FROM users;
SELECT AVG(price) FROM products;
SELECT user_id, SUM(amount) FROM orders GROUP BY user_id;
```

**Indexes:**
```sql
CREATE INDEX idx_email ON users(email);
-- Speeds up queries on email column
```

**Why important for DevOps:**
- Understand database performance
- Write monitoring queries
- Troubleshoot database issues
- Optimize queries
- Plan database migrations

### Q12: How do you monitor and optimize databases?

**Answer:**
**Monitoring:**
```sql
-- Slow queries
SELECT * FROM sys.dm_exec_query_stats
ORDER BY total_elapsed_time DESC;

-- Connection count
SELECT COUNT(*) FROM sys.dm_exec_sessions;

-- Database size
SELECT 
    name,
    size * 8 / 1024 AS size_mb
FROM sys.master_files
WHERE database_id = DB_ID();
```

**Optimization:**
- Add indexes on frequently queried columns
- Optimize queries (avoid SELECT *)
- Use connection pooling
- Monitor query performance
- Regular maintenance (index rebuild, statistics update)

**Azure SQL:**
- Query Performance Insight
- Automatic tuning
- Database Advisor
- Monitor metrics in Azure Portal

## Python Basics

### Q13: What Python skills are useful for DevOps?

**Answer:**
**Essential concepts:**

**Variables and types:**
```python
name = "John"
age = 30
is_active = True
```

**Lists and dictionaries:**
```python
users = ["user1", "user2", "user3"]
user_info = {
    "name": "John",
    "age": 30
}
```

**Loops:**
```python
for user in users:
    print(user)

for i in range(10):
    print(i)
```

**Functions:**
```python
def deploy_app(app_name):
    print(f"Deploying {app_name}")
    # Deployment logic
```

**File operations:**
```python
with open("config.json", "r") as f:
    config = json.load(f)
```

**Use cases:**
- Automation scripts
- API interactions
- Configuration management
- Data processing
- Infrastructure automation

**Libraries:**
- **boto3**: AWS SDK
- **azure-sdk**: Azure SDK
- **requests**: HTTP requests
- **jinja2**: Templates
- **pyyaml**: YAML parsing

### Q14: Write a Python script for Azure automation.

**Answer:**
```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient
from azure.mgmt.resource import ResourceManagementClient

# Authenticate
credential = DefaultAzureCredential()
subscription_id = "your-subscription-id"

# Initialize clients
compute_client = ComputeManagementClient(credential, subscription_id)
resource_client = ResourceManagementClient(credential, subscription_id)

# List all VMs
resource_groups = resource_client.resource_groups.list()
for rg in resource_groups:
    vms = compute_client.virtual_machines.list(rg.name)
    for vm in vms:
        print(f"VM: {vm.name} in {rg.name}")
        print(f"  Status: {vm.provisioning_state}")
        print(f"  Size: {vm.hardware_profile.vm_size}")

# Start a VM
compute_client.virtual_machines.begin_start(
    resource_group_name="myRG",
    vm_name="myVM"
)
print("VM started")
```

**Use cases:**
- Automate Azure operations
- Generate reports
- Bulk operations
- Custom tooling
- Integration with other systems

