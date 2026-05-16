# 🔐 GitHub Environments Setup Guide

## Configuration Steps

### 1. Navigate to Repository Settings

1. Go to your GitHub repository
2. Click on **Settings** (top right)
3. Click on **Environments** (left sidebar under "Code and automation")

---

## 2. Create "dev" Environment

### a. Create New Environment
- Click **New environment**
- Enter name: `dev`
- Click **Configure environment**

### b. Deployment branches restriction
- Select: **Deployment branches and tags (recommended)**
- Rule: **All branches** (or specific branch filter)

### c. Environment secrets (Optional for DEV)
- No required secrets for DEV (uses defaults)

### d. Protection rules
- ❌ **Require reviewers** (Optional - not needed for DEV)
- ❌ **Require status checks to pass**

---

## 3. Create "qa" Environment (With Approval)

### a. Create New Environment
- Click **New environment**
- Enter name: `qa`
- Click **Configure environment**

### b. Deployment branches restriction
- Select: **Deployment branches and tags (recommended)**
- Rule: **All branches**

### c. Environment secrets
Add the following environment variables:

| Variable Name | Value | Type |
|---|---|---|
| `DB_PASSWORD_QA` | Your QA DB password | Secret |

**Steps:**
- Click **Add secret**
- Name: `DB_PASSWORD_QA`
- Value: (your QA database password)
- Click **Add secret**

### d. Protection rules 🔑
✅ **Enable required reviewers:**
- Click checkbox for "Require reviewers"
- Number of required reviewers: **1** (or more)
- Reviewers: Add QA team members or leads
  - Click **Add reviewer**
  - Select team members who can approve QA deployments

✅ **Optional - Require status checks to pass:**
- This ensures CI tests passed before allowing approval

### e. Deployment branches
- Keep as **All branches** or restrict to `main`/`develop`

---

## 4. Create "production" Environment (With Strict Approval)

### a. Create New Environment
- Click **New environment**
- Enter name: `production`
- Click **Configure environment**

### b. Deployment branches restriction
- Select: **Deployment branches and tags (recommended)**
- Rule: **Selected branches and tags** (Recommended for PROD)
- Specify: `main` only (or your production branch)

### c. Environment secrets
Add the following environment variables:

| Variable Name | Value | Type |
|---|---|---|
| `DB_PASSWORD_PROD` | Your PROD DB password | Secret |

**Steps:**
- Click **Add secret**
- Name: `DB_PASSWORD_PROD`
- Value: (your Production database password)
- Click **Add secret**

### d. Protection rules 🔐 (STRICT)
✅ **Enable required reviewers:**
- Click checkbox for "Require reviewers"
- Number of required reviewers: **2** or more (for PROD)
- Reviewers: Add senior team members, tech leads, architects
  - Click **Add reviewer**
  - Options:
    - A specific team (e.g., `@company/engineering-leads`)
    - Specific users (e.g., `@senior-dev`, `@architect`)

✅ **Require status checks to pass:**
- Click checkbox
- This ensures CI pipeline passed

✅ **Custom deployment branches:**
- Limit deployments to `main` branch or specific tags only

---

## Summary Table

| Environment | Reviewers Required | Min. Reviewers | Protection | Deploy Branch |
|---|---|---|---|---|
| **dev** | ❌ NO | 0 | Low | All |
| **qa** | ✅ YES | 1-2 | Medium | All |
| **production** | ✅ YES | 2+ | High | main only |

---

## How Approvals Work in Action

### Workflow Triggered
When you run the workflow and select an environment:

```
1. CI stage runs (always)
   ↓
2. If successful AND environment is QA/PROD:
   → GitHub pauses the job
   → Sends notifications to required reviewers
   → Job waits for approval
   ↓
3. Reviewers get notified via:
   - Email notification
   - GitHub web notification
   - (Optional) Slack/Teams webhook
   ↓
4. Reviewer clicks "Review deployments" link
   → Reviews deployment details
   → Sees: Environment, Branch, Commit info
   → Can "Approve and Deploy" or "Reject"
   ↓
5. If Approved:
   → Job continues to deployment step
   → Sends webhook to Render.com
   → Application deploys
   ↓
6. Logs show:
   → Approved by: username
   → Approval timestamp
   → Deployment started/completed
```

---

## Testing the Setup

### Test 1: Verify DEV Deployment (No Approval)
1. Go to Actions
2. Click "Manual CI/CD Deploy"
3. Click "Run workflow"
4. Select: `dev`
5. Expected: Deploys immediately after CI passes ✅

### Test 2: Verify QA Approval Gate
1. Go to Actions
2. Click "Manual CI/CD Deploy"
3. Click "Run workflow"
4. Select: `qa`
5. Expected:
   - CI runs ✅
   - Job pauses ⏳
   - Approval notifications sent 📧
   - Shows "Awaiting approval" status
   - (Your QA reviewer can click to approve)

### Test 3: Verify PROD Approval Gate
1. Go to Actions
2. Click "Manual CI/CD Deploy"
3. Click "Run workflow"
4. Select: `prod`
5. Expected:
   - CI runs ✅
   - Job pauses ⏳
   - Approval notifications to seniors 📧
   - Shows "Awaiting approval" status
   - (Your senior reviewer can click to approve)

---

## Viewing Approvals

### Location 1: Actions Tab
1. Go to **Actions**
2. Click the running workflow
3. Scroll down to deployment job
4. Click **Review deployments**
5. See approval options

### Location 2: Pull Requests (if linked)
1. Go to **Pull Requests**
2. Click the PR that triggered deployment
3. Scroll to **Deployments** section
4. Click **Review and approve**

---

## Managing Environment Reviewers

### To Add Reviewers Later
1. Go to **Settings → Environments**
2. Click the environment name
3. Scroll to **Required reviewers**
4. Click **Add reviewers**
5. Select users or teams

### To Modify Team Reviewers
1. In required reviewers section
2. Type team name with `@` symbol
3. Example: `@mycompany/backend-team`
4. Click **Add reviewer**

---

## Environment Variables vs Secrets

| Type | Visibility | Access | Use For |
|---|---|---|---|
| **Variable** | Visible in logs | Build & Deploy jobs | Non-sensitive: app name, port, URL |
| **Secret** | Hidden in logs | Build & Deploy jobs | Sensitive: passwords, tokens, keys |

For this setup:
- ✅ Store `DB_PASSWORD_QA` as **Secret**
- ✅ Store `DB_PASSWORD_PROD` as **Secret**
- ✅ Store `RENDER_*_HOOK` as **Secret** (already in repo secrets)

---

## Complete Environment Checklist

### Before Your First Deployment

- [ ] Create `dev` environment
- [ ] Create `qa` environment
- [ ] Add `DB_PASSWORD_QA` secret to `qa` environment
- [ ] Add 1-2 reviewers to `qa` environment
- [ ] Create `production` environment
- [ ] Add `DB_PASSWORD_PROD` secret to `production` environment
- [ ] Add 2+ senior reviewers to `production` environment
- [ ] Restrict production deployments to `main` branch only
- [ ] Repository secrets configured:
  - [ ] `RENDER_DEV_HOOK`
  - [ ] `RENDER_QA_HOOK`
  - [ ] `RENDER_PROD_HOOK`
- [ ] Test workflow with each environment

---

## Troubleshooting

### Q: Deployment doesn't wait for approval
**A:** Environment might not be configured correctly
- Verify environment name matches workflow file
- Check "Required reviewers" is enabled
- Make sure users/teams are added as reviewers

### Q: Notification not received
**A:** Check notification settings
1. Go to **Settings → Notifications**
2. Ensure "Actions" is selected
3. Check email settings

### Q: Can't see "Review deployments" button
**A:** 
- You might not be a required reviewer
- CI might not have passed yet
- Wrong branch restrictions

### Q: Wrong password being used
**A:**
- Verify environment secret is set correctly
- Check spelling matches in workflow file
- Test with: `echo ${{ secrets.DB_PASSWORD_QA }}`

---

## Reference

- [GitHub Environments Documentation](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [Required Reviewers](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#required-reviewers)
- [GitHub Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)

---

**Last Updated:** May 16, 2026

