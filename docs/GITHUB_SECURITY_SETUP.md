# GitHub Advanced Security Setup Guide

This guide explains how to enable and configure GitHub's built-in security features for the IBM Equity Dashboard repository.

## 🎯 Overview

We've configured the following automated security features:

1. **Dependabot** - Automated dependency updates
2. **CodeQL** - Code vulnerability scanning
3. **Secret Scanning** - Detects exposed credentials
4. **Security Advisories** - Vulnerability reporting
5. **Additional Scans** - Bandit, Safety, npm audit, TruffleHog

## ✅ What's Already Configured

The following files have been added to the repository:

- `.github/dependabot.yml` - Dependency update configuration
- `.github/workflows/codeql.yml` - CodeQL security analysis
- `.github/workflows/security-scan.yml` - Additional security checks
- `.github/SECURITY.md` - Security policy and reporting guidelines

## 🚀 Enabling GitHub Security Features

### Step 1: Enable Dependabot

Dependabot is **automatically enabled** for public repositories. To verify:

1. Go to https://github.com/Citi-Testing-Site/ibm-equity-dashboard
2. Click **"Settings"** tab
3. Click **"Security"** in the left sidebar
4. Under **"Code security and analysis"**:
   - ✅ **Dependabot alerts** should be enabled
   - ✅ **Dependabot security updates** should be enabled
   - ✅ **Dependabot version updates** should be enabled (uses our `.github/dependabot.yml`)

**What it does:**
- Scans `requirements.txt` and `package.json` for vulnerable dependencies
- Creates pull requests to update dependencies weekly
- Groups related updates to reduce PR noise

### Step 2: Enable CodeQL Analysis

CodeQL is **automatically enabled** for public repositories. To verify:

1. Go to **Settings** → **Security** → **Code security and analysis**
2. Under **"Code scanning"**:
   - ✅ **CodeQL analysis** should show as "Set up" or "Enabled"

**What it does:**
- Scans Python and JavaScript code for security vulnerabilities
- Runs on every push to `main` and on pull requests
- Runs weekly on Monday at 6 AM UTC
- Detects: SQL injection, XSS, path traversal, insecure crypto, etc.

### Step 3: Enable Secret Scanning

For **public repositories**, secret scanning is **automatically enabled**.

For **private repositories** (if you change visibility):

1. Go to **Settings** → **Security** → **Code security and analysis**
2. Enable **"Secret scanning"**
3. Enable **"Push protection"** (prevents accidental commits of secrets)

**What it scans for:**
- API keys (AWS, Azure, Google Cloud, etc.)
- Database credentials
- Private keys and certificates
- OAuth tokens
- Passphrases in code

### Step 4: Review Security Advisories

1. Go to **Security** tab → **Advisories**
2. Click **"New draft security advisory"** to report vulnerabilities
3. Review any existing advisories

### Step 5: Enable GitHub Actions (if not already)

1. Go to **Settings** → **Actions** → **General**
2. Under **"Actions permissions"**, select:
   - ✅ **"Allow all actions and reusable workflows"**
3. Under **"Workflow permissions"**, select:
   - ✅ **"Read and write permissions"**
   - ✅ **"Allow GitHub Actions to create and approve pull requests"**

This allows our security workflows to run automatically.

## 📊 Monitoring Security Alerts

### Dependabot Alerts

**View alerts:**
1. Go to **Security** tab → **Dependabot alerts**
2. Review any vulnerabilities found in dependencies
3. Click **"Review security update"** to see the proposed fix

**Auto-merge safe updates:**
```bash
# Enable auto-merge for Dependabot PRs (optional)
gh repo set-default Citi-Testing-Site/ibm-equity-dashboard
gh pr list --author app/dependabot
gh pr merge <PR-NUMBER> --auto --squash
```

### CodeQL Alerts

**View alerts:**
1. Go to **Security** tab → **Code scanning**
2. Review any vulnerabilities found in code
3. Click on an alert to see:
   - Vulnerability description
   - Affected code location
   - Remediation guidance
   - Severity level

**Dismiss false positives:**
1. Click on the alert
2. Click **"Dismiss alert"**
3. Select reason (e.g., "False positive", "Won't fix", "Used in tests")

### Secret Scanning Alerts

**View alerts:**
1. Go to **Security** tab → **Secret scanning**
2. Review any exposed secrets
3. **Immediately revoke** any real credentials found
4. Update the code to remove the secret
5. Mark as resolved once fixed

## 🔔 Notification Settings

Configure how you receive security alerts:

1. Go to **Settings** (your profile) → **Notifications**
2. Under **"Security alerts"**:
   - ✅ Enable **"Dependabot alerts"**
   - ✅ Enable **"Secret scanning alerts"**
   - ✅ Enable **"Code scanning alerts"**
3. Choose notification method:
   - Email
   - Web notifications
   - GitHub mobile app

## 📈 Security Workflow Behavior

### On Every Push to `main`:
- ✅ CodeQL scans Python and JavaScript code
- ✅ Bandit scans Python for security issues
- ✅ npm audit checks JavaScript dependencies
- ✅ TruffleHog scans for secrets in commits

### On Every Pull Request:
- ✅ All above scans run
- ✅ Dependency Review checks for new vulnerabilities
- ✅ Results posted as PR comments

### Weekly (Monday 6 AM UTC):
- ✅ CodeQL full scan
- ✅ Dependabot checks for dependency updates

### Daily (2 AM UTC):
- ✅ Full security scan suite runs

## 🛠️ Customizing Security Scans

### Add Custom CodeQL Queries

Edit `.github/workflows/codeql.yml`:

```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: ${{ matrix.language }}
    queries: security-extended,security-and-quality
    # Add custom queries:
    config-file: ./.github/codeql/codeql-config.yml
```

### Adjust Dependabot Schedule

Edit `.github/dependabot.yml`:

```yaml
schedule:
  interval: "daily"  # Change from "weekly" to "daily"
  time: "09:00"      # Change time
```

### Add More Security Tools

Edit `.github/workflows/security-scan.yml` to add:
- **Snyk**: `snyk test`
- **Trivy**: Container scanning
- **OWASP Dependency-Check**: Java/Maven projects
- **GitLeaks**: Alternative secret scanner

## 📋 Security Checklist

After enabling all features, verify:

- [ ] Dependabot alerts enabled
- [ ] Dependabot security updates enabled
- [ ] Dependabot version updates enabled
- [ ] CodeQL analysis enabled
- [ ] Secret scanning enabled
- [ ] Push protection enabled (for private repos)
- [ ] GitHub Actions workflows running successfully
- [ ] Notifications configured
- [ ] Security policy (SECURITY.md) visible in Security tab
- [ ] No existing security alerts (or all reviewed)

## 🔍 Viewing Security Status

### Repository Security Overview

1. Go to **Security** tab
2. View the **"Security overview"** dashboard:
   - Open alerts by severity
   - Recent activity
   - Dependency graph
   - Security advisories

### Security Insights

1. Go to **Insights** tab → **"Security"**
2. View:
   - Alert trends over time
   - Time to remediation
   - Alert distribution by severity

## 🚨 Responding to Security Alerts

### Critical Severity
1. **Immediate action required**
2. Review the vulnerability details
3. Apply the fix or update dependency
4. Test thoroughly
5. Deploy fix within 24-48 hours

### High Severity
1. **Action required within 7 days**
2. Review and prioritize
3. Create a fix or update
4. Test and deploy

### Medium/Low Severity
1. **Address in next sprint**
2. Add to backlog
3. Fix during regular maintenance

## 📚 Additional Resources

- [GitHub Security Documentation](https://docs.github.com/en/code-security)
- [Dependabot Documentation](https://docs.github.com/en/code-security/dependabot)
- [CodeQL Documentation](https://codeql.github.com/docs/)
- [Secret Scanning Documentation](https://docs.github.com/en/code-security/secret-scanning)

## 🎓 Security Best Practices

1. **Review alerts weekly** - Don't let them pile up
2. **Keep dependencies updated** - Merge Dependabot PRs promptly
3. **Never commit secrets** - Use environment variables
4. **Test security updates** - Don't blindly merge
5. **Document exceptions** - If dismissing an alert, explain why
6. **Monitor trends** - Watch for increasing alert counts
7. **Educate team** - Ensure everyone understands security practices

---

**Last Updated**: 2026-05-27  
**Version**: 1.0