# 🚀 CI/CD Pipeline Documentation

## Overview
This document describes the complete Continuous Integration/Continuous Deployment (CI/CD) pipeline for the Employee Service application with **manual deployments to three environments: Dev, QA, and Prod**.

---

## 📋 Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  GitHub Actions Workflow                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  1️⃣  CI STAGE (Always Runs First)                           │
│     ├── Checkout code                                        │
│     ├── Setup Java 21                                        │
│     ├── Build & Test (mvn clean test)                       │
│     ├── Package (mvn clean package -DskipTests)             │
│     └── Upload artifacts                                     │
│                                                               │
│  2️⃣  DEPLOYMENT STAGE (Manual Selection)                    │
│     ├── Deploy to DEV      → ✅ Direct (No Approval)        │
│     ├── Deploy to QA       → ⏳ Requires Manual Approval     │
│     └── Deploy to PROD     → 🔒 Requires Senior Approval    │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 Deployment Environments

### 1. **DEV Environment** 🟢
- **Approval Required:** ❌ NO
- **Action:** Deploys immediately when selected
- **Purpose:** For development and testing
- **Database:** Local/Dev PostgreSQL
- **URL:** https://dev-employee-service.render.com

### 2. **QA Environment** 🟡
- **Approval Required:** ✅ YES (Manual Approval)
- **Action:** Waits for manual approval before deployment
- **Purpose:** For quality assurance and testing
- **Database:** QA PostgreSQL
- **URL:** https://qa-employee-service.render.com

### 3. **PROD Environment** 🔴
- **Approval Required:** ✅ YES (Senior Approval)
- **Action:** Waits for manual approval before deployment
- **Purpose:** Production use - Live environment
- **Database:** Production PostgreSQL
- **URL:** https://employee-service.render.com

---

## 🔧 How to Use

### Step 1: Trigger the Workflow
1. Go to **GitHub Repo → Actions Tab**
2. Click on **Manual CI/CD Deploy** workflow
3. Click **Run workflow** button

### Step 2: Select Environment
A dropdown menu appears with three options:
- **dev** - Direct deployment (no approval needed)
- **qa** - Requires manual approval
- **prod** - Requires senior approval

### Step 3: Workflow Execution

#### Option A: Deploy to DEV ✅
```
1. Select "dev"
2. CI runs automatically (Build & Test)
3. If CI passes → Deploy to DEV (immediate)
4. Done! 🎉
```

#### Option B: Deploy to QA 🟡
```
1. Select "qa"
2. CI runs automatically (Build & Test)
3. If CI passes → Job waits for approval
4. GitHub sends approval notification
5. Authorized person clicks "Review deployments"
6. Clicks "Approve and deploy"
7. QA deployment starts
8. Done! 🎉
```

#### Option C: Deploy to PROD 🔴
```
1. Select "prod"
2. CI runs automatically (Build & Test)
3. If CI passes → Job waits for approval
4. GitHub sends approval notification
5. Senior/Lead person clicks "Review deployments"
6. Reviews deployment details
7. Clicks "Approve and deploy"
8. PROD deployment starts
9. Done! 🎉
```

---

## 📁 Environment-Specific Configuration Files

### Main Configuration
- **File:** `src/main/resources/application.properties`
- **Role:** Default configuration (Dev profile)
- **Key Settings:**
  - `spring.profiles.active=dev` (default)
  - Database URL for DEV
  - Logging level: DEBUG

### Dev Environment
- **File:** `src/main/resources/application-dev.properties`
- **Activated When:** `spring.profiles.active=dev`
- **Database:** Local PostgreSQL or DEV server
- **Logging:** DEBUG (verbose)
- **DDL:** `update` (schema auto-update)

### QA Environment
- **File:** `src/main/resources/application-qa.properties`
- **Activated When:** `spring.profiles.active=qa`
- **Database:** QA PostgreSQL server
- **Logging:** INFO
- **DDL:** `validate` (schema validation only)
- **Password:** Uses environment variable `DB_PASSWORD_QA`

### Prod Environment
- **File:** `src/main/resources/application-prod.properties`
- **Activated When:** `spring.profiles.active=prod`
- **Database:** Production PostgreSQL server
- **Logging:** WARN (limited)
- **DDL:** `validate` (no schema changes)
- **Password:** Uses environment variable `DB_PASSWORD_PROD`

---

## 🔑 Required GitHub Secrets

You need to configure the following secrets in your GitHub repository settings:

### Settings → Secrets and variables → Actions

| Secret Name | Purpose | Example |
|---|---|---|
| `RENDER_DEV_HOOK` | Webhook to trigger DEV deployment | `https://api.render.com/deploy/...` |
| `RENDER_QA_HOOK` | Webhook to trigger QA deployment | `https://api.render.com/deploy/...` |
| `RENDER_PROD_HOOK` | Webhook to trigger PROD deployment | `https://api.render.com/deploy/...` |
| `DB_PASSWORD_QA` | Database password for QA | (provided by DevOps) |
| `DB_PASSWORD_PROD` | Database password for PROD | (provided by DevOps) |

---

## 📦 Maven Profiles

You can also build locally with specific profiles:

### Build for DEV (Default)
```bash
mvn clean package -Pdev
```

### Build for QA
```bash
mvn clean package -Pqa
```

### Build for PROD
```bash
mvn clean package -Pprod
```

---

## 🛡️ Security & Best Practices

1. **Environment Secrets:**
   - Database credentials are stored as GitHub Secrets
   - Never commit credentials to the repository
   - Use environment variables in CI/CD pipeline

2. **Approvals:**
   - QA requires team approval (code reviewers/QA leads)
   - Prod requires senior approval (leads/architects)
   - Approval logs are tracked in GitHub

3. **Validation Modes:**
   - DEV: `ddl-auto=update` (flexible for development)
   - QA: `ddl-auto=validate` (validates schema only)
   - PROD: `ddl-auto=validate` (strict - no schema changes)

4. **Logging Levels:**
   - DEV: DEBUG (see everything)
   - QA: INFO (important info only)
   - PROD: WARN (warnings and errors only)

---

## 🔄 Workflow File Location

**File:** `.github/workflows/manual-deploy.yml`

This file contains:
- ✅ CI jobs (Build & Test)
- 🟢 DEV deployment (direct)
- 🟡 QA deployment (requires approval)
- 🔴 PROD deployment (requires approval)

---

## ➕ Adding New Environments

To add a new environment (e.g., Staging):

### 1. Create properties file
```bash
touch src/main/resources/application-staging.properties
```

### 2. Add content similar to QA/Prod

### 3. Add Maven profile to pom.xml
```xml
<profile>
    <id>staging</id>
    <properties>
        <app.profile>staging</app.profile>
    </properties>
    <!-- ... -->
</profile>
```

### 4. Add job to workflow file
```yaml
deploy-staging:
    needs: ci
    if: ${{ github.event.inputs.environment == 'staging' }}
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging-employee-service.render.com
    # ... rest of the configuration
```

### 5. Add to workflow_dispatch options
```yaml
options:
  - dev
  - qa
  - staging
  - prod
```

---

## 📊 Deployment Checklist

Before deploying to each environment:

### DEV Deployment ✅
- [ ] CI passed (Build & Test successful)
- [ ] Code is committed and pushed

### QA Deployment 📋
- [ ] CI passed
- [ ] Code reviewed (if applicable)
- [ ] Get approval from QA lead/team
- [ ] Verify QA database credentials are set

### PROD Deployment 🔒
- [ ] CI passed
- [ ] Code reviewed and approved
- [ ] QA testing completed
- [ ] Get approval from team lead/architect
- [ ] Verify all secrets are configured
- [ ] Have rollback plan ready

---

## 🐛 Troubleshooting

### Build Fails
- Check Java version compatibility (Java 21 required)
- Run locally: `mvn clean test`
- Check dependencies in `pom.xml`

### Deployment Fails
- Verify webhook URLs in GitHub secrets
- Check database credentials in environment-specific properties
- Verify Render.com service is running

### Approval Not Working
- Ensure GitHub environment is configured for the job
- Check repository settings → Environments
- Verify user has appropriate role/permissions

### Wrong Profile Applied
- Check `spring.profiles.active` property
- Verify Maven profile is activated correctly
- Check environment variables in GitHub Actions

---

## 📚 Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Spring Boot Profiles](https://spring.io/blog/2015/02/17/spring-boot-management-endpoints-http-versus-jmx)
- [Maven Profiles](https://maven.apache.org/guides/introduction/introduction-to-profiles.html)
- [Render.com Deployment](https://render.com/docs)

---

**Last Updated:** May 16, 2026

