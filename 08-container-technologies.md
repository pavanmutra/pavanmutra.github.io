# Container Technologies (Docker) - Interview Q&A

## Docker Fundamentals

### Q1: What is Docker and why is it important?

**Answer:**
Docker is a platform that packages applications and their dependencies into containers - lightweight, portable units that run consistently anywhere.

**Key concepts:**
- **Container**: Running instance of an image
- **Image**: Template for creating containers (like a blueprint)
- **Dockerfile**: Instructions for building an image

**Why important:**
- **Consistency**: "Works on my machine" problem solved - runs same everywhere
- **Isolation**: Applications don't interfere with each other
- **Portability**: Run on any system with Docker (Windows, Linux, Mac, cloud)
- **Efficiency**: Containers share OS kernel (lighter than VMs)
- **Scalability**: Easy to run multiple instances
- **DevOps**: Standard way to package and deploy applications

**Docker vs VMs:**
- **VMs**: Full OS per VM, heavier, slower to start
- **Containers**: Share host OS, lighter, start in seconds

### Q2: Explain the difference between Docker image and container.

**Answer:**
**Docker Image:**
- **Template/Blueprint**: Instructions for creating containers
- **Read-only**: Can't be modified (immutable)
- **Layered**: Built from multiple layers (each instruction in Dockerfile creates a layer)
- **Stored**: In registry (Docker Hub, Azure Container Registry)
- **Example**: `nginx:latest`, `node:18`

**Docker Container:**
- **Running instance**: Created from an image
- **Read-write**: Has writable layer on top of image
- **Ephemeral**: Can be created, started, stopped, deleted
- **Running**: Actually executes the application

**Analogy:**
- **Image**: Like a class in programming (template)
- **Container**: Like an object (instance of the class)

**Example:**
```bash
# Image (template)
docker pull nginx:latest

# Container (running instance)
docker run -d -p 80:80 nginx:latest
```

**Key point**: One image can create many containers (like one class can create many objects).

### Q3: What is a Dockerfile and how do you write one?

**Answer:**
Dockerfile is a text file with instructions for building a Docker image.

**Basic structure:**
```dockerfile
# Base image
FROM node:18

# Working directory
WORKDIR /app

# Copy files
COPY package.json .
COPY package-lock.json .

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Command to run
CMD ["node", "server.js"]
```

**Common instructions:**
- **FROM**: Base image to start from
- **WORKDIR**: Set working directory
- **COPY/ADD**: Copy files into image
- **RUN**: Execute commands during build
- **EXPOSE**: Document which port container uses
- **CMD**: Default command when container starts
- **ENV**: Set environment variables
- **ARG**: Build-time variables

**Example for Node.js app:**
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files first (for better caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

EXPOSE 3000

CMD ["node", "server.js"]
```

**Best practices:**
- Use specific version tags (not `latest`)
- Order instructions for better caching (copy package.json before code)
- Use multi-stage builds for smaller images
- Don't run as root user
- Use .dockerignore to exclude files

### Q4: Explain Docker layers and how they affect image size.

**Answer:**
Docker images are built in layers. Each instruction in Dockerfile creates a new layer.

**How layers work:**
```dockerfile
FROM node:18          # Layer 1: Base image
COPY package.json .   # Layer 2: Copy package.json
RUN npm install       # Layer 3: Install dependencies
COPY . .              # Layer 4: Copy application code
CMD ["node", "app.js"] # Layer 5: Command (metadata, no layer)
```

**Layer caching:**
- If layer hasn't changed, Docker reuses cached layer
- If layer changes, all subsequent layers are rebuilt
- This is why order matters in Dockerfile

**Example - Good order (better caching):**
```dockerfile
COPY package.json .    # Changes rarely
RUN npm install        # Only rebuilds if package.json changes
COPY . .               # Changes frequently
```

**Example - Bad order (worse caching):**
```dockerfile
COPY . .               # Changes frequently - invalidates cache
RUN npm install        # Always rebuilds
```

**Image size:**
- Each layer adds to image size
- Layers are shared between images (if same base)
- Deleting files in one layer doesn't remove them from previous layers
- Use multi-stage builds to reduce final image size

**View layers:**
```bash
docker history image-name
docker inspect image-name
```

**Best practices:**
- Order instructions for better caching
- Combine RUN commands to reduce layers
- Use .dockerignore to exclude unnecessary files
- Use multi-stage builds for smaller final images

### Q5: What is Docker Compose and when would you use it?

**Answer:**
Docker Compose is a tool for defining and running multi-container Docker applications using a YAML file.

**Use when:**
- Running multiple containers together (app + database + cache)
- Local development environment
- Testing multi-container setups
- Simple deployments

**Example docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    depends_on:
      - db
      - redis

  db:
    image: postgres:14
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - db-data:/var/lib/postgresql/data

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

volumes:
  db-data:
```

**Commands:**
```bash
# Start all services
docker-compose up

# Start in background
docker-compose up -d

# Stop all services
docker-compose down

# View logs
docker-compose logs

# Rebuild and start
docker-compose up --build
```

**Benefits:**
- Single command to start entire application
- Easy to define relationships between services
- Environment variables and volumes in one place
- Good for development and testing

**Limitations:**
- Not for production orchestration (use Kubernetes)
- Single host only
- No auto-scaling or high availability

## Containerization Best Practices

### Q6: What are Docker best practices for security?

**Answer:**
**1. Don't run as root:**
```dockerfile
# Create non-root user
RUN adduser -D -s /bin/sh appuser
USER appuser
```

**2. Use minimal base images:**
```dockerfile
# Good: Alpine Linux (small, minimal)
FROM node:18-alpine

# Bad: Full OS image (larger, more vulnerabilities)
FROM ubuntu:latest
```

**3. Scan images for vulnerabilities:**
```bash
docker scan image-name
# or use Azure Container Registry scanning
```

**4. Don't store secrets in images:**
```dockerfile
# Bad
ENV DB_PASSWORD=secret123

# Good: Use environment variables or secrets management
# Pass at runtime: docker run -e DB_PASSWORD=...
```

**5. Use .dockerignore:**
```dockerignore
node_modules
.git
.env
*.log
```

**6. Keep images updated:**
```dockerfile
# Use specific versions, update regularly
FROM node:18.17.0-alpine
```

**7. Limit container capabilities:**
```bash
docker run --read-only --tmpfs /tmp image-name
```

**8. Use multi-stage builds:**
```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm run build

# Runtime stage (smaller, no build tools)
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package.json .
RUN npm ci --only=production
CMD ["node", "dist/server.js"]
```

**9. Set resource limits:**
```bash
docker run --memory="512m" --cpus="1.0" image-name
```

**10. Use health checks:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1
```

### Q7: Explain multi-stage builds and their benefits.

**Answer:**
Multi-stage builds use multiple FROM statements in one Dockerfile. Each stage can have different base images and purposes.

**Example:**
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Runtime (final image)
FROM node:18-alpine
WORKDIR /app
# Copy only built files from builder stage
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --only=production
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Benefits:**
- **Smaller final image**: Only includes runtime dependencies, not build tools
- **Security**: Fewer tools in production image = fewer vulnerabilities
- **Faster deployments**: Smaller images download faster
- **Separation**: Build tools separate from runtime

**Before multi-stage**: Final image ~500MB (includes Node.js, npm, build tools, source code)
**After multi-stage**: Final image ~150MB (only Node.js runtime, built files)

**Use cases:**
- Compiled languages (Go, C++)
- Applications that need build tools
- When you want to minimize production image size

### Q8: How do you optimize Docker image size?

**Answer:**
**1. Use minimal base images:**
```dockerfile
# Good: Alpine (5MB)
FROM node:18-alpine

# Bad: Full Ubuntu (100MB+)
FROM ubuntu:latest
```

**2. Multi-stage builds:**
- Build in one stage, copy only needed files to final stage

**3. Combine RUN commands:**
```dockerfile
# Bad: Multiple layers
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2

# Good: Single layer
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    rm -rf /var/lib/apt/lists/*
```

**4. Use .dockerignore:**
```dockerignore
node_modules
.git
*.log
.env
dist
```

**5. Remove unnecessary files:**
```dockerfile
RUN npm install && \
    npm cache clean --force && \
    rm -rf /tmp/*
```

**6. Order instructions for caching:**
```dockerfile
# Copy package.json first (changes rarely)
COPY package*.json ./
RUN npm install

# Copy code last (changes frequently)
COPY . .
```

**7. Use specific tags:**
```dockerfile
# Good: Specific version
FROM node:18.17.0-alpine

# Bad: Latest (might pull larger version)
FROM node:latest
```

**8. Remove package managers after install:**
```dockerfile
# If you don't need apt-get after installs
RUN apt-get update && \
    apt-get install -y package && \
    apt-get remove -y apt-get && \
    rm -rf /var/lib/apt/lists/*
```

**Results**: Can reduce image size by 50-80%.

## Docker in Azure

### Q9: How do you deploy Docker containers to Azure?

**Answer:**
**Multiple options:**

**1. Azure Container Instances (ACI)** - Simplest:
```bash
# Deploy single container
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image myimage:latest \
  --dns-name-label myapp \
  --ports 80
```

**2. Azure App Service** - For web apps:
```bash
# Deploy to App Service
az webapp create \
  --resource-group myRG \
  --plan myAppServicePlan \
  --name mywebapp \
  --deployment-container-image-name myimage:latest
```

**3. Azure Container Apps** - Serverless containers:
```bash
# Deploy container app
az containerapp create \
  --name myapp \
  --resource-group myRG \
  --image myimage:latest \
  --target-port 3000
```

**4. Azure Kubernetes Service (AKS)** - For orchestration:
```bash
# Deploy to AKS
kubectl create deployment myapp --image=myimage:latest
kubectl expose deployment myapp --port=80 --type=LoadBalancer
```

**5. Azure Virtual Machines** - Full control:
```bash
# SSH into VM and run
docker run -d -p 80:80 myimage:latest
```

**Best practices:**
- Use Azure Container Registry to store images
- Use managed services (ACI, App Service) when possible
- Use AKS for complex multi-container applications
- Enable auto-scaling
- Set up health probes

### Q10: Explain Azure Container Registry and its features.

**Answer:**
Azure Container Registry (ACR) is a private Docker registry in Azure for storing container images.

**Features:**
- **Private registry**: Only you can access (unlike Docker Hub public)
- **Geo-replication**: Replicate images to multiple regions
- **Security scanning**: Automatically scans images for vulnerabilities
- **Webhooks**: Notify when images are pushed
- **Integration**: Works with AKS, App Service, ACI
- **Azure AD integration**: Use Azure AD for authentication

**Common operations:**
```bash
# Login to ACR
az acr login --name myregistry

# Build and push image
az acr build --registry myregistry --image myapp:latest .

# Pull image
docker pull myregistry.azurecr.io/myapp:latest

# List images
az acr repository list --name myregistry

# Delete image
az acr repository delete --name myregistry --image myapp:latest
```

**Security:**
- **Admin user**: Enable admin user for Docker login
- **Service principals**: Use for CI/CD pipelines
- **Managed identity**: Use from Azure services (no credentials)
- **Network rules**: Restrict access by IP/VNet
- **Private endpoints**: Access from VNet only

**Pricing tiers:**
- **Basic**: For learning/testing
- **Standard**: For production (includes geo-replication)
- **Premium**: Advanced features (content trust, zone redundancy)

**Best practices:**
- Use ACR for all production images
- Enable vulnerability scanning
- Use tags for versions (not just `latest`)
- Set up retention policies
- Use geo-replication for global deployments

## Scenario-Based Questions

### Q11: How would you containerize a legacy .NET application for Azure?

**Answer:**
**Step 1: Create Dockerfile:**
```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /app
COPY *.csproj ./
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:6.0
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 80
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

**Step 2: Create .dockerignore:**
```dockerignore
bin/
obj/
*.user
*.suo
.vs/
```

**Step 3: Build and test locally:**
```bash
docker build -t myapp:latest .
docker run -p 8080:80 myapp:latest
```

**Step 4: Push to Azure Container Registry:**
```bash
az acr login --name myregistry
az acr build --registry myregistry --image myapp:latest .
```

**Step 5: Deploy to Azure:**
```bash
# Option 1: App Service
az webapp create \
  --resource-group myRG \
  --plan myPlan \
  --name myapp \
  --deployment-container-image-name myregistry.azurecr.io/myapp:latest

# Option 2: Container Instances
az container create \
  --resource-group myRG \
  --name myapp \
  --image myregistry.azurecr.io/myapp:latest \
  --registry-login-server myregistry.azurecr.io \
  --registry-username myregistry \
  --registry-password <password> \
  --dns-name-label myapp \
  --ports 80
```

**Considerations:**
- Handle configuration (use environment variables or Key Vault)
- Database connections (update connection strings)
- File storage (use Azure Storage, not local filesystem)
- Logging (send to Azure Monitor/Application Insights)
- Health checks (add health endpoint)

### Q12: Explain how you would set up CI/CD for Docker containers in Azure.

**Answer:**
**GitLab CI/CD example:**
```yaml
stages:
  - build
  - test
  - push
  - deploy

variables:
  ACR_NAME: myregistry
  IMAGE_NAME: myapp

build:
  stage: build
  script:
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHA .
    - docker tag $IMAGE_NAME:$CI_COMMIT_SHA $IMAGE_NAME:latest

test:
  stage: test
  script:
    - docker run --rm $IMAGE_NAME:$CI_COMMIT_SHA npm test

push:
  stage: push
  script:
    - az acr login --name $ACR_NAME
    - az acr build --registry $ACR_NAME --image $IMAGE_NAME:$CI_COMMIT_SHA .
    - az acr build --registry $ACR_NAME --image $IMAGE_NAME:latest .
  only:
    - main

deploy:
  stage: deploy
  script:
    - az container create \
        --resource-group myRG \
        --name myapp \
        --image $ACR_NAME.azurecr.io/$IMAGE_NAME:$CI_COMMIT_SHA \
        --registry-login-server $ACR_NAME.azurecr.io \
        --registry-username $ACR_NAME \
        --registry-password $ACR_PASSWORD \
        --dns-name-label myapp \
        --ports 80
  only:
    - main
```

**Azure DevOps example:**
```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: Docker@2
  displayName: 'Build image'
  inputs:
    containerRegistry: 'myACR'
    repository: 'myapp'
    command: 'build'
    Dockerfile: '**/Dockerfile'
    tags: '$(Build.BuildId)'

- task: Docker@2
  displayName: 'Push image'
  inputs:
    containerRegistry: 'myACR'
    repository: 'myapp'
    command: 'push'
    tags: '$(Build.BuildId)'

- task: AzureWebAppContainer@1
  displayName: 'Deploy to App Service'
  inputs:
    azureSubscription: 'my-azure-connection'
    appName: 'myapp'
    containers: 'myregistry.azurecr.io/myapp:$(Build.BuildId)'
```

**Best practices:**
- Tag images with commit SHA (traceable)
- Scan images for vulnerabilities
- Test before pushing
- Use ACR tasks for building in Azure
- Deploy to staging first, then production
- Use health checks in deployment

