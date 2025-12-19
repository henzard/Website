---
description: "Deployment, CI/CD, and environment management for Firebase projects"
alwaysApply: false
globs: ["**/.github/**", "**/firebase.json", "**/.env*"]
---

# Deployment & CI/CD Standards

## 🎯 Core Principles

- ✅ Automated deployments via GitHub Actions
- ✅ Separate staging and production environments
- ✅ Environment variables for secrets
- ✅ Preview deployments for PRs
- ✅ Rollback capability

---

## 🔧 Firebase Project Setup

### Multiple Environments

```bash
# Create projects
firebase projects:create myapp-staging
firebase projects:create myapp-production

# Add aliases
firebase use --add myapp-staging --alias staging
firebase use --add myapp-production --alias production
```

### .firebaserc

```json
{
  "projects": {
    "default": "myapp-staging",
    "staging": "myapp-staging",
    "production": "myapp-production"
  }
}
```

---

## 🔐 Environment Variables

### .env Files

```bash
# .env.local (development - gitignored)
VITE_FIREBASE_API_KEY=dev_key
VITE_FIREBASE_PROJECT_ID=myapp-dev
VITE_API_URL=http://localhost:5001

# .env.staging
VITE_FIREBASE_API_KEY=staging_key
VITE_FIREBASE_PROJECT_ID=myapp-staging
VITE_API_URL=https://staging-api.myapp.com

# .env.production
VITE_FIREBASE_API_KEY=prod_key
VITE_FIREBASE_PROJECT_ID=myapp-production
VITE_API_URL=https://api.myapp.com
```

### .gitignore

```
# Environment files
.env.local
.env*.local
.env.production.local

# Keep templates
!.env.example
```

---

## 🚀 GitHub Actions

### Deploy to Staging (on push to main)

```yaml
# .github/workflows/deploy-staging.yml
name: Deploy to Staging

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
        env:
          VITE_FIREBASE_API_KEY: ${{ secrets.STAGING_FIREBASE_API_KEY }}
          VITE_FIREBASE_PROJECT_ID: ${{ secrets.STAGING_PROJECT_ID }}
      
      - name: Deploy to Firebase Staging
        uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: ${{ secrets.GITHUB_TOKEN }}
          firebaseServiceAccount: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_STAGING }}
          projectId: myapp-staging
          channelId: live
```

### Deploy to Production (manual)

```yaml
# .github/workflows/deploy-production.yml
name: Deploy to Production

on:
  workflow_dispatch:
    inputs:
      confirm:
        description: 'Type "deploy" to confirm'
        required: true

jobs:
  deploy:
    if: github.event.inputs.confirm == 'deploy'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
        env:
          VITE_FIREBASE_API_KEY: ${{ secrets.PROD_FIREBASE_API_KEY }}
          VITE_FIREBASE_PROJECT_ID: ${{ secrets.PROD_PROJECT_ID }}
      
      - name: Deploy to Firebase Production
        uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: ${{ secrets.GITHUB_TOKEN }}
          firebaseServiceAccount: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_PROD }}
          projectId: myapp-production
          channelId: live
```

### Preview Deployments (PRs)

```yaml
# .github/workflows/preview.yml
name: Preview Deploy

on: pull_request

jobs:
  preview:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install & Build
        run: |
          npm ci
          npm run build
      
      - name: Deploy Preview
        uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: ${{ secrets.GITHUB_TOKEN }}
          firebaseServiceAccount: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_STAGING }}
          projectId: myapp-staging
          # Creates unique preview URL
```

---

## 🔄 Rollback Procedure

```bash
# List recent deployments
firebase hosting:channel:list

# View deployment history
firebase hosting:releases:list --limit 5

# Rollback to previous version
firebase hosting:rollback
```

---

## 📋 Pre-Deployment Checklist

```
Before Staging
☐ All tests pass locally
☐ No console.log statements
☐ No hardcoded secrets
☐ Environment variables set
☐ Build completes without errors

Before Production
☐ Staging tested and approved
☐ Database migrations run (if any)
☐ Backup current production data
☐ Team notified of deployment
☐ Rollback plan ready

After Deployment
☐ Smoke test critical paths
☐ Monitor error logs
☐ Check performance metrics
☐ Verify analytics working
```

---

## ✅ Checklist

```
Environment Setup
☐ Staging project created
☐ Production project created
☐ Environment variables configured
☐ Secrets in GitHub repository

CI/CD
☐ Build workflow created
☐ Test workflow created
☐ Staging auto-deploy configured
☐ Production manual deploy configured
☐ Preview deployments for PRs

Security
☐ Service account keys as secrets
☐ No secrets in code
☐ Production requires approval
☐ Audit logs enabled

IF ANY ☐ UNCHECKED → Fix before deploying
```

---

**Remember: "Automate deployments. Manual processes cause mistakes."** 🚀✨

