# Security Policy

## Supported Versions

We release security updates for the following versions:

| Version | Supported          |
| ------- | ------------------ |
| 0.1.x   | :white_check_mark: |

## Reporting a Vulnerability

**Please do not report security vulnerabilities through public GitHub issues.**

If you discover a security vulnerability in this project, please report it by:

1. **Email**: Contact the repository owner directly
2. **GitHub Security Advisories**: Use the "Security" tab → "Report a vulnerability"

### What to Include

Please include the following information in your report:

- Type of vulnerability
- Full paths of source file(s) related to the vulnerability
- Location of the affected source code (tag/branch/commit or direct URL)
- Step-by-step instructions to reproduce the issue
- Proof-of-concept or exploit code (if possible)
- Impact of the vulnerability, including how an attacker might exploit it

### Response Timeline

- **Initial Response**: Within 48 hours
- **Status Update**: Within 7 days
- **Fix Timeline**: Depends on severity
  - Critical: 1-7 days
  - High: 7-14 days
  - Medium: 14-30 days
  - Low: 30-90 days

## Security Features

This application implements the following security measures:

### Data Protection
- ✅ **AES-128-GCM encryption** for database at rest
- ✅ **Argon2id** for passphrase hashing
- ✅ **No cloud storage** - all data stays local
- ✅ **Secure file permissions** (600 for database)

### Code Security
- ✅ **Parameterized SQL queries** (SQLAlchemy ORM)
- ✅ **Input validation** (Pydantic)
- ✅ **React auto-escaping** (XSS protection)
- ✅ **Content Security Policy** headers
- ✅ **CORS restrictions** (localhost only)

### Dependency Security
- ✅ **Dependabot** for automated updates
- ✅ **CodeQL** for vulnerability scanning
- ✅ **npm audit** and **Safety** checks
- ✅ **Secret scanning** with TruffleHog

## Security Best Practices for Users

### Before Using
1. Enable FileVault (macOS) or BitLocker (Windows) for full-disk encryption
2. Choose a strong passphrase (12+ characters, mixed case, numbers, symbols)
3. Review IBM's insider trading policy
4. Verify you're not subject to SEC Section 16 reporting

### During Use
1. Never share your passphrase
2. Lock your screen when stepping away
3. Only import files from trusted sources (Morgan Stanley exports)
4. Check blackout window status before viewing dashboard
5. Don't take screenshots and share publicly

### Ongoing Maintenance
1. Back up database file weekly
2. Update dependencies monthly (`npm audit`, `pip-audit`)
3. Review access logs (future feature)
4. Change passphrase if device is compromised

## Known Limitations

### Out of Scope Threats
The following threats are **explicitly out of scope** for this application:

1. **OS-level malware** - User must maintain secure operating system
2. **Physical coercion** - "Rubber hose cryptanalysis"
3. **Quantum computing attacks** - AES-128 is quantum-resistant for foreseeable future
4. **Social engineering** - User must not be tricked into revealing passphrase
5. **Hardware keyloggers** - Physical device security is user's responsibility

### Residual Risks
- **Medium**: Keylogger/screen capture malware (OS-level threat)
- **Low**: Stolen laptop with weak passphrase
- **Low**: Supply chain attack on dependencies
- **Very Low**: Network interception (localhost-only)

## Security Audits

### Automated Scans
- **CodeQL**: Runs on every push and weekly
- **Dependabot**: Checks dependencies weekly
- **npm audit**: Runs on every push
- **Bandit**: Python security linter on every push
- **Safety**: Python dependency vulnerability check
- **TruffleHog**: Secret scanning on every push

### Manual Reviews
- Security architecture reviewed in `docs/THREAT_MODEL.md`
- All calculations documented in `docs/FORMULAS.md`
- Threat scenarios analyzed with mitigations

## Compliance Considerations

### Legal Disclaimers
⚠️ **This application is for informational purposes only and does not constitute financial, legal, or tax advice.**

Before using:
1. Check IBM's insider trading policy and blackout windows
2. Verify if you're subject to SEC Section 16 reporting
3. Obtain pre-clearance for trades if required
4. Consult a tax professional for actual tax liability
5. Review Morgan Stanley at Work Terms of Service

### Data Privacy
- **No data leaves your device** - local-only architecture
- **No telemetry or analytics** - zero external requests except yfinance
- **No cloud sync** - all data stays on your machine
- **GDPR/CCPA**: N/A (no data collection)

## Security Contacts

- **Repository Owner**: Citi-Testing-Site
- **GitHub Security**: Use "Security" tab → "Report a vulnerability"

## Acknowledgments

We thank the security research community for responsible disclosure of vulnerabilities.

---

**Last Updated**: 2026-05-27  
**Version**: 1.0