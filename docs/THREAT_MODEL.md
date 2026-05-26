# IBM Equity Dashboard — Threat Model & Security Analysis

## 🎯 Security Objectives

1. **Confidentiality**: Equity holdings data must remain private and encrypted at rest
2. **Integrity**: Data cannot be tampered with without detection
3. **Availability**: User can access their data when needed (local-only, no cloud dependency)
4. **Auditability**: User can verify what data exists and when it was last modified
5. **Compliance**: Application must not violate Morgan Stanley ToS or IBM insider trading policies

---

## 🔐 Assets to Protect

### **Critical Assets (High Value)**
- RSU grant details (grant dates, quantities, vesting schedules)
- Stock option details (strike prices, expiration dates, vested quantities)
- ESPP purchase history (purchase prices, quantities, dates)
- Vesting schedule (future vest dates and quantities)
- User's passphrase (never stored, only used for key derivation)

### **Sensitive Assets (Medium Value)**
- Historical import files (may contain PII like employee ID)
- Calculated net worth and unrealized gains
- Tax withholding estimates
- User preferences (refresh intervals, display settings)

### **Public Assets (Low Value)**
- IBM stock price (publicly available)
- Application source code (open source)
- UI screenshots (no sensitive data visible)

---

## 🚨 Threat Actors & Motivations

### **1. Opportunistic Attacker (Physical Access)**
- **Profile**: Thief who steals laptop, roommate, family member
- **Motivation**: Identity theft, financial gain, curiosity
- **Capabilities**: Can access file system, cannot break strong encryption
- **Likelihood**: Medium (laptop theft is common)
- **Impact**: High (full equity portfolio exposed)

### **2. Targeted Attacker (Malware)**
- **Profile**: Sophisticated malware, keylogger, RAT (Remote Access Trojan)
- **Motivation**: Financial fraud, corporate espionage
- **Capabilities**: Can capture keystrokes, screenshots, memory dumps
- **Likelihood**: Low (requires user to install malware)
- **Impact**: Critical (can capture passphrase, all data)

### **3. Network Eavesdropper**
- **Profile**: ISP, coffee shop WiFi attacker, nation-state
- **Motivation**: Surveillance, data collection
- **Capabilities**: Can intercept network traffic
- **Likelihood**: Low (localhost-only app, minimal external traffic)
- **Impact**: Low (only IBM price data is transmitted, which is public)

### **4. Insider Threat (Morgan Stanley Employee)**
- **Profile**: Rogue employee at Morgan Stanley or IBM
- **Motivation**: Data theft, competitive intelligence
- **Capabilities**: Access to upstream systems, not to user's local app
- **Likelihood**: Very Low (strong internal controls)
- **Impact**: Medium (could affect data source integrity, not local app)

### **5. Legal/Compliance Risk**
- **Profile**: SEC, IBM Legal, Morgan Stanley Compliance
- **Motivation**: Enforce insider trading rules, ToS violations
- **Capabilities**: Subpoena, account suspension
- **Likelihood**: Low (if user follows blackout windows)
- **Impact**: High (account suspension, legal penalties)

---

## 🛡️ Threat Scenarios & Mitigations

### **Scenario 1: Stolen Laptop**

**Attack Path:**
1. Attacker steals laptop or gains physical access
2. Boots into recovery mode or removes hard drive
3. Attempts to read SQLite database file

**Mitigations:**
- ✅ **SQLCipher encryption** (AES-128-GCM) protects database at rest
- ✅ **Argon2id key derivation** makes brute-force attacks computationally expensive
- ✅ **No passphrase storage** - attacker cannot decrypt without passphrase
- ✅ **File permissions** (600) prevent other users from reading database
- ⚠️ **Limitation**: If attacker has passphrase (written down, weak password), data is exposed

**Residual Risk:** LOW (assuming strong passphrase)

**User Actions Required:**
- Choose a strong passphrase (12+ characters, mixed case, numbers, symbols)
- Enable FileVault (macOS) or BitLocker (Windows) for full-disk encryption
- Do not write passphrase down or store in password manager on same device

---

### **Scenario 2: Keylogger / Screen Capture Malware**

**Attack Path:**
1. User unknowingly installs malware (phishing, fake software update)
2. Malware captures keystrokes when user enters passphrase
3. Malware screenshots dashboard showing equity values
4. Attacker exfiltrates data over network

**Mitigations:**
- ⚠️ **Out of scope** - OS-level threat, application cannot defend against this
- ✅ **Auto-lock after idle** (future feature) limits exposure window
- ✅ **No cloud sync** means data stays local, harder to exfiltrate
- ⚠️ **Limitation**: If OS is compromised, all bets are off

**Residual Risk:** MEDIUM (depends on user's security hygiene)

**User Actions Required:**
- Keep macOS updated with latest security patches
- Use reputable antivirus software
- Enable macOS Gatekeeper (only run signed apps)
- Be cautious of phishing emails and suspicious downloads
- Consider using a hardware security key for passphrase (future feature)

---

### **Scenario 3: Network Interception (MITM)**

**Attack Path:**
1. User connects to untrusted WiFi (coffee shop, airport)
2. Attacker performs MITM attack on network traffic
3. Attempts to intercept yfinance API calls or localhost traffic

**Mitigations:**
- ✅ **Localhost-only** - frontend and backend communicate over 127.0.0.1 (never leaves machine)
- ✅ **HTTPS for yfinance** - IBM price data fetched over TLS
- ✅ **No authentication tokens** sent over network
- ✅ **No telemetry or analytics** - zero external requests except yfinance
- ⚠️ **Limitation**: yfinance could be compromised (supply chain attack)

**Residual Risk:** VERY LOW

**User Actions Required:**
- None - application design prevents this attack

---

### **Scenario 4: SQL Injection**

**Attack Path:**
1. Attacker crafts malicious input in Excel import file
2. Attempts to inject SQL commands via grant numbers, dates, or amounts
3. Tries to extract data or modify database

**Mitigations:**
- ✅ **Parameterized queries** - SQLAlchemy ORM prevents SQL injection
- ✅ **Pydantic validation** - all inputs validated before database insertion
- ✅ **Type checking** - TypeScript and Python typing catch malformed data
- ✅ **Schema validation** - Excel parser rejects files with unexpected structure

**Residual Risk:** VERY LOW

**User Actions Required:**
- Only import files from trusted sources (Morgan Stanley exports)

---

### **Scenario 5: Cross-Site Scripting (XSS)**

**Attack Path:**
1. Attacker injects malicious JavaScript into grant number or description field
2. Script executes when user views dashboard
3. Attempts to steal session data or exfiltrate equity values

**Mitigations:**
- ✅ **React auto-escaping** - all user input is escaped by default
- ✅ **Content Security Policy** headers prevent inline scripts
- ✅ **No `dangerouslySetInnerHTML`** used in codebase
- ✅ **TypeScript** prevents accidental XSS via type safety

**Residual Risk:** VERY LOW

**User Actions Required:**
- None - framework handles this automatically

---

### **Scenario 6: Unauthorized File Access (Multi-User System)**

**Attack Path:**
1. Another user on the same macOS system tries to access database
2. Attempts to read SQLite file or import files
3. Tries to view application logs

**Mitigations:**
- ✅ **File permissions** (600) - only owner can read/write database
- ✅ **Encryption** - even if file is copied, cannot be decrypted without passphrase
- ✅ **No plaintext logs** - logs contain no PII or equity data
- ✅ **Secure temp file handling** - import files deleted after processing

**Residual Risk:** LOW

**User Actions Required:**
- Do not share macOS user account with others
- Use separate user accounts for family members

---

### **Scenario 7: Supply Chain Attack (Compromised Dependency)**

**Attack Path:**
1. Attacker compromises a npm or PyPI package used by the app
2. Malicious code exfiltrates data or steals passphrase
3. User installs compromised version via `npm install` or `pip install`

**Mitigations:**
- ✅ **Dependency pinning** - exact versions in package-lock.json and requirements.txt
- ✅ **Open-source only** - all dependencies are auditable
- ✅ **No telemetry libraries** - no analytics, no crash reporting
- ⚠️ **Limitation**: Cannot prevent zero-day in legitimate package

**Residual Risk:** LOW (but non-zero)

**User Actions Required:**
- Review dependency updates before installing
- Use `npm audit` and `pip-audit` to check for known vulnerabilities
- Consider using Dependabot or Renovate for automated security updates

---

### **Scenario 8: Insider Trading Violation**

**Attack Path:**
1. User views equity dashboard during IBM blackout window
2. User makes trading decision based on material non-public information (MNPI)
3. SEC or IBM Legal investigates

**Mitigations:**
- ✅ **Disclaimer in UI** - warns user to check blackout windows and pre-clearance
- ✅ **Read-only app** - cannot place trades, only view data
- ✅ **No cloud sync** - no evidence of access outside user's device
- ⚠️ **Limitation**: App cannot enforce blackout windows (user responsibility)

**Residual Risk:** MEDIUM (depends on user compliance)

**User Actions Required:**
- **CRITICAL**: Check IBM's insider trading policy before using app
- Verify you are not subject to Section 16 reporting (officers, directors, 10%+ owners)
- Respect blackout windows (typically around earnings announcements)
- Obtain pre-clearance for large trades if required by your role
- Consult IBM Legal or a securities attorney if unsure

---

### **Scenario 9: Morgan Stanley ToS Violation**

**Attack Path:**
1. User automates data extraction via browser scraping (future feature)
2. Morgan Stanley detects automated access and flags account
3. Account is suspended or restricted

**Mitigations:**
- ✅ **Manual import for MVP** - no automated scraping in initial version
- ✅ **Read-only access** - never modifies upstream accounts
- ⚠️ **Future risk**: If Playwright automation is added, must respect rate limits and ToS

**Residual Risk:** LOW (for MVP), MEDIUM (if automation added)

**User Actions Required:**
- Review Morgan Stanley at Work Terms of Service
- If using automation (future), add delays and respect rate limits
- Do not share Morgan Stanley credentials with third parties

---

### **Scenario 10: Data Loss (Accidental Deletion)**

**Attack Path:**
1. User accidentally deletes database file
2. Hard drive fails or laptop is lost
3. No backup exists

**Mitigations:**
- ✅ **Backup instructions** in README
- ✅ **Export functionality** (future) - export to encrypted CSV
- ⚠️ **Limitation**: User must manually back up database file

**Residual Risk:** MEDIUM (depends on user backup discipline)

**User Actions Required:**
- Regularly back up `~/.ibm-equity-dashboard/equity.db` to external drive
- Use Time Machine (macOS) or similar backup solution
- Test restore process periodically

---

## 🔍 Security Controls Summary

| Control | Type | Effectiveness | Implementation |
|---------|------|---------------|----------------|
| AES-128-GCM encryption | Preventive | High | SQLCipher |
| Argon2id key derivation | Preventive | High | argon2-cffi |
| Parameterized queries | Preventive | High | SQLAlchemy ORM |
| Input validation | Preventive | High | Pydantic |
| File permissions (600) | Preventive | Medium | OS-level |
| React auto-escaping | Preventive | High | Framework default |
| Content Security Policy | Preventive | Medium | HTTP headers |
| Localhost-only | Preventive | High | Architecture |
| No telemetry | Preventive | High | Design choice |
| Compliance disclaimer | Detective | Low | UI warning |
| Audit logs (future) | Detective | Medium | Not yet implemented |
| Auto-lock (future) | Preventive | Medium | Not yet implemented |

---

## 📋 Security Checklist for Users

### **Before First Use:**
- [ ] Enable full-disk encryption (FileVault on macOS)
- [ ] Choose a strong passphrase (12+ characters, unique)
- [ ] Review IBM insider trading policy
- [ ] Verify you are not subject to Section 16 reporting
- [ ] Review Morgan Stanley at Work Terms of Service

### **During Use:**
- [ ] Only import files from official Morgan Stanley exports
- [ ] Check blackout window status before viewing dashboard
- [ ] Do not share passphrase with anyone
- [ ] Lock screen when stepping away from computer
- [ ] Do not take screenshots of dashboard and share publicly

### **Ongoing Maintenance:**
- [ ] Back up database file weekly to external drive
- [ ] Update application dependencies monthly (`npm audit`, `pip-audit`)
- [ ] Review access logs (future feature) for suspicious activity
- [ ] Change passphrase if device is compromised

---

## 🚫 Out of Scope Threats

The following threats are **explicitly out of scope** for this application:

1. **OS-level malware** - User must maintain secure operating system
2. **Physical coercion** - "Rubber hose cryptanalysis" (forcing user to reveal passphrase)
3. **Quantum computing attacks** - AES-128 is quantum-resistant for foreseeable future
4. **Social engineering** - User must not be tricked into revealing passphrase
5. **Hardware keyloggers** - Physical device security is user's responsibility
6. **Nation-state attacks** - Application is not designed for high-security environments

---

## 🔮 Future Security Enhancements

### **Phase 2 (Post-MVP):**
- [ ] Auto-lock after 15 minutes of inactivity
- [ ] Audit log of all data access and modifications
- [ ] Export to encrypted CSV with separate passphrase
- [ ] Two-factor authentication (TOTP) for passphrase unlock
- [ ] Secure passphrase reset via recovery key

### **Phase 3 (Advanced):**
- [ ] Hardware security key support (YubiKey, Touch ID)
- [ ] Encrypted cloud backup (user-controlled, zero-knowledge)
- [ ] Certificate pinning for yfinance API
- [ ] Anomaly detection (unusual access patterns)
- [ ] Integration with IBM's official blackout window API (if available)

---

## ⚖️ Legal & Compliance Considerations

### **Insider Trading (SEC Rule 10b-5)**
- **Risk**: Using MNPI to trade IBM stock
- **Mitigation**: App is read-only, displays only user's own holdings
- **User Responsibility**: Check blackout windows, obtain pre-clearance

### **Section 16 Reporting (SEC)**
- **Risk**: Officers/directors must report trades within 2 business days
- **Mitigation**: App does not place trades, only displays data
- **User Responsibility**: If subject to Section 16, consult legal counsel

### **Morgan Stanley ToS**
- **Risk**: Automated scraping may violate terms of service
- **Mitigation**: MVP uses manual import only
- **User Responsibility**: Review ToS before using automation features

### **Tax Reporting (IRS)**
- **Risk**: Incorrect tax withholding estimates
- **Mitigation**: App displays estimates only, not tax advice
- **User Responsibility**: Consult tax professional for actual tax liability

### **Data Privacy (GDPR, CCPA)**
- **Risk**: N/A - no data leaves user's device
- **Mitigation**: Local-only architecture, no cloud storage
- **User Responsibility**: None

---

## 📞 Incident Response Plan

### **If Laptop is Stolen:**
1. Report theft to local police
2. Remotely wipe device if Find My Mac is enabled
3. Change Morgan Stanley password immediately
4. Monitor brokerage account for unauthorized trades
5. Consider changing passphrase on new device

### **If Passphrase is Compromised:**
1. Stop using the application immediately
2. Delete database file: `rm ~/.ibm-equity-dashboard/equity.db`
3. Re-import data with new passphrase
4. Review brokerage account for unauthorized access

### **If Malware is Detected:**
1. Disconnect from internet
2. Run full antivirus scan
3. Change all passwords (Morgan Stanley, brokerage, email)
4. Reinstall operating system if necessary
5. Restore data from clean backup

---

## ✅ Security Review Checklist

Please review this threat model and confirm:
1. Are there any threat scenarios I missed?
2. Do the mitigations seem reasonable for a personal-use application?
3. Are you comfortable with the residual risks?
4. Should I proceed with the **Milestone-Based Implementation Plan** next?

Once approved, I'll create the detailed implementation roadmap with MVP → v1 → v2 milestones.