# NeoLoad Web Upload & Test Setup

Since running tests directly from local `.nlp` files is encountering issues, here's the recommended approach: **Upload your project to NeoLoad Web first**, then run tests from there.

## Option 1: Manual Upload (Recommended - One Time Setup)

### Step 1: Upload Project via NeoLoad Desktop
1. Open your project in **NeoLoad Desktop**
2. Go to **File** > **Upload to NeoLoad Web**
3. Select your workspace: "Default Workspace"
4. Upload the project (this includes all scenarios)

### Step 2: Verify in NeoLoad Web
1. Login to https://neoload.saas.neotys.com/
2. Go to **Projects** section
3. Verify "Neoload_GIT_HUB" project is uploaded
4. Check that "scenario1" is available

### Step 3: Run GitHub Workflow
1. Go to Actions → **NeoLoad Web Test**
2. Click **Run workflow**
3. Enter scenario name: `scenario1`
4. Click **Run workflow**

---

## Option 2: Automated Upload via GitHub Actions

I've created a new workflow **"NeoLoad Web Test"** that attempts to:
1. Package your project files
2. Upload to NeoLoad Web via API
3. Create test settings
4. Run the test

**Note:** This may require additional API configuration depending on your NeoLoad Web plan.

---

## Comparison

| Method | Pros | Cons |
|--------|------|------|
| **Manual Upload** | ✅ Reliable<br>✅ One-time setup<br>✅ Verifiable in UI | ❌ Requires NeoLoad Desktop<br>❌ Manual step |
| **Automated Upload** | ✅ Fully automated<br>✅ No desktop needed | ❌ May need API config<br>❌ More complex troubleshooting |

---

## Recommended Workflow

### For Initial Setup:
1. **Manually upload** project once via NeoLoad Desktop
2. Verify in NeoLoad Web UI
3. Use GitHub Actions to run tests going forward

### For Updates:
- When you modify your NeoLoad project:
  1. Update in NeoLoad Desktop
  2. Re-upload to NeoLoad Web
  3. Commit changes to GitHub
  4. GitHub Actions can run the updated tests

---

## Quick Commands

### List Projects in NeoLoad Web:
```bash
neoload login --url https://neoload-api.saas.neotys.com YOUR_TOKEN
neoload workspaces use "Default Workspace"
neoload projects ls
```

### List Available Scenarios:
```bash
neoload test-settings ls
```

---

## Troubleshooting

### "No object associated to the name 'None'"
- **Cause:** Project not uploaded to NeoLoad Web or scenario doesn't exist
- **Fix:** Upload project manually via NeoLoad Desktop first

### "No id associated to the name"
- **Cause:** Workspace not properly selected
- **Fix:** Verify workspace name is exactly "Default Workspace"

### Test settings not found
- **Cause:** Test settings not created for the project
- **Fix:** Create test settings in NeoLoad Web UI or via CLI

---

## Next Steps

**I recommend Option 1 (Manual Upload) for now:**

1. Upload your project to NeoLoad Web via NeoLoad Desktop
2. Then use the GitHub Actions workflow to run tests automatically

Would you like help with any of these steps?
