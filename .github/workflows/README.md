# NeoLoad GitHub Integration Setup

## Prerequisites
1. **NeoLoad Web Account**: Sign up at https://neoload.saas.neotys.com/
2. **API Token**: Generate from NeoLoad Web (Profile > API Token)

## Setup Steps

### 1. Add NeoLoad Token to GitHub Secrets
1. Go to your GitHub repository
2. Navigate to **Settings** > **Secrets and variables** > **Actions**
3. Click **New repository secret**
4. Name: `NEOLOAD_TOKEN`
5. Value: Your NeoLoad Web API token
6. Click **Add secret**

### 2. Run Tests Manually
1. Go to **Actions** tab in your GitHub repository
2. Select **NeoLoad Performance Test** workflow
3. Click **Run workflow**
4. Optionally configure:
   - Test scenario name
   - Test duration
5. Click **Run workflow**

### 3. View Results
- Check the workflow run logs for test execution details
- Download artifacts (JUnit XML results) from the workflow run
- View detailed reports in NeoLoad Web dashboard

## Workflow Features
- ✅ Manual trigger (workflow_dispatch)
- ✅ Configurable test parameters
- ✅ Automatic project upload
- ✅ JUnit test results export
- ✅ Artifact upload for test results

## Customization
Edit `.github/workflows/neoload-test.yml` to:
- Change triggers (e.g., on push, PR, schedule)
- Modify test duration and scenarios
- Add environment variables
- Configure load zones
- Set SLA thresholds
