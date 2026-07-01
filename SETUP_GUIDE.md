# NeoLoad GitHub Integration - Complete Setup Guide

This guide will help you set up NeoLoad performance testing with GitHub Actions on any system.

## Prerequisites

- Git installed on your system
- GitHub account
- NeoLoad project file (`.nlp`)
- NeoLoad Web account (sign up at https://neoload.saas.neotys.com/)

---

## Step 1: Create GitHub Actions Workflow Files

### 1.1 Create Directory Structure
In your NeoLoad project directory, create the workflows folder:

```bash
mkdir -p .github/workflows
```

### 1.2 Create Workflow File
Create a file named `.github/workflows/neoload-test.yml` with the following content:

```yaml
name: NeoLoad Performance Test

on:
  workflow_dispatch:
    inputs:
      test_name:
        description: 'Test scenario name'
        required: false
        default: 'default'
      duration:
        description: 'Test duration (e.g., 5m, 10m)'
        required: false
        default: '5m'

jobs:
  neoload-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Install NeoLoad CLI
        run: |
          pip install neoload
          neoload --version

      - name: Configure NeoLoad Web
        env:
          NEOLOAD_TOKEN: ${{ secrets.NEOLOAD_TOKEN }}
        run: |
          neoload login --url https://neoload-api.saas.neotys.com $NEOLOAD_TOKEN
          neoload workspaces use "Default Workspace"

      - name: Upload NeoLoad project
        run: |
          neoload project --path YOUR_PROJECT_NAME.nlp upload

      - name: Run NeoLoad test
        run: |
          neoload test-settings --scenario ${{ github.event.inputs.test_name }} patch
          neoload run \
            --name "GitHub-Run-${{ github.run_number }}" \
            --description "Triggered by ${{ github.actor }}" \
            --as-code YOUR_PROJECT_NAME.nlp

      - name: Generate test report
        if: always()
        run: |
          neoload test-results --junit-file neoload-results.xml junitsla
          neoload logs-url

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: neoload-results
          path: neoload-results.xml
```

**IMPORTANT:** Replace `YOUR_PROJECT_NAME.nlp` with your actual NeoLoad project filename (e.g., `Neoload_GIT_HUB.nlp`).

---

## Step 2: Initialize Git Repository (if not already done)

### 2.1 Initialize Git
```bash
git init
```

### 2.2 Add All Files
```bash
git add .
```

### 2.3 Commit Files
```bash
git commit -m "Add GitHub Actions workflow for NeoLoad performance testing"
```

---

## Step 3: Create GitHub Repository

### 3.1 Via GitHub Website:
1. Go to https://github.com
2. Click the **+** icon (top-right) → **New repository**
3. Enter repository name (e.g., `Neoload_E2E`)
4. Choose visibility (Public or Private)
5. **DO NOT** initialize with README, .gitignore, or license
6. Click **Create repository**
7. Copy the repository URL (e.g., `https://github.com/username/repo.git`)

### 3.2 Via GitHub CLI (Alternative):
```bash
gh repo create your-repo-name --public --source=. --remote=origin --push
```

---

## Step 4: Push to GitHub

### 4.1 Add Remote Repository
```bash
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
```

Replace `YOUR_USERNAME` and `YOUR_REPO_NAME` with your actual values.

### 4.2 Rename Branch to Main (if needed)
```bash
git branch -M main
```

### 4.3 Push to GitHub
```bash
git push -u origin main
```

---

## Step 5: Get NeoLoad Web API Token

### 5.1 Login to NeoLoad Web
1. Go to https://neoload.saas.neotys.com/
2. Sign in with your account

### 5.2 Generate API Token
1. Click on your **profile icon** (top-right)
2. Select **API Token** or **Account Settings**
3. Click **Generate Token** or **Create new token**
4. **Copy the token** (you won't be able to see it again!)

---

## Step 6: Add Token to GitHub Secrets

### 6.1 Navigate to Repository Settings
1. Go to your GitHub repository
2. Click **Settings** tab
3. In the left sidebar, click **Secrets and variables** → **Actions**

### 6.2 Create Secret
1. Click **New repository secret**
2. Name: `NEOLOAD_TOKEN`
3. Secret: Paste your NeoLoad API token
4. Click **Add secret**

---

## Step 7: Run NeoLoad Tests

### 7.1 Navigate to Actions Tab
1. Go to your GitHub repository
2. Click the **Actions** tab
3. You should see **NeoLoad Performance Test** workflow

### 7.2 Run Workflow Manually
1. Click on **NeoLoad Performance Test**
2. Click **Run workflow** button (right side)
3. (Optional) Configure parameters:
   - Test scenario name
   - Test duration
4. Click **Run workflow**

### 7.3 Monitor Test Execution
1. Click on the running workflow
2. Watch the progress in real-time
3. View logs for each step

### 7.4 Download Results
1. After workflow completes, scroll down to **Artifacts**
2. Download **neoload-results** (JUnit XML format)
3. View detailed reports in NeoLoad Web dashboard

---

## Troubleshooting

### Workflow Not Visible in Actions Tab
- Ensure `.github/workflows/neoload-test.yml` is pushed to the **main** branch
- Check file is in correct location: `.github/workflows/` (not `github/workflows/`)
- Refresh the Actions page
- Verify YAML syntax is correct

### Authentication Errors
- Verify `NEOLOAD_TOKEN` secret is created correctly
- Check token hasn't expired
- Ensure token has correct permissions

### Project Upload Fails
- Verify `.nlp` filename in workflow matches your actual file
- Ensure `.nlp` file is committed to repository
- Check NeoLoad Web workspace name is correct

### Test Execution Fails
- Review workflow logs for specific error messages
- Verify test scenario name exists in your project
- Check NeoLoad Web quota/limits

---

## Customization Options

### Change Trigger to Automatic

Replace the `on:` section to run on push:

```yaml
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
```

### Schedule Tests (Nightly)

```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # Runs at 2 AM UTC daily
```

### Add Multiple Scenarios

Modify the workflow to run multiple test scenarios in parallel or sequentially.

---

## File Checklist

Before pushing, ensure you have:
- [ ] `.github/workflows/neoload-test.yml` created
- [ ] NeoLoad project file (`.nlp`) in repository root
- [ ] Correct project filename in workflow YAML
- [ ] Git repository initialized
- [ ] GitHub remote added
- [ ] Files committed and pushed
- [ ] `NEOLOAD_TOKEN` secret created in GitHub

---

## Quick Command Reference

```bash
# Initialize and push to GitHub
git init
git add .
git commit -m "Add NeoLoad GitHub Actions workflow"
git remote add origin https://github.com/USERNAME/REPO.git
git branch -M main
git push -u origin main

# Update workflow after changes
git add .github/workflows/neoload-test.yml
git commit -m "Update NeoLoad workflow"
git push
```

---

## Support

- **NeoLoad Documentation**: https://www.neotys.com/documents/doc/neoload/latest/
- **NeoLoad CLI Guide**: https://github.com/Neotys-Labs/neoload-cli
- **GitHub Actions Docs**: https://docs.github.com/en/actions

---

## Summary

✅ Created workflow file in `.github/workflows/`  
✅ Initialized Git and pushed to GitHub  
✅ Generated NeoLoad API token  
✅ Added token to GitHub Secrets  
✅ Executed workflow from Actions tab  

Your NeoLoad performance tests are now automated via GitHub Actions!
