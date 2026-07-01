# Complete NeoLoad GitHub Actions Setup Guide

This guide will help you set up NeoLoad performance testing with GitHub Actions on any system from scratch.

---

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [System Requirements](#system-requirements)
3. [Step 1: Install Required Software](#step-1-install-required-software)
4. [Step 2: Prepare Your NeoLoad Project](#step-2-prepare-your-neoload-project)
5. [Step 3: Create GitHub Repository](#step-3-create-github-repository)
6. [Step 4: Set Up GitHub Actions Workflows](#step-4-set-up-github-actions-workflows)
7. [Step 5: Configure NeoLoad Web](#step-5-configure-neoload-web)
8. [Step 6: Upload Project to NeoLoad Web](#step-6-upload-project-to-neoload-web)
9. [Step 7: Run Tests from GitHub](#step-7-run-tests-from-github)
10. [Updating Your Project](#updating-your-project)
11. [Troubleshooting](#troubleshooting)

---

## Prerequisites

- Windows, Mac, or Linux computer
- NeoLoad Desktop installed (for project creation and upload)
- NeoLoad Web account (https://neoload.saas.neotys.com/)
- GitHub account
- Git installed on your computer
- Basic understanding of command line operations

---

## System Requirements

### Software Needed:
- **NeoLoad Desktop** (version 2024.x or later)
- **Git** (for version control)
- **GitHub account** (free or paid)
- **Text editor** (VS Code, Notepad++, etc.)

### NeoLoad Web Account:
- Sign up at: https://neoload.saas.neotys.com/
- Free trial or subscription required
- Note your workspace name (usually "Default Workspace")

---

## Step 1: Install Required Software

### 1.1 Install Git

**Windows:**
- Download from: https://git-scm.com/download/win
- Run installer with default settings
- Verify: Open Command Prompt and run `git --version`

**Mac:**
- Install via Homebrew: `brew install git`
- Or download from: https://git-scm.com/download/mac
- Verify: Open Terminal and run `git --version`

**Linux:**
```bash
sudo apt-get update
sudo apt-get install git
git --version
```

### 1.2 Install NeoLoad Desktop

1. Download from Neotys website or your license portal
2. Install with default settings
3. Launch and activate with your license
4. Verify installation is working

### 1.3 Configure Git (First Time Only)

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

---

## Step 2: Prepare Your NeoLoad Project

### 2.1 Create or Open Your NeoLoad Project

1. **Open NeoLoad Desktop**
2. **Create a new project** or **open existing project**
3. **Create your test scenarios:**
   - Design → User Paths (create your test scripts)
   - Design → Population (define virtual users)
   - Design → Scenarios (create test scenarios like "scenario1")

### 2.2 Save and Note Project Location

1. **Save your project** (File → Save)
2. **Note the project folder location:**
   - Example: `C:\Users\YourName\Documents\NeoLoad Projects\MyProject\`
3. **Project folder should contain:**
   - `ProjectName.nlp` file
   - `config.zip` file
   - Other folders (recorded-requests, lib, etc.)

---

## Step 3: Create GitHub Repository

### 3.1 Create Repository on GitHub

1. **Login to GitHub:** https://github.com
2. **Click the "+" icon** (top-right) → "New repository"
3. **Repository settings:**
   - Name: `Neoload_E2E` (or your preferred name)
   - Description: "NeoLoad Performance Testing"
   - Visibility: Private (recommended) or Public
   - **DO NOT** check "Initialize with README"
4. **Click "Create repository"**
5. **Copy the repository URL** (e.g., `https://github.com/username/Neoload_E2E.git`)

### 3.2 Initialize Git in Your NeoLoad Project Folder

Open Command Prompt/Terminal and navigate to your NeoLoad project folder:

```bash
# Navigate to your project folder
cd "C:\Users\YourName\Documents\NeoLoad Projects\MyProject"

# Or on Mac/Linux
cd "/Users/YourName/Documents/NeoLoad Projects/MyProject"

# Initialize Git repository
git init

# Add all files
git add .

# Create initial commit
git commit -m "Initial NeoLoad project commit"

# Add GitHub remote (replace with your URL)
git remote add origin https://github.com/username/Neoload_E2E.git

# Rename branch to main
git branch -M main

# Push to GitHub
git push -u origin main
```

---

## Step 4: Set Up GitHub Actions Workflows

### 4.1 Create Workflows Folder

In your project folder, create the GitHub Actions directory structure:

```bash
mkdir -p .github/workflows
```

### 4.2 Create Workflow Files

Create these three workflow files in `.github/workflows/`:

#### File 1: `neoload-run-only.yml`

Create: `.github/workflows/neoload-run-only.yml`

```yaml
name: NeoLoad Run Test (Project Already Uploaded)

on:
  workflow_dispatch:
    inputs:
      test_settings_name:
        description: 'Test settings name (created in NeoLoad Web)'
        required: true
        default: 'GitHub-Test-Settings'

jobs:
  neoload-run:
    runs-on: ubuntu-latest

    steps:
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.x'

      - name: Install NeoLoad CLI
        run: |
          pip install neoload
          neoload --version

      - name: Login to NeoLoad Web
        env:
          NEOLOAD_TOKEN: ${{ secrets.NEOLOAD_TOKEN }}
        run: |
          echo "Logging into NeoLoad Web..."
          neoload login --url https://neoload-api.saas.neotys.com $NEOLOAD_TOKEN

          echo "Selecting workspace..."
          neoload workspaces use "Default Workspace"

          echo "Current status:"
          neoload status

      - name: List Available Tests
        env:
          NEOLOAD_TOKEN: ${{ secrets.NEOLOAD_TOKEN }}
        run: |
          echo "Available test settings:"
          neoload test-settings ls

      - name: Run Test
        env:
          NEOLOAD_TOKEN: ${{ secrets.NEOLOAD_TOKEN }}
        run: |
          TEST_NAME="${{ github.event.inputs.test_settings_name }}"

          echo "Running test: $TEST_NAME"
          neoload run \
            --name "GitHub-Run-${{ github.run_number }}" \
            --description "Triggered by ${{ github.actor }} on $(date)" \
            "$TEST_NAME"

      - name: Get Test Results
        if: always()
        env:
          NEOLOAD_TOKEN: ${{ secrets.NEOLOAD_TOKEN }}
        run: |
          echo "Listing test results..."
          neoload test-results ls

          echo "Getting latest result details..."
          neoload test-results use latest || true

          echo "Generating JUnit report..."
          neoload test-results --junit-file neoload-results.xml junitsla || echo "Failed to generate report"

          echo "Test logs URL:"
          neoload logs-url || echo "No logs available"

      - name: Upload Test Results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: neoload-results
          path: neoload-results.xml
          if-no-files-found: warn
```

#### File 2: `neoload-upload-project.yml`

Create: `.github/workflows/neoload-upload-project.yml`

```yaml
name: Reminder - Upload Project via NeoLoad Desktop

on:
  workflow_dispatch:
  push:
    branches:
      - main
    paths:
      - '*.nlp'
      - 'config.zip'
      - 'recorded-*/**'
      - 'custom-resources/**'
      - 'lib/**'

jobs:
  upload-reminder:
    runs-on: ubuntu-latest

    steps:
      - name: NeoLoad Project Upload Reminder
        run: |
          echo "================================================"
          echo "⚠️ NeoLoad Project Files Changed"
          echo "================================================"
          echo ""
          echo "Detected changes to NeoLoad project files"
          echo ""
          echo "📋 ACTION REQUIRED:"
          echo "================================================"
          echo ""
          echo "1. Open NeoLoad Desktop"
          echo "2. Open your project"
          echo "3. Go to: File → Upload to NeoLoad Web"
          echo "4. Select workspace: Default Workspace"
          echo "5. Click Upload"
          echo ""
          echo "================================================"
          echo "✅ After upload, run tests via:"
          echo "   Actions → 'NeoLoad Run Test'"
          echo "================================================"
```

### 4.3 Commit and Push Workflow Files

```bash
# Add workflow files
git add .github/workflows/

# Commit
git commit -m "Add NeoLoad GitHub Actions workflows"

# Push to GitHub
git push
```

---

## Step 5: Configure NeoLoad Web

### 5.1 Get NeoLoad Web API Token

1. **Login to NeoLoad Web:** https://neoload.saas.neotys.com/
2. **Click your profile icon** (top-right corner)
3. **Go to "API Token"** or "Account Settings"
4. **Click "Generate Token"** or "Create new token"
5. **Copy the token** (it will look like a long string of characters)
6. **⚠️ IMPORTANT:** Save this token securely - you won't see it again!

### 5.2 Verify Your Workspace Name

1. In NeoLoad Web, check the **workspace dropdown** (usually top-left)
2. Note the **exact name** (typically "Default Workspace")
3. If different, you'll need to update the workflows later

### 5.3 Add Token to GitHub Secrets

1. **Go to your GitHub repository**
2. **Click "Settings"** tab
3. **In left sidebar:** Click "Secrets and variables" → "Actions"
4. **Click "New repository secret"**
5. **Fill in:**
   - Name: `NEOLOAD_TOKEN`
   - Secret: Paste your NeoLoad API token
6. **Click "Add secret"**

**✅ Verification:**
- The secret should now appear as `NEOLOAD_TOKEN` in the list
- You won't be able to view the value again (this is normal)

---

## Step 6: Upload Project to NeoLoad Web

### 6.1 Upload via NeoLoad Desktop (First Time)

1. **Open NeoLoad Desktop**
2. **Open your project**
3. **Upload to NeoLoad Web:**
   - **File** → **Upload to NeoLoad Web**
   - Or look for a **cloud/upload icon** in the toolbar
4. **Select Workspace:** "Default Workspace"
5. **Click "Upload"**
6. **Wait for upload to complete** (progress bar will show)

### 6.2 Verify Upload in NeoLoad Web

1. **Go to NeoLoad Web:** https://neoload.saas.neotys.com/
2. **Navigate to "Projects"** section
3. **Verify your project appears** in the list
4. **Click on your project** to see details
5. **Check that scenarios are present**

### 6.3 Create Test Settings (One Time)

**Option A: Via NeoLoad Web UI (Recommended)**

1. In NeoLoad Web, go to your project
2. Click **"Tests"** or **"Create Test"**
3. **Configure test settings:**
   - Name: `GitHub-Test-Settings`
   - Scenario: Select your scenario (e.g., "scenario1")
   - Zone: Select `defaultzone` or available zone
   - Load Generators: Set to 1 or more
4. Click **"Save"**

**Option B: Via NeoLoad CLI (Alternative)**

```bash
# Install NeoLoad CLI
pip install neoload

# Login
neoload login --url https://neoload-api.saas.neotys.com YOUR_TOKEN_HERE

# Select workspace
neoload workspaces use "Default Workspace"

# Create test settings
neoload test-settings --zone defaultzone --lgs 1 --scenario "scenario1" create "GitHub-Test-Settings"
```

---

## Step 7: Run Tests from GitHub

### 7.1 Navigate to GitHub Actions

1. **Go to your GitHub repository**
2. **Click "Actions" tab** (top menu)
3. **You should see workflows:**
   - "NeoLoad Run Test (Project Already Uploaded)"
   - "Reminder - Upload Project via NeoLoad Desktop"

### 7.2 Run Your First Test

1. **Click** on **"NeoLoad Run Test (Project Already Uploaded)"**
2. **Click "Run workflow"** button (right side)
3. **Configure parameters:**
   - Branch: `main`
   - Test settings name: `GitHub-Test-Settings`
4. **Click green "Run workflow"** button
5. **Wait for workflow to start** (refreshes in a few seconds)

### 7.3 Monitor Test Execution

1. **Click on the running workflow** (shows as yellow/orange circle)
2. **Click on the job name** (e.g., "neoload-run")
3. **Watch the real-time logs** as steps execute
4. **Wait for completion** (green checkmark = success, red X = failure)

### 7.4 View Test Results

**In GitHub:**
1. Scroll down to **"Artifacts"** section
2. Download **"neoload-results"** (JUnit XML file)

**In NeoLoad Web:**
1. Go to https://neoload.saas.neotys.com/
2. Navigate to **"Results"** or **"Test Results"**
3. Find your test run (named "GitHub-Run-X")
4. Click to view detailed performance metrics

---

## Updating Your Project

### When You Make Changes to Your NeoLoad Scripts:

#### Step 1: Update in NeoLoad Desktop
1. Open your project in NeoLoad Desktop
2. Make your changes (modify scenarios, user paths, etc.)
3. Save the project

#### Step 2: Commit to GitHub
```bash
# Navigate to project folder
cd "path/to/your/project"

# Check what changed
git status

# Add changed files
git add config.zip YourProject.nlp

# Commit with descriptive message
git commit -m "Updated scenario1 - added new transaction"

# Push to GitHub
git push
```

#### Step 3: Upload to NeoLoad Web
1. In NeoLoad Desktop: **File → Upload to NeoLoad Web**
2. Select "Default Workspace"
3. Upload

#### Step 4: Run Tests
1. Go to GitHub Actions
2. Run "NeoLoad Run Test" workflow

---

## Troubleshooting

### Issue 1: "NEOLOAD_TOKEN not found" Error

**Symptoms:**
- Workflow fails with authentication error
- Message: "ERROR: NEOLOAD_TOKEN is not set!"

**Solution:**
1. Verify secret is created in GitHub:
   - Settings → Secrets and variables → Actions
   - Check `NEOLOAD_TOKEN` exists
2. If missing, add it following [Step 5.3](#53-add-token-to-github-secrets)
3. Re-run the workflow

---

### Issue 2: "No object associated to the name 'None'"

**Symptoms:**
- Test fails to start
- Error about name 'None' or missing object

**Causes:**
- Project not uploaded to NeoLoad Web
- Test settings don't exist
- Workspace not selected correctly

**Solutions:**

**A) Verify project is uploaded:**
1. Login to NeoLoad Web
2. Go to Projects
3. Check if your project is listed
4. If not, upload via NeoLoad Desktop

**B) Verify test settings exist:**
1. In NeoLoad Web, go to Tests
2. Check if "GitHub-Test-Settings" exists
3. If not, create it (see [Step 6.3](#63-create-test-settings-one-time))

**C) Check workspace name:**
1. In NeoLoad Web, verify exact workspace name
2. If different from "Default Workspace", update workflows:
   - Edit `.github/workflows/neoload-run-only.yml`
   - Change `neoload workspaces use "Default Workspace"` to your workspace name

---

### Issue 3: Workflow Not Visible in Actions Tab

**Symptoms:**
- Actions tab is empty or workflows don't appear

**Solutions:**

1. **Verify files are in correct location:**
   ```
   .github/
   └── workflows/
       ├── neoload-run-only.yml
       └── neoload-upload-project.yml
   ```

2. **Check files are pushed to main branch:**
   ```bash
   git status
   git log --oneline
   ```

3. **Verify YAML syntax:**
   - Use a YAML validator
   - Check for indentation errors
   - Ensure no tabs (use spaces only)

4. **Wait a few minutes:**
   - GitHub may take 1-2 minutes to detect new workflows
   - Refresh the Actions page

---

### Issue 4: "Test settings already exist" Error

**Symptoms:**
- Error creating test settings
- Message: "A test with the same name already exists"

**Solution:**
This is actually **OK**! The test settings already exist in NeoLoad Web.

**Action:** Run the test workflow directly - it will use the existing settings.

---

### Issue 5: Workspace Name Mismatch

**Symptoms:**
- Error: "Workspace not found"
- Login succeeds but workspace selection fails

**Solution:**

1. **Find your exact workspace name:**
   ```bash
   pip install neoload
   neoload login --url https://neoload-api.saas.neotys.com YOUR_TOKEN
   neoload workspaces ls
   ```

2. **Update workflow files:**
   - Edit `.github/workflows/neoload-run-only.yml`
   - Find line: `neoload workspaces use "Default Workspace"`
   - Replace with your exact workspace name

3. **Commit and push:**
   ```bash
   git add .github/workflows/
   git commit -m "Fix workspace name"
   git push
   ```

---

### Issue 6: Token Expired or Invalid

**Symptoms:**
- Authentication fails
- 401 Unauthorized error

**Solution:**

1. **Generate new token** in NeoLoad Web (see [Step 5.1](#51-get-neoload-web-api-token))
2. **Update GitHub secret:**
   - Go to repository Settings → Secrets → Actions
   - Click on `NEOLOAD_TOKEN`
   - Click "Update secret"
   - Paste new token
   - Click "Update secret"
3. **Re-run workflow**

---

### Issue 7: Test Runs But No Results

**Symptoms:**
- Workflow completes successfully
- No test results visible in NeoLoad Web

**Solutions:**

1. **Check test actually started:**
   - Look for "Test started" message in workflow logs
   - Note the test run ID

2. **Check NeoLoad Web:**
   - Results may take 1-2 minutes to appear
   - Go to "Results" or "Test Results" section
   - Filter by date/time

3. **Check load zone availability:**
   - Ensure load generators are available
   - Check NeoLoad Web infrastructure status

---

## Advanced Configuration

### Customizing Test Parameters

Edit `.github/workflows/neoload-run-only.yml` to add more input parameters:

```yaml
on:
  workflow_dispatch:
    inputs:
      test_settings_name:
        description: 'Test settings name'
        required: true
        default: 'GitHub-Test-Settings'
      duration:
        description: 'Test duration (e.g., 5m, 10m)'
        required: false
        default: '5m'
      users:
        description: 'Number of virtual users'
        required: false
        default: '10'
```

### Running Tests on Schedule

Add cron trigger to run tests automatically:

```yaml
on:
  workflow_dispatch:
  schedule:
    - cron: '0 2 * * *'  # Runs at 2 AM UTC daily
```

### Running Tests on Push/PR

Trigger tests when code changes:

```yaml
on:
  workflow_dispatch:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

---

## Best Practices

### 1. Project Management
- ✅ Keep NeoLoad project in a dedicated folder
- ✅ Use meaningful scenario names
- ✅ Document your test scenarios in README
- ✅ Version control all NeoLoad files (.nlp, config.zip)

### 2. Security
- ✅ Never commit API tokens to Git
- ✅ Use GitHub Secrets for sensitive data
- ✅ Use private repositories for proprietary tests
- ✅ Rotate API tokens periodically

### 3. Testing
- ✅ Test workflows manually before automating
- ✅ Start with short test durations
- ✅ Monitor NeoLoad Web quota/usage
- ✅ Review test results after each run

### 4. Maintenance
- ✅ Keep NeoLoad Desktop updated
- ✅ Update workflow actions periodically
- ✅ Document changes in commit messages
- ✅ Clean up old test results regularly

---

## Quick Reference Commands

### Git Commands
```bash
# Check status
git status

# Add files
git add .

# Commit
git commit -m "Your message"

# Push
git push

# Pull latest
git pull
```

### NeoLoad CLI Commands
```bash
# Install
pip install neoload

# Login
neoload login --url https://neoload-api.saas.neotys.com YOUR_TOKEN

# List workspaces
neoload workspaces ls

# Select workspace
neoload workspaces use "Default Workspace"

# List test settings
neoload test-settings ls

# List test results
neoload test-results ls
```

---

## Additional Resources

- **NeoLoad Documentation:** https://www.neotys.com/documents/doc/neoload/latest/
- **NeoLoad CLI Guide:** https://github.com/Neotys-Labs/neoload-cli
- **GitHub Actions Docs:** https://docs.github.com/en/actions
- **NeoLoad Web:** https://neoload.saas.neotys.com/

---

## Summary Checklist

Use this checklist when setting up on a new system:

- [ ] Git installed and configured
- [ ] NeoLoad Desktop installed
- [ ] NeoLoad Web account created
- [ ] NeoLoad project created with scenarios
- [ ] GitHub repository created
- [ ] Project files committed to GitHub
- [ ] Workflow files created and pushed
- [ ] NeoLoad API token generated
- [ ] Token added to GitHub Secrets
- [ ] Project uploaded to NeoLoad Web via Desktop
- [ ] Test settings created in NeoLoad Web
- [ ] First test run successfully from GitHub Actions
- [ ] Test results visible in NeoLoad Web

---

## Getting Help

If you encounter issues not covered in this guide:

1. **Check GitHub Actions logs** for detailed error messages
2. **Review NeoLoad Web** for project and test status
3. **Consult NeoLoad documentation** for product-specific issues
4. **Contact Neotys support** for license or account issues

---

**Document Version:** 1.0  
**Last Updated:** 2026-07-01  
**Tested With:** NeoLoad 2025.3.1, GitHub Actions (ubuntu-latest)

---

✅ **You're now ready to run NeoLoad performance tests via GitHub Actions!**
