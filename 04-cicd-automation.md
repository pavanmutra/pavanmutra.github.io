# CI/CD Automation - Interview Q&A

## GitLab CI/CD

### Q1: What is GitLab CI/CD and how does it work?

**Answer:**
GitLab CI/CD is a built-in continuous integration and deployment tool in GitLab. It works by:
- **CI/CD Pipeline**: Automated process that runs when you push code
- **GitLab Runner**: Agent that executes jobs (can be on GitLab servers or your own servers)
- **.gitlab-ci.yml**: Configuration file that defines the pipeline
- **Stages**: Groups of jobs that run in sequence (build, test, deploy)
- **Jobs**: Individual tasks that run commands

**Workflow:**
1. Developer pushes code to GitLab
2. GitLab detects `.gitlab-ci.yml` file
3. GitLab Runner picks up the pipeline
4. Jobs run in defined stages
5. Results reported back to GitLab

**Benefits**: Integrated with GitLab, easy to set up, supports Docker, parallel execution, artifacts management.

### Q2: Explain the structure of a GitLab CI/CD pipeline.

**Answer:**
**Basic structure:**
```yaml
# .gitlab-ci.yml

# Define stages (order matters)
stages:
  - build
  - test
  - deploy

# Variables available to all jobs
variables:
  DOCKER_IMAGE: myapp:latest

# Build stage
build:
  stage: build
  script:
    - echo "Building application"
    - docker build -t $DOCKER_IMAGE .
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

# Test stage
test:
  stage: test
  script:
    - echo "Running tests"
    - npm test
  only:
    - merge_requests
    - main

# Deploy stage
deploy:
  stage: deploy
  script:
    - echo "Deploying to production"
    - ./deploy.sh
  only:
    - main
  environment:
    name: production
```

**Key components:**
- **stages**: Define order of execution
- **jobs**: Individual tasks (build, test, deploy)
- **script**: Commands to run
- **artifacts**: Files to save between jobs
- **only/except**: Control when jobs run
- **environment**: Track deployments

### Q3: How do you handle secrets in GitLab CI/CD?

**Answer:**
**Method 1: GitLab CI/CD Variables** (recommended):
1. Go to Project → Settings → CI/CD → Variables
2. Add variable (e.g., `AZURE_CLIENT_SECRET`)
3. Mark as "Protected" (only runs on protected branches)
4. Mark as "Masked" (hidden in logs)

**Use in pipeline:**
```yaml
deploy:
  script:
    - az login --service-principal \
        -u $AZURE_CLIENT_ID \
        -p $AZURE_CLIENT_SECRET \
        --tenant $AZURE_TENANT_ID
```

**Method 2: External secrets (Azure Key Vault)**:
```yaml
deploy:
  script:
    - az keyvault secret show --vault-name myvault --name mysecret
```

**Method 3: File-based secrets**:
```yaml
deploy:
  before_script:
    - echo "$AZURE_CREDENTIALS" > azure-credentials.json
  script:
    - az login --service-principal @azure-credentials.json
  after_script:
    - rm azure-credentials.json
```

**Best practices:**
- Never commit secrets to repository
- Use masked variables for sensitive data
- Use protected variables for production secrets
- Rotate secrets regularly
- Use least privilege (only give access needed)

### Q4: Write a GitLab CI/CD pipeline for a Node.js application deploying to Azure.

**Answer:**
```yaml
# .gitlab-ci.yml

stages:
  - build
  - test
  - deploy

variables:
  NODE_VERSION: "18"
  AZURE_WEB_APP_NAME: "my-nodejs-app"
  AZURE_RESOURCE_GROUP: "myResourceGroup"

# Build job
build:
  stage: build
  image: node:$NODE_VERSION
  script:
    - echo "Installing dependencies"
    - npm ci
    - echo "Building application"
    - npm run build
  artifacts:
    paths:
      - node_modules/
      - dist/
    expire_in: 1 hour

# Test job
test:
  stage: test
  image: node:$NODE_VERSION
  script:
    - echo "Running unit tests"
    - npm test
    - echo "Running linting"
    - npm run lint
  coverage: '/Coverage: \d+\.\d+%/'
  only:
    - merge_requests
    - main
    - develop

# Deploy to staging
deploy_staging:
  stage: deploy
  image: mcr.microsoft.com/azure-cli:latest
  script:
    - echo "Deploying to staging"
    - az login --service-principal \
        -u $AZURE_CLIENT_ID \
        -p $AZURE_CLIENT_SECRET \
        --tenant $AZURE_TENANT_ID
    - az webapp deployment source config-zip \
        --resource-group $AZURE_RESOURCE_GROUP \
        --name $AZURE_WEB_APP_NAME-staging \
        --src dist.zip
  environment:
    name: staging
    url: https://$AZURE_WEB_APP_NAME-staging.azurewebsites.net
  only:
    - develop

# Deploy to production
deploy_production:
  stage: deploy
  image: mcr.microsoft.com/azure-cli:latest
  script:
    - echo "Deploying to production"
    - az login --service-principal \
        -u $AZURE_CLIENT_ID \
        -p $AZURE_CLIENT_SECRET \
        --tenant $AZURE_TENANT_ID
    - az webapp deployment source config-zip \
        --resource-group $AZURE_RESOURCE_GROUP \
        --name $AZURE_WEB_APP_NAME \
        --src dist.zip
  environment:
    name: production
    url: https://$AZURE_WEB_APP_NAME.azurewebsites.net
  only:
    - main
  when: manual  # Requires manual approval
```

### Q5: Explain GitLab CI/CD caching and artifacts.

**Answer:**
**Caching** (speeds up builds by reusing data):
```yaml
build:
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
      - .npm/
  script:
    - npm ci  # Uses cached node_modules if available
```
- Cache is shared between pipelines
- Can be cleared manually
- Faster than downloading dependencies every time

**Artifacts** (files passed between jobs):
```yaml
build:
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
      - build/
    expire_in: 1 week
    reports:
      junit: test-results.xml

test:
  script:
    - npm test
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
```
- Artifacts are downloaded by subsequent jobs
- Can set expiration time
- Can store test reports, coverage, build outputs

**Difference**: Cache is for speed (dependencies), artifacts are for passing files between jobs (build outputs, reports).

## Azure DevOps Pipelines

### Q6: What is Azure DevOps and how does it differ from GitLab CI/CD?

**Answer:**
Azure DevOps is Microsoft's suite of development tools including:
- **Azure Repos**: Git repositories
- **Azure Pipelines**: CI/CD (similar to GitLab CI/CD)
- **Azure Boards**: Project management
- **Azure Artifacts**: Package management
- **Azure Test Plans**: Testing tools

**Key differences from GitLab:**
- **Integration**: Better integration with Azure services
- **UI**: More visual pipeline designer (YAML or visual)
- **Agents**: Self-hosted or Microsoft-hosted agents
- **Pricing**: Free tier for open source, paid for private projects
- **Multi-cloud**: Works with Azure, AWS, GCP

**Similarities**: Both use YAML for pipelines, both support Docker, both have built-in repositories.

### Q7: Explain Azure DevOps pipeline structure (YAML).

**Answer:**
```yaml
# azure-pipelines.yml

trigger:
  branches:
    include:
      - main
      - develop
  paths:
    exclude:
      - README.md

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: my-variable-group
  - name: buildConfiguration
    value: 'Release'

stages:
- stage: Build
  displayName: 'Build Application'
  jobs:
  - job: BuildJob
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '18.x'
    
    - script: |
        npm ci
        npm run build
      displayName: 'Build application'
    
    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: '$(System.DefaultWorkingDirectory)/dist'
        artifactName: 'drop'

- stage: Test
  displayName: 'Run Tests'
  dependsOn: Build
  jobs:
  - job: TestJob
    steps:
    - download: current
      artifact: drop
    
    - script: npm test
      displayName: 'Run tests'

- stage: Deploy
  displayName: 'Deploy to Azure'
  dependsOn: Test
  condition: succeeded()
  jobs:
  - deployment: DeployJob
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'my-azure-connection'
              appName: 'my-web-app'
              package: '$(Pipeline.Workspace)/drop'
```

**Key components:**
- **trigger**: When pipeline runs
- **pool**: Agent pool (Microsoft-hosted or self-hosted)
- **variables**: Configuration values
- **stages**: Groups of jobs
- **jobs**: Groups of steps
- **steps**: Individual tasks
- **tasks**: Pre-built or custom tasks

### Q8: How do you manage secrets in Azure DevOps pipelines?

**Answer:**
**Method 1: Variable Groups** (recommended):
1. Go to Pipelines → Library → Variable groups
2. Create variable group
3. Add variables, mark as "Secret"
4. Link to pipeline

**Use in pipeline:**
```yaml
variables:
  - group: my-secrets

steps:
  - script: |
      az login --service-principal \
        -u $(AZURE_CLIENT_ID) \
        -p $(AZURE_CLIENT_SECRET) \
        --tenant $(AZURE_TENANT_ID)
```

**Method 2: Azure Key Vault integration**:
```yaml
variables:
  - group: my-keyvault-vars

steps:
  - task: AzureKeyVault@2
    inputs:
      azureSubscription: 'my-azure-connection'
      KeyVaultName: 'my-keyvault'
      SecretsFilter: 'secret1,secret2'

  - script: |
      echo "Secret value: $(secret1)"
```

**Method 3: Service Connections**:
- Create service connection in Project Settings
- Authenticates to Azure automatically
- No need to store credentials

**Best practices**: Use variable groups for secrets, use Key Vault for sensitive data, use service connections for Azure authentication.

### Q9: Write an Azure DevOps pipeline for infrastructure deployment using Terraform.

**Answer:**
```yaml
# azure-pipelines.yml

trigger:
  branches:
    include:
      - main
      - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: terraform-vars
  - name: terraformVersion
    value: '1.5.0'

stages:
- stage: Validate
  displayName: 'Validate Terraform'
  jobs:
  - job: ValidateJob
    steps:
    - task: TerraformInstaller@0
      inputs:
        terraformVersion: $(terraformVersion)
    
    - task: TerraformTaskV2@2
      displayName: 'Terraform Init'
      inputs:
        provider: 'azurerm'
        command: 'init'
        backendServiceArm: 'my-azure-connection'
        backendAzureRmResourceGroupName: 'tfstate-rg'
        backendAzureRmStorageAccountName: 'tfstatestorage'
        backendAzureRmContainerName: 'tfstate'
        backendAzureRmKey: 'dev.terraform.tfstate'
    
    - task: TerraformTaskV2@2
      displayName: 'Terraform Validate'
      inputs:
        command: 'validate'
    
    - task: TerraformTaskV2@2
      displayName: 'Terraform Format Check'
      inputs:
        command: 'fmt'
        commandOptions: '-check -diff'

- stage: Plan
  displayName: 'Terraform Plan'
  dependsOn: Validate
  jobs:
  - job: PlanJob
    steps:
    - task: TerraformInstaller@0
      inputs:
        terraformVersion: $(terraformVersion)
    
    - task: TerraformTaskV2@2
      displayName: 'Terraform Init'
      inputs:
        provider: 'azurerm'
        command: 'init'
        backendServiceArm: 'my-azure-connection'
        backendAzureRmResourceGroupName: 'tfstate-rg'
        backendAzureRmStorageAccountName: 'tfstatestorage'
        backendAzureRmContainerName: 'tfstate'
        backendAzureRmKey: 'dev.terraform.tfstate'
    
    - task: TerraformTaskV2@2
      displayName: 'Terraform Plan'
      inputs:
        command: 'plan'
        commandOptions: '-out=tfplan'
        environmentServiceName: 'my-azure-connection'
    
    - task: PublishBuildArtifacts@1
      displayName: 'Publish Plan'
      inputs:
        pathToPublish: '$(System.DefaultWorkingDirectory)/tfplan'
        artifactName: 'tfplan'

- stage: Deploy
  displayName: 'Terraform Apply'
  dependsOn: Plan
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployJob
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: TerraformInstaller@0
            inputs:
              terraformVersion: $(terraformVersion)
          
          - task: TerraformTaskV2@2
            displayName: 'Terraform Init'
            inputs:
              provider: 'azurerm'
              command: 'init'
              backendServiceArm: 'my-azure-connection'
              backendAzureRmResourceGroupName: 'tfstate-rg'
              backendAzureRmStorageAccountName: 'tfstatestorage'
              backendAzureRmContainerName: 'tfstate'
              backendAzureRmKey: 'dev.terraform.tfstate'
          
          - task: TerraformTaskV2@2
            displayName: 'Terraform Apply'
            inputs:
              command: 'apply'
              commandOptions: 'tfplan'
              environmentServiceName: 'my-azure-connection'
```

### Q10: Explain Azure DevOps release pipelines vs YAML pipelines.

**Answer:**
**YAML Pipelines** (modern approach):
- Code-based (YAML file in repository)
- Version controlled
- Easy to review changes
- Supports everything release pipelines do
- Recommended for new projects

**Release Pipelines** (classic, visual):
- UI-based configuration
- Not in version control (stored in Azure DevOps)
- Visual designer
- Good for complex multi-environment deployments
- Being phased out in favor of YAML

**Comparison:**
- **YAML**: Better for DevOps practices, code review, version control
- **Release Pipelines**: Easier for non-developers, visual approval gates

**Best practice**: Use YAML pipelines. They're the future and support all features.

## Build, Test, and Deployment Automation

### Q11: How do you implement automated testing in CI/CD pipelines?

**Answer:**
**Types of tests in pipeline:**

1. **Unit Tests** (fast, run on every commit):
```yaml
test_unit:
  stage: test
  script:
    - npm test -- --coverage
  coverage: '/Coverage: \d+\.\d+%/'
```

2. **Integration Tests** (slower, run on merge requests):
```yaml
test_integration:
  stage: test
  script:
    - docker-compose up -d
    - npm run test:integration
  only:
    - merge_requests
```

3. **Security Scanning** (SAST/DAST):
```yaml
security_scan:
  stage: test
  script:
    - npm audit
    - sonar-scanner
  allow_failure: true  # Don't fail pipeline, but report
```

4. **Performance Tests** (run before production deploy):
```yaml
test_performance:
  stage: test
  script:
    - npm run test:performance
  only:
    - main
```

**Best practices:**
- Run fast tests first (unit tests)
- Run slow tests in parallel
- Fail fast (stop on first failure)
- Generate test reports
- Set quality gates (minimum coverage, no critical bugs)

### Q12: Explain blue-green deployment strategy in CI/CD.

**Answer:**
Blue-Green deployment means running two identical production environments:
- **Blue**: Current production (users connected here)
- **Green**: New version (deployed but not receiving traffic)

**Process:**
1. Deploy new version to Green environment
2. Test Green environment thoroughly
3. Switch traffic from Blue to Green (load balancer change)
4. Monitor Green environment
5. If issues, switch back to Blue
6. If successful, keep Green and make it new Blue

**Benefits:**
- Zero downtime deployment
- Instant rollback (just switch traffic)
- Easy testing of new version before going live

**In CI/CD:**
```yaml
deploy_green:
  stage: deploy
  script:
    - deploy-to-green-environment.sh
    - run-smoke-tests.sh
  environment:
    name: green

switch_traffic:
  stage: deploy
  script:
    - switch-load-balancer-to-green.sh
  when: manual  # Manual approval before switching
  environment:
    name: production
```

**Azure example**: Deploy to App Service slot (staging), test it, then swap slots.

### Q13: How do you implement canary deployments in CI/CD?

**Answer:**
Canary deployment gradually rolls out new version to small percentage of users:
- **10% users**: Get new version
- **90% users**: Still on old version
- Monitor metrics (errors, performance)
- If good, increase to 25%, then 50%, then 100%
- If bad, rollback immediately

**Implementation:**
```yaml
deploy_canary:
  stage: deploy
  script:
    - az webapp deployment slot create \
        --name myapp \
        --resource-group myRG \
        --slot canary
    - deploy-to-slot.sh canary
    - configure-traffic-split.sh 10  # 10% to canary
  environment:
    name: canary

monitor_canary:
  stage: monitor
  script:
    - check-error-rates.sh
    - check-performance.sh
  only:
    - main

promote_canary:
  stage: deploy
  script:
    - configure-traffic-split.sh 50  # Increase to 50%
  when: manual
  only:
    - main
```

**Benefits**: Catch issues early, minimize impact, gradual rollout.

## Pipeline Optimization

### Q14: How do you optimize CI/CD pipeline performance?

**Answer:**
**Optimization strategies:**

1. **Parallel execution**:
```yaml
test_unit:
  stage: test
  script: npm test
  parallel:
    matrix:
      - NODE_VERSION: ["16", "18", "20"]
```

2. **Caching dependencies**:
```yaml
build:
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  script: npm ci  # Faster with cache
```

3. **Conditional execution**:
```yaml
deploy:
  only:
    - main  # Only run on main branch
  except:
    - schedules  # Don't run on scheduled pipelines
```

4. **Artifact management**:
```yaml
build:
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour  # Don't keep forever
```

5. **Job dependencies**:
```yaml
deploy:
  needs: ["build", "test"]  # Only run if both succeed
```

6. **Use faster agents**:
```yaml
build:
  tags:
    - docker
    - fast  # Use agents with these tags
```

7. **Skip unnecessary steps**:
```yaml
test:
  only:
    changes:
      - "src/**/*"  # Only run if source code changed
```

**Results**: Can reduce pipeline time from 30 minutes to 5 minutes.

### Q15: How do you handle pipeline failures and implement retry logic?

**Answer:**
**Retry on transient failures:**
```yaml
deploy:
  retry:
    max: 2  # Retry up to 2 times
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
  script:
    - deploy.sh
```

**Conditional retry:**
```yaml
deploy:
  script:
    - deploy.sh || (sleep 30 && deploy.sh)  # Manual retry with delay
```

**Failure notifications:**
```yaml
deploy:
  script:
    - deploy.sh
  on_failure:
    - echo "Deployment failed!"
    - send-slack-notification.sh
  on_success:
    - echo "Deployment succeeded!"
```

**Manual intervention on failure:**
```yaml
deploy:
  script:
    - deploy.sh
  when: on_failure
  allow_failure: true  # Don't fail entire pipeline
```

**Best practices:**
- Retry transient errors (network, timeouts)
- Don't retry logic errors (code bugs)
- Set maximum retry count
- Notify team on failures
- Log failures for analysis

## Scenario-Based Questions

### Q16: How would you set up a CI/CD pipeline for a microservices architecture?

**Answer:**
**Approach:**
1. **Separate pipelines per service** (or monorepo with path filters):
```yaml
# Service A pipeline
build_service_a:
  only:
    changes:
      - "services/service-a/**/*"

# Service B pipeline
build_service_b:
  only:
    changes:
      - "services/service-b/**/*"
```

2. **Shared pipeline templates** (DRY principle):
```yaml
# .gitlab-ci-template.yml
.build_template: &build_template
  stage: build
  script:
    - docker build -t $IMAGE_NAME .
    - docker push $IMAGE_NAME

# Use template
build_service_a:
  <<: *build_template
  variables:
    IMAGE_NAME: service-a:$CI_COMMIT_SHA
```

3. **Deployment strategy**:
   - Deploy services independently
   - Use feature flags for gradual rollout
   - Health checks before marking as ready
   - Canary deployments per service

4. **Dependency management**:
   - Deploy dependencies first (databases, message queues)
   - Use health checks to verify dependencies ready
   - Version APIs for backward compatibility

5. **Monitoring**:
   - Deploy with monitoring enabled
   - Set up alerts per service
   - Track deployment metrics

### Q17: Your production deployment failed. How would you implement a rollback strategy?

**Answer:**
**Automated rollback:**
```yaml
deploy:
  script:
    - deploy.sh
  on_failure:
    - echo "Deployment failed, rolling back"
    - rollback.sh
    - send-alert.sh

rollback:
  script:
    - az webapp deployment slot swap \
        --name myapp \
        --resource-group myRG \
        --slot staging \
        --target-slot production
  when: on_failure
  only:
    - main
```

**Manual rollback with approval:**
```yaml
rollback:
  script:
    - rollback-to-previous-version.sh
  when: manual
  environment:
    name: production
    action: rollback
```

**Version tagging for rollback:**
```yaml
deploy:
  script:
    - docker tag myapp:latest myapp:$CI_COMMIT_SHA
    - docker push myapp:$CI_COMMIT_SHA
    - deploy-version.sh $CI_COMMIT_SHA

rollback:
  script:
    - deploy-version.sh $PREVIOUS_VERSION  # Deploy previous known good version
```

**Best practices:**
- Always tag deployments with version
- Keep previous versions available
- Test rollback procedure regularly
- Have rollback runbook documented
- Monitor after rollback

### Q18: How would you implement security scanning in your CI/CD pipeline?

**Answer:**
**Multi-layer security:**

1. **SAST (Static Application Security Testing)**:
```yaml
sast_scan:
  stage: test
  script:
    - sonar-scanner
    - npm audit
    - bandit -r .  # Python security scanner
  artifacts:
    reports:
      sast: sast-report.json
```

2. **DAST (Dynamic Application Security Testing)**:
```yaml
dast_scan:
  stage: test
  script:
    - docker-compose up -d
    - zap-baseline.py -t http://localhost:8080
  only:
    - main
```

3. **Dependency scanning**:
```yaml
dependency_scan:
  stage: test
  script:
    - npm audit
    - pip-audit  # Python
    - bundler audit  # Ruby
  artifacts:
    reports:
      dependency_scanning: dependency-report.json
```

4. **Container scanning**:
```yaml
container_scan:
  stage: test
  script:
    - trivy image myapp:latest
    - docker scan myapp:latest
```

5. **Infrastructure scanning** (Terraform):
```yaml
infra_scan:
  stage: test
  script:
    - tfsec .
    - checkov -d .
```

6. **Secrets scanning**:
```yaml
secrets_scan:
  stage: test
  script:
    - truffleHog --regex --entropy=False .
    - git-secrets scan
```

**Quality gates:**
- Block deployment if critical vulnerabilities found
- Warn on high severity
- Generate security reports
- Track vulnerabilities over time

