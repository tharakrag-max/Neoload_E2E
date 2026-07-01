# NeoLoad GitHub Actions - Quick Setup Checklist

## Quick Setup (5 Minutes)

### 1. Create Workflow File
```bash
mkdir -p .github/workflows
```

Copy the workflow template from `SETUP_GUIDE.md` to `.github/workflows/neoload-test.yml`

**⚠️ IMPORTANT:** Update `YOUR_PROJECT_NAME.nlp` to your actual `.nlp` filename

---

### 2. Initialize Git & Push
```bash
# Initialize repository
git init
git add .
git commit -m "Add NeoLoad GitHub Actions workflow"

# Create GitHub repo and get URL, then:
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git branch -M main
git push -u origin main
```

---

### 3. Get NeoLoad Token
1. Login: https://neoload.saas.neotys.com/
2. Profile → API Token → Generate
3. **Copy token** (save it somewhere safe!)

---

### 4. Add Secret to GitHub
1. Go to: `https://github.com/YOUR_USERNAME/YOUR_REPO/settings/secrets/actions`
2. Click **New repository secret**
3. Name: `NEOLOAD_TOKEN`
4. Paste your token
5. Click **Add secret**

---

### 5. Run Test
1. Go to: `https://github.com/YOUR_USERNAME/YOUR_REPO/actions`
2. Click **NeoLoad Performance Test**
3. Click **Run workflow**
4. Click **Run workflow** (green button)

---

## That's It! 🎉

Your NeoLoad tests are now running on GitHub!

---

## Common Issues & Fixes

| Issue | Fix |
|-------|-----|
| Workflow not showing | Check file is in `.github/workflows/` and pushed to main |
| Auth error | Verify `NEOLOAD_TOKEN` secret is created |
| Project upload fails | Update `.nlp` filename in workflow YAML |
| Test fails | Check scenario name exists in your project |

---

## URLs You'll Need

- **Your Actions**: `https://github.com/YOUR_USERNAME/YOUR_REPO/actions`
- **Add Secrets**: `https://github.com/YOUR_USERNAME/YOUR_REPO/settings/secrets/actions`
- **NeoLoad Web**: https://neoload.saas.neotys.com/
- **Full Guide**: See `SETUP_GUIDE.md`
