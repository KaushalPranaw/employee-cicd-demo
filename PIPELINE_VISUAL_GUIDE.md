# 📊 CI/CD Pipeline Visual Reference

## Pipeline Architecture Diagram

```
╔════════════════════════════════════════════════════════════════════════════╗
║                     GITHUB ACTIONS CI/CD PIPELINE                         ║
║                                                                             ║
║  Manual Trigger: Select Environment (dev / qa / prod)                     ║
║  Webhook: You click ▶️ Run Workflow                                        ║
╚════════════════════════════════════════════════════════════════════════════╝
                                    ▼
        ┌───────────────────────────────────────────────────────────┐
        │  🔨 CI STAGE: Always Runs First                           │
        ├───────────────────────────────────────────────────────────┤
        │  1. 📥 Checkout code                                      │
        │  2. ☕ Setup Java 21                                      │
        │  3. 🧪 Build & Test          → mvn clean test            │
        │  4. 📦 Package JAR           → mvn clean package         │
        │  5. 📤 Upload artifacts                                   │
        └───────────────────────────────┬─────────────────────────┘
                                        │
                                        ▼
                                ✓ Tests Pass?
                              /                \
                            YES              NO
                           /                   \
                          ▼                     ▼
                      Continue          ❌ Pipeline Fails
                          │                   (Fix & Retry)
                          │
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
      dev?           qa?              prod?
      (Yes)          (Yes)            (Yes)
      │              │                │
      ▼              ▼                ▼
╔═════════════════╗ ╔═════════════╗ ╔══════════════════╗
║ 🟢 DEV DEPLOY  ║ ║ 🟡 QA DEPLOY ║ ║ 🔴 PROD DEPLOY  ║
╠═════════════════╣ ╠═════════════╣ ╠══════════════════╣
║                 ║ ║             ║ ║                  ║
║ ⚡ INSTANT    ║ ║ ⏳ WAITING  ║ ║ 🔒 RESTRICTED  ║
║ ✓ No approval ║ ║ ✓ 1-2 Votes ║ ║ ✓ 2+ Votes      ║
║ → Deploy to   ║ ║ → Team     ║ ║ → Seniors       ║
║   DEV env     ║ ║   Notified  ║ ║   Notified      ║
║                 ║ ║             ║ ║                  ║
└────────┬────────┘ └────┬────────┘ └────────┬─────────┘
         │               │                   │
         │          (Waiting for)       (Waiting for)
         │          Manual Approval     Senior Approval
         │               │                   │
         │               ▼                   ▼
         │          ┌──────────────────────────────────┐
         │          │ 👥 Notification Sent 📧          │
         │          │ GitHub → Email/Web/Slack        │
         │          │ "Review deployments" link       │
         │          │                                  │
         │          │ Reviewer clicks link             │
         │          │ ▼                                │
         │          │ 🔍 Review Details:              │
         │          │  - Branch                        │
         │          │  - Commit SHA                    │
         │          │  - Changed files                 │
         │          │                                  │
         │          │ ✓ Approve or ✗ Reject           │
         │          └──────────┬───────────────────────┘
         │                     │
         │              Approved?
         │             /        \
         │          YES          NO
         │          /              \
         │         ▼                ▼
         │    Continue         ❌ Rejected
         │         │           (Stop here)
         │         └──────┬─────────────────┘
         │                │
         ▼                ▼
    ╔──────────────────────────────────┐
    │ 🚀 DEPLOYMENT IN PROGRESS        │
    ├──────────────────────────────────┤
    │ 1. Prepare artifact              │
    │ 2. Send webhook to Render.com    │
    │ 3. Render triggers deployment    │
    │ 4. Application updates           │
    │ 5. Health check                  │
    └──────────┬───────────────────────┘
               │
               ▼
         ✅ Deployment Complete!
         - Approve notification sent
         - Approval logged with timestamp
         - User who approved recorded
         
```

---

## Environment Configuration Flow

```
┌─────────────────────────────────────────────────────────────┐
│         When Pipeline Starts                                │
│         (You select environment)                            │
└──────────────────┬──────────────────────────────────────────┘
                   │
              ┌────┴────┬─────────┬──────────┐
              │         │         │          │
              ▼         ▼         ▼          ▼
           Selected:  dev      qa        prod
              │         │         │          │
              │         │         │          │
        Property Files Applied:
              │         │         │          │
              ▼         ▼         ▼          ▼
         ┌─────┐   ┌─────┐   ┌──────┐   ┌──────┐
         │ DEV │   │ QA  │   │PROD  │   │BASE  │
         ├─────┤   ├─────┤   ├──────┤   ├──────┤
         │.    │   │.    │   │.     │   │.     │
         │ dev │   │ qa  │   │ prod │   │ props│
         │.    │   │.    │   │.     │   │.     │
         │ props   │props │   │props │   │props │
         └──┬──┘   └──┬──┘   └───┬──┘   └──────┘
            │        │          │       (Fallback)
            └────┬───┴──────────┘
                 │
                 ▼
        Spring Boot Active Profile:
         spring.profiles.active=dev
         spring.profiles.active=qa
         spring.profiles.active=prod
```

---

## Database Configuration Per Environment

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Database Configuration                           │
├─────────────────────────────────────────────────────────────────────┤

   🟢 DEV Environment
   ├─ URL: jdbc:postgresql://localhost:5432/employee_db_dev
   ├─ Username: employee_user_dev
   ├─ Password: dev_password_123 (stored in properties)
   ├─ DDL Mode: update ✓ (auto-creates/updates schema)
   ├─ Logging: DEBUG (verbose)
   └─ Purpose: Local development

   🟡 QA Environment
   ├─ URL: jdbc:postgresql://qa-postgres-server:5432/employee_db_qa
   ├─ Username: employee_user_qa
   ├─ Password: ${DB_PASSWORD_QA} (from GitHub secret)
   ├─ DDL Mode: validate ✓ (schema validation only)
   ├─ Logging: INFO (important only)
   └─ Purpose: Quality assurance testing

   🔴 PROD Environment
   ├─ URL: jdbc:postgresql://prod-postgres-server:5432/employee_db_prod
   ├─ Username: employee_user_prod
   ├─ Password: ${DB_PASSWORD_PROD} (from GitHub secret)
   ├─ DDL Mode: validate ✓ (no schema changes)
   ├─ Logging: WARN (errors only)
   └─ Purpose: Live production

└─────────────────────────────────────────────────────────────────────┘
```

---

## Maven Profile Build Flow

```
$ mvn clean package -Pdev    →  app-dev.jar
                                 Uses: spring.profiles.active=dev
                                 Config: application-dev.properties

$ mvn clean package -Pqa     →  app-qa.jar
                                 Uses: spring.profiles.active=qa
                                 Config: application-qa.properties

$ mvn clean package -Pprod   →  app-prod.jar
                                 Uses: spring.profiles.active=prod
                                 Config: application-prod.properties

$ mvn clean package          →  app.jar (default)
                                 Uses: spring.profiles.active=dev
                                 Config: application.properties + application-dev.properties
```

---

## GitHub Actions Matrix

```
┌─────────────────────────────────────────────────────────────────┐
│      Environment        │  Approval  │  Review  │  Branch       │
├─────────────────────────────────────────────────────────────────┤
│  dev                    │  ❌ NO     │  0       │  Any          │
│  qa                     │  ✅ YES    │  1-2     │  Any          │
│  production             │  ✅ YES    │  2+      │  main only    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Authorization Flow

```
Deployment Request
       │
       ▼
   ┌─────────────────┐
   │ CI Tests Pass?  │
   └────┬────────────┘
        │ NO  ❌
        ├──────────► STOP (Fix & Retry)
        │
        │ YES ✓
        ▼
   ┌─────────────────┐
   │ Environment?    │
   └────┬────────────┘
      / | \
    /   |   \
dev?  qa?  prod?
  │    │     │
  ▼    ▼     ▼
  ✓    ⏳    ⏳
DEPLOY WAIT WAIT
       │     │
       │     ▼
       │  ┌──────────────────────────┐
       │  │ Notify Required Reviewers│
       │  ├──────────────────────────┤
       │  │ QA: 1-2 votes needed    │
       │  │ PROD: 2+ votes needed   │
       │  └────────┬─────────────────┘
       │           │
       │           ▼
       │     ┌──────────────────┐
       │     │ Manual Approval? │
       │     └────┬─────────────┘
       │       /         \
       │      /           \
       │   YES ✓        NO ✗
       │    │             │
       ▼    ▼             ▼
       DEPLOY        REJECTED
         │
         ▼
   ✅ SUCCESS
```

---

## Deployment Scenario Examples

### Scenario 1: Deploy to DEV ✅
```
Timeline:
10:00 AM → You click "Run workflow" → select "dev"
10:01 AM → CI starts: Build & Test
10:02 AM → Tests pass ✓
10:02 AM → Deploy to DEV starts
10:03 AM → 🎉 DEV environment updated!

Approval chain? NONE (direct deployment)
```

### Scenario 2: Deploy to QA 🟡
```
Timeline:
10:05 AM → You click "Run workflow" → select "qa"
10:06 AM → CI starts: Build & Test
10:07 AM → Tests pass ✓
10:07 AM → ⏳ WAITING for approval
10:07 AM → GitHub sends email to QA team
10:08 AM → QA lead clicks approval link
10:08 AM → QA lead reviews commit
10:09 AM → QA lead clicks "Approve and deploy"
10:09 AM → Deployment to QA starts
10:10 AM → 🎉 QA environment updated!

Approval chain: 1-2 people from QA team
```

### Scenario 3: Deploy to PROD 🔴
```
Timeline:
11:00 AM → You click "Run workflow" → select "prod"
11:01 AM → CI starts: Build & Test
11:02 AM → Tests pass ✓
11:02 AM → ⏳ WAITING for approval (CRITICAL!)
11:02 AM → GitHub sends email to seniors
11:03 AM → Team Lead receives notification
11:10 AM → Team Lead reviews changes
11:11 AM → Tech Lead also reviews (2 required)
11:12 AM → Team Lead clicks "Approve"
11:12 AM → Tech Lead clicks "Approve"
11:12 AM → Deployment to PROD starts
11:13 AM → Health checks running
11:14 AM → 🎉 PROD environment updated!

Approval chain: 2+ senior team members
```

---

## Security Layers

```
LAYER 1: Code Review
├─ Local testing (mvn test)
└─ Git workflow (commits, pushes)

LAYER 2: Automated Testing
├─ mvn clean test
├─ Build verification
└─ Package validation

LAYER 3: Secret Management
├─ GitHub Secrets (encrypted)
├─ DB_PASSWORD_QA (secret storage)
├─ DB_PASSWORD_PROD (secret storage)
└─ API Webhooks (encrypted)

LAYER 4: Environment Approval Gates
├─ DEV: Direct (trusted environment)
├─ QA: 1-2 reviewers
└─ PROD: 2+ senior approvers

LAYER 5: Branch Protection
└─ PROD: main branch only (no hotfixes)

LAYER 6: Logging & Audit
├─ Who approved
├─ When approved
├─ What was deployed
└─ Deployment status
```

---

## Quick Decision Tree

```
Want to deploy?
    │
    ▼
Which environment?
    │
    ├─ Development/Testing?
    │  └─ Use DEV ✅ (instant)
    │
    ├─ QA testing?
    │  └─ Use QA 🟡 (need approval)
    │
    └─ Live users?
       └─ Use PROD 🔴 (senior approval only!)

Not sure? Always start with DEV to test!
```

---

## Rollback Path

```
If deployment goes wrong:

PROD Rollback:
  1. Code fix in dev branch
  2. Commit & push
  3. Re-run workflow
  4. Select PROD
  5. Wait for approvals
  6. New deployment with fix
  
  OR
  
  Contact deployment team for emergency rollback

Keep deployment history in GitHub Actions for tracking!
```

---

## Monitoring Deployments

```
Where to check status:
├─ GitHub: Actions tab → Click workflow → See status
├─ Render.com: Dashboard → See deployment logs
├─ App Health: Hit /health endpoint of deployed app
└─ Logs: Check application logs for errors

What to monitor:
├─ CI test results
├─ Approval wait times
├─ Deployment duration
├─ Application startup logs
└─ Database migration logs
```

---

**This visual guide helps understand:**
- ✅ How the pipeline flows
- ✅ Where approvals happen
- ✅ How configurations are applied
- ✅ Database setup per environment
- ✅ Security at each layer
- ✅ Real-world deployment scenarios


