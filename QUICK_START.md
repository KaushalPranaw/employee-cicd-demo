# 🚀 CI/CD Setup - Quick Summary

## What Was Done ✅

### 1. **Environment-Specific Configuration Files**
Created 3 new property files for different environments:
- `application-dev.properties` - Dev config (DEBUG logging, auto-update DDL)
- `application-qa.properties` - QA config (INFO logging, validate DDL)
- `application-prod.properties` - Prod config (WARN logging, validate DDL)

### 2. **Maven Profiles**
Added to `pom.xml`:
- `dev` profile (default)
- `qa` profile
- `prod` profile

Build commands:
```bash
mvn clean package -Pdev   # Build for Dev
mvn clean package -Pqa    # Build for QA
mvn clean package -Pprod  # Build for Prod
```

### 3. **GitHub Actions Workflow**
Updated `.github/workflows/manual-deploy.yml`:
- ✅ **CI Stage**: Always runs (Build & Test)
- 🟢 **DEV Deploy**: Direct (no approval)
- 🟡 **QA Deploy**: Requires manual approval
- 🔴 **PROD Deploy**: Requires senior approval

### 4. **Documentation**
Created 3 comprehensive guides:
- `CD_PIPELINE_GUIDE.md` - Complete pipeline overview
- `GITHUB_ENVIRONMENTS_SETUP.md` - Step-by-step GitHub setup
- `LOCAL_DEVELOPMENT_GUIDE.md` - Local development instructions

---

## How to Deploy 🎯

### Option 1: Deploy to DEV (Instant) ✅
```
1. Go to GitHub → Actions
2. Click "Manual CI/CD Deploy"
3. Click "Run workflow"
4. Select: dev
5. CI runs → Deploys immediately
Done! 🎉
```

### Option 2: Deploy to QA (Requires Approval) 🟡
```
1. Go to GitHub → Actions
2. Click "Manual CI/CD Deploy"
3. Click "Run workflow"
4. Select: qa
5. CI runs → Waits for approval
6. Team gets notified 📧
7. QA lead clicks "Approve and deploy"
8. QA deployment starts
Done! 🎉
```

### Option 3: Deploy to PROD (Requires Senior Approval) 🔴
```
1. Go to GitHub → Actions
2. Click "Manual CI/CD Deploy"
3. Click "Run workflow"
4. Select: prod
5. CI runs → Waits for approval
6. Seniors get notified 📧
7. Lead clicks "Approve and deploy"
8. PROD deployment starts
Done! 🎉
```

---

## What You Need to Configure 🔧

### In GitHub Repository Settings

#### 1. Create GitHub Environments
Go to **Settings → Environments**

**Environment: dev** (No approval needed)
- No special setup required

**Environment: qa** (Manual approval)
- Required reviewers: 1-2 QA team members
- Add secret: `DB_PASSWORD_QA`

**Environment: production** (Senior approval)
- Required reviewers: 2+ technical leads
- Add secret: `DB_PASSWORD_PROD`
- Limit deployments to: `main` branch only

#### 2. Set GitHub Secrets
Go to **Settings → Secrets and variables → Actions**

Add these secrets:
- `RENDER_DEV_HOOK` → Your Render webhook URL for DEV
- `RENDER_QA_HOOK` → Your Render webhook URL for QA
- `RENDER_PROD_HOOK` → Your Render webhook URL for PROD

---

## File Structure 📁

```
employee-service/
├── .github/
│   └── workflows/
│       └── manual-deploy.yml ← Updated with approvals
├── src/
│   └── main/
│       └── resources/
│           ├── application.properties ← Main config (DEV default)
│           ├── application-dev.properties ← NEW
│           ├── application-qa.properties ← NEW
│           └── application-prod.properties ← NEW
├── pom.xml ← Updated with profiles
├── CD_PIPELINE_GUIDE.md ← NEW Documentation
├── GITHUB_ENVIRONMENTS_SETUP.md ← NEW Setup guide
└── LOCAL_DEVELOPMENT_GUIDE.md ← NEW Local dev guide
```

---

## Key Points 🔑

| Feature | Dev | QA | Prod |
|---------|-----|-----|------|
| **Approval Required** | ❌ | ✅ | ✅ |
| **Deployment Speed** | Instant | ⏳ Manual | ⏳ Manual |
| **Logging Level** | DEBUG | INFO | WARN |
| **DDL Mode** | UPDATE | VALIDATE | VALIDATE |
| **Reviewers Needed** | 0 | 1-2 | 2+ |
| **Branch Allowed** | Any | Any | main only |

---

## Local Development 💻

### Build
```bash
# Dev (default)
mvn clean package -Pdev

# QA
mvn clean package -Pqa

# Prod
mvn clean package -Pprod
```

### Run Locally
```bash
# Dev
mvn spring-boot:run

# QA
mvn spring-boot:run -Pqa

# Prod
mvn spring-boot:run -Pprod
```

### Test
```bash
mvn clean test
```

---

## Pipeline Flow 🔄

```
┌─────────────────────┐
│  You Click Deploy   │
│  (Select Env)       │
└──────────┬──────────┘
           │
           ▼
┌──────────────────────────────┐
│ CI Stage - Always First       │
│ ✓ Checkout                    │
│ ✓ Java 21 Setup              │
│ ✓ Build & Test (mvn test)    │
│ ✓ Package (mvn package)       │
│ ✓ Upload artifacts            │
└──────────┬───────────────────┘
           │
      CI PASSED?
       /    \
      /      \
    YES      NO ❌ STOP (Fix & retry)
     │
  ┌──┴──┐
  │     │
▼      ▼
DEV   QA/PROD?
 │      │
 │      ▼
 │   WAIT FOR APPROVAL ⏳
 │      │
 │      ▼ (After approval click)
 │   APPROVED?
 │    /    \
 │   /      \
 │ YES      NO ❌ STOP
 │  │
 ▼  ▼
DEPLOY to environment
     │
     ▼
✅ DONE
```

---

## Common Commands 📝

### Build for Env
```bash
mvn clean package -Pdev   # DEV
mvn clean package -Pqa    # QA
mvn clean package -Pprod  # PROD
```

### Run for Env
```bash
mvn spring-boot:run           # DEV (default)
mvn spring-boot:run -Pqa      # QA
mvn spring-boot:run -Pprod    # PROD
```

### Test
```bash
mvn test              # Run all tests
mvn test -DskipTests  # Skip tests
```

---

## Checklist Before First Deploy ✔️

- [ ] 3 property files created (dev, qa, prod)
- [ ] Maven profiles added to pom.xml
- [ ] Workflow file updated with approvals
- [ ] GitHub "dev" environment created
- [ ] GitHub "qa" environment created (with reviewers + secret)
- [ ] GitHub "production" environment created (with 2+ reviewers + secret)
- [ ] GitHub secrets added (RENDER_*_HOOK)
- [ ] Tests pass: `mvn clean test`
- [ ] Build succeeds: `mvn clean package`
- [ ] Local run works: `mvn spring-boot:run`
- [ ] Code pushed to GitHub

---

## Testing the Pipeline 🧪

### Test 1: DEV (Instant)
1. Select "dev" → CI runs → Auto-deploys ✅

### Test 2: QA (With Approval)
1. Select "qa" → CI runs → Waits for approval ⏳
2. (You) Approve → QA deploys ✅

### Test 3: PROD (With Senior Approval)
1. Select "prod" → CI runs → Waits for approval ⏳
2. (Senior) Approves → PROD deploys ✅

---

## Troubleshooting 🔧

| Problem | Solution |
|---------|----------|
| Deployment stuck waiting | Check if you're a reviewer in GitHub environment |
| Wrong profile applied | Verify `spring.profiles.active` in properties |
| Build fails | Run `mvn clean install` locally first |
| Database error | Ensure PostgreSQL is running with correct credentials |
| Tests failing | Run `mvn test -X` for verbose output |

---

## Next Steps 🎯

1. **Configure GitHub Environments** ← Most important!
   - Read: `GITHUB_ENVIRONMENTS_SETUP.md`
   - Setup: dev, qa, production environments
   - Add: Reviewers for qa and production

2. **Test Locally**
   - Read: `LOCAL_DEVELOPMENT_GUIDE.md`
   - Run: `mvn clean test`
   - Run: `mvn spring-boot:run`

3. **Configure Secrets**
   - Add: `RENDER_DEV_HOOK`
   - Add: `RENDER_QA_HOOK`
   - Add: `RENDER_PROD_HOOK`
   - Add: `DB_PASSWORD_QA` (to qa env)
   - Add: `DB_PASSWORD_PROD` (to prod env)

4. **Test Workflow**
   - Go to Actions
   - Run workflow with dev (should deploy instant)
   - Run workflow with qa (should wait for approval)
   - Run workflow with prod (should wait for approval)

5. **Document in Team**
   - Share: `CD_PIPELINE_GUIDE.md`
   - Share: Approval process
   - Share: When to use each environment

---

## Resources 📚

- [CD Pipeline Guide](./CD_PIPELINE_GUIDE.md)
- [GitHub Environments Setup](./GITHUB_ENVIRONMENTS_SETUP.md)
- [Local Development Guide](./LOCAL_DEVELOPMENT_GUIDE.md)

---

**Status:** ✅ Complete  
**Last Updated:** May 16, 2026  
**Ready for:** First deployment

