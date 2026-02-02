
# Event Management Web App with Azure CI/CD

A .NET Core + Angular event management system showcasing automated CI/CD pipeline implementation using Azure DevOps.

## Overview

Full-stack event booking application demonstrating modern DevOps practices with automated build, test, and deployment workflows.

**Tech Stack:** .NET Core Web API | Angular | Azure DevOps | Azure App Service | Azure SQL | Azure SQL Database

## Quick Start

```bash
# Clone repository
git clone https://github.com/Shashank7708/EventManagementWebAppWith_AzureCiCd_Pipeline.git

# Backend
cd EventBookingAppNew
dotnet restore && dotnet run

# Frontend
cd UI/EventBookingWeb
npm install && ng serve
```

## Project Structure

```
├── EventBookingAppNew/              # .NET Core Web API
├── EventBookingAppNew.Service/      # Business logic layer
├── Event-Booking-AppCore/           # Domain models
├── UI/EventBookingWeb/              # Angular frontend
└── Azure DevOps PipelineFor .NETWebApi.yml    # CI/CD Pipeline
```

## 🚀 DevOps Pipeline

### Pipeline Architecture

The Azure DevOps pipeline automates the entire software delivery lifecycle with SonarQube integration:

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   COMMIT     │───▶│    BUILD     │───▶│  SONARQUBE   │───▶│     TEST     │
│  (GitHub)    │    │  (Compile)   │    │ (Code Scan)  │    │  (Validate)  │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
                                               │                     │
                                               ▼                     ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   MONITOR    │◀───│    DEPLOY    │◀───│   PACKAGE    │◀───│ QUALITY GATE │
│ (App Insights)    │   (Azure)    │    │ (Artifacts)  │    │  (SonarQube) │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

### Branch Strategy

The project implements a Git Flow branching model with environment-specific deployments:

```
┌─────────────────────────────────────────────────────────────┐
│                     Branch Strategy                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  dev branch      →  Development Environment (Auto Deploy)   │
│       ↓                                                      │
│  uat branch      →  UAT Environment (Auto Deploy)           │
│       ↓                                                      │
│  main branch     →  Production Environment (Manual Gate)    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```


### Pipeline Stages

**1. Build Stage**
- Restore NuGet packages for .NET Core API
- Build Angular frontend application
- Compile solution with Release configuration
- Generate build artifacts

**2. SonarQube Analysis**
- **Prepare Analysis:** Initialize SonarQube scanner
- **Code Scanning:** Analyze code quality, bugs, vulnerabilities
- **Coverage Analysis:** Process test coverage data
- **Publish Results:** Send metrics to SonarQube server
- **Quality Gate Check:** Validate code meets quality standards

**3. Test Stage**
- Execute unit tests
- Run integration tests
- Generate code coverage reports
- Publish test results to pipeline

**4. Quality Gate Stage**
- Wait for SonarQube quality gate status
- Block deployment if quality gate fails
- Report quality metrics in pipeline

**5. Package Stage**
- Create deployment packages
- Publish artifacts to pipeline
- Prepare release assets

**6. Deploy Stage (Environment-Specific)**
- **Dev:** Auto-deploy to Development App Service
- **UAT:** Auto-deploy to UAT App Service  
- **Production:** Manual approval → Deploy to Production App Service
- Configure environment variables
- Health check validation
- Smoke tests post-deployment

### Pipeline Configuration

**Triggers:**
- **CI Trigger:** Automatic build on push to `dev`, `uat`, or `main` branches
- **PR Trigger:** Validation on pull requests to `uat` or `main`
- **Manual Trigger:** On-demand deployment

**Key Variables:**
```yaml
variables:
  # Azure Configuration
  azureSubscription: 'Azure-Service-Connection'
  buildConfiguration: 'Release'
  
  # SonarQube Configuration
  sonarQubeServiceConnection: 'SonarQube-Connection'
  sonarQubeProjectKey: 'event-management-app'
  sonarQubeProjectName: 'Event Management Web App'
  
  # Environment-Specific App Services
  devWebAppName: 'event-mgmt-api-dev'
  uatWebAppName: 'event-mgmt-api-uat'
  prodWebAppName: 'event-mgmt-api-prod'
  
  # Resource Groups
  devResourceGroup: 'rg-event-mgmt-dev'
  uatResourceGroup: 'rg-event-mgmt-uat'
  prodResourceGroup: 'rg-event-mgmt-prod'
```

### Setting Up the Pipeline



**1. SonarQube Setup**

**Option A: SonarCloud (Recommended)**
```bash
# 1. Sign up at https://sonarcloud.io
# 2. Create new organization
# 3. Create new project with key: event-management-app
# 4. Generate authentication token
```

**Option B: Self-Hosted SonarQube**
```bash
# Deploy SonarQube on Azure Container Instance or VM
az container create --resource-group rg-sonarqube \
  --name sonarqube --image sonarqube:latest \
  --ports 9000 --cpu 2 --memory 4
```

**2. Azure DevOps Configuration**

**Service Connections:**
- Navigate to Project Settings → Service Connections
- Create **Azure Resource Manager** connection for Azure subscription
- Create **SonarQube** connection:
  - Server URL: `https://sonarcloud.io` or your SonarQube server URL
  - Token: Your SonarQube authentication token

**3. Pipeline Setup**
- Navigate to Azure DevOps → Pipelines → New Pipeline
- Select GitHub repository
- Use existing YAML file: `Azure DevOps PipelineFor .NETWebApi.yml`
- Configure pipeline variables (see table below)
- Save and run


### Deployment Strategy

**Multi-Environment Deployment:**

| Environment | Branch | Deployment | Approval | URL |
|------------|--------|------------|----------|-----|
| **Development** | `dev` | Automatic | None | dev-app.azurewebsites.net |
| **UAT** | `uat` | Automatic | Post-merge | uat-app.azurewebsites.net |
| **Production** | `main` | Manual | Required | app.azurewebsites.net |

**Deployment Flow:**
```
Feature Branch
      ↓ (PR + Review)
   dev branch ──────────→ Auto Deploy to DEV
      ↓ (PR + Review + Tests)
   uat branch ──────────→ Auto Deploy to UAT
      ↓ (PR + 2 Reviews + Approval)
   main branch ─────────→ Manual Deploy to PROD
```

**Quality Gates at Each Stage:**
- ✅ All unit tests pass
- ✅ SonarQube quality gate passed
- ✅ Code coverage > 80%
- ✅ No critical/blocker bugs
- ✅ Security vulnerabilities resolved
- ✅ Technical debt ratio acceptable

**Blue-Green Deployment (Production Only):**
- Zero-downtime deployments using Azure deployment slots
- Deploy to staging slot first
- Run smoke tests on staging
- Swap slots only after validation
- Automatic rollback on failure

## CI/CD Features

✅ **Continuous Integration**
- Automated builds on every commit (dev, uat, main branches)
- Parallel build execution for faster feedback
- Unit and integration testing
- Code quality gates with SonarQube
- Build artifact generation and versioning

✅ **Code Quality & Security (SonarQube)**
- Static code analysis for bugs and code smells
- Security vulnerability detection (OWASP Top 10)
- Code coverage tracking and enforcement
- Technical debt measurement
- Duplicate code detection
- Quality gate enforcement (blocks bad code from deployment)

✅ **Continuous Deployment**
- Environment-specific automated deployments
- Dev: Auto-deploy on every commit
- UAT: Auto-deploy after PR approval
- Production: Manual approval with quality gates
- Deployment slots for zero-downtime releases
- Automated rollback capabilities

✅ **Multi-Environment Support**
- Separate pipelines for dev, uat, and production
- Environment-specific configurations
- Isolated Azure resources per environment
- Environment-specific secrets and variables

✅ **Infrastructure as Code**
- YAML-based pipeline definition
- Version-controlled CI/CD configuration
- Reusable pipeline templates
- Parameterized environment deployments

✅ **Monitoring & Feedback**
- Application Insights integration per environment
- SonarQube quality dashboards
- Build status notifications (Teams/Slack)
- Deployment tracking and audit logs
- Performance monitoring and alerts
- Real-time pipeline status updates

### SonarQube Configuration

**sonar-project.properties:**
```properties
# Project identification
sonar.projectKey=event-management-app
sonar.projectName=Event Management Web App
sonar.projectVersion=1.0

# Source code location
sonar.sources=.
sonar.exclusions=**/bin/**,**/obj/**,**/node_modules/**,**/dist/**

# Test coverage
sonar.cs.opencover.reportsPaths=**/coverage.opencover.xml
sonar.javascript.lcov.reportPaths=**/coverage/lcov.info

# Code analysis
sonar.qualitygate.wait=true
sonar.qualitygate.timeout=300
```

**Quality Gate Rules:**
```yaml
Conditions:
  - Coverage: >= 80%
  - Duplicated Lines: <= 3%
  - Maintainability Rating: A
  - Reliability Rating: A
  - Security Rating: A
  - Security Hotspots: Reviewed 100%
```

## SonarQube Quality Metrics

### Current Project Health

The project maintains high code quality standards enforced by SonarQube:

```
┌─────────────────────────────────────────────────────┐
│           SonarQube Quality Dashboard                │
├─────────────────────────────────────────────────────┤
│  Reliability Rating          │  A                   │
│  Security Rating             │  A                   │
│  Maintainability Rating      │  A                   │
│  Coverage                    │  > 80%               │
│  Duplications                │  < 3%                │
│  Code Smells                 │  Acceptable          │
│  Technical Debt              │  < 5%                │
└─────────────────────────────────────────────────────┘
```



**In Azure DevOps:**
- Navigate to Pipeline → Run → Extensions → SonarQube Quality Gate
- View detailed analysis reports and metrics
- Check quality gate status in pipeline logs


### Code Quality Monitoring
- **SonarQube Dashboard:**
  - Real-time code quality metrics
  - Bug and vulnerability trends
  - Code coverage history
  - Technical debt tracking
  - Security hotspot monitoring
  - Code smell detection

### Pipeline Monitoring
- **Azure DevOps Analytics:**
  - Build success/failure rates
  - Deployment frequency metrics
  - Lead time for changes
  - Mean time to recovery (MTTR)
  - Test results and coverage trends

### Key Metrics Tracked
| Metric | Target | Tool |
|--------|--------|------|
| Code Coverage | ≥ 80% | SonarQube |
| Build Success Rate | ≥ 95% | Azure DevOps |
| Deployment Frequency | Daily (dev/uat) | Azure DevOps |
| Mean Time to Deploy | < 15 min | Azure DevOps |
| Application Availability | ≥ 99.9% | Application Insights |
| API Response Time | < 500ms | Application Insights |
| Security Vulnerabilities | 0 Critical | SonarQube |
| Technical Debt Ratio | < 5% | SonarQube |

## Best Practices Implemented

🔧 **DevOps:**
- Infrastructure as Code (YAML pipelines)
- GitFlow branching strategy (dev → uat → main)
- Automated testing at every stage
- Environment parity across dev, uat, and production
- Secret management with Azure Key Vault
- Continuous monitoring and feedback loops
- Deployment frequency optimization

🔒 **Security:**
- SonarQube security vulnerability scanning
- OWASP Top 10 vulnerability checks
- Secure service connections with managed identities
- Environment-specific secrets and credentials
- Role-based access control (RBAC) in Azure
- Automated security scanning in pipeline
- Secret detection in code repositories
- SSL/TLS enforcement on all environments

📊 **Code Quality:**
- SonarQube static code analysis
- Mandatory code review via pull requests
- Quality gates block poor quality deployments
- Code coverage requirements (≥80%)
- Automated code quality checks in pipeline
- Branch protection policies
- Technical debt monitoring and management
- Duplicate code detection

🧪 **Testing:**
- Unit tests with > 80% coverage
- Integration testing in pipeline
- End-to-end testing in UAT environment
- Automated smoke tests post-deployment
- Performance testing before production
- Test results published in pipeline

🚀 **Deployment:**
- Multi-environment deployment strategy
- Blue-green deployments for production
- Automated rollback on failure
- Zero-downtime deployments
- Canary releases capability
- Feature flags for gradual rollout
- Environment-specific configurations

📈 **Monitoring:**
- Application Insights per environment
- SonarQube quality dashboards
- Real-time alerts and notifications
- Performance metrics tracking
- Build and deployment analytics
- Infrastructure monitoring with Azure Monitor


### Pull Request Requirements

✅ **For dev branch:**
- Pipeline must pass (build + tests + SonarQube)


✅ **For main branch & uat branch:**
- All dev requirements
- At least 2 approvals
- UAT sign-off required for main
- Automated CI/CD deployment 


### Local SonarQube Testing (Optional)

```bash
# Install SonarScanner for .NET
dotnet tool install --global dotnet-sonarscanner

# Run analysis
dotnet sonarscanner begin /k:"event-management-app" /d:sonar.host.url="http://localhost:9000"
dotnet build
dotnet test --collect:"XPlat Code Coverage"
dotnet sonarscanner end
```







### Pipeline Stages

**1. Build Stage**
- Restore NuGet packages for .NET Core API
- Build Angular frontend application
- Compile solution with Release configuration
- Generate build artifacts

**2. Test Stage**
- Execute unit tests
- Run integration tests
- Generate code coverage reports
- Perform code quality analysis

**3. Package Stage**
- Create deployment packages
- Publish artifacts to pipeline
- Prepare release assets

**4. Deploy Stage**
- Deploy API to Azure App Service
- Deploy frontend to Azure
- Configure environment variables
- Health check validation

### Pipeline Configuration

**Triggers:**
- **CI Trigger:** Automatic build on push to `main` branch
- **PR Trigger:** Validation on pull requests
- **Manual Trigger:** On-demand deployment






### Deployment Strategy

**Blue-Green Deployment:**
- Zero-downtime deployments using Azure deployment slots
- Automatic rollback on failure
- Health checks before slot swap

**Environment Promotion:**
```
Development → Staging → Production
     ↓            ↓          ↓
  Auto Deploy  Manual Gate  Manual Gate
```



## Contact

**Shashank** - [@Shashank7708](https://github.com/Shashank7708)

---
