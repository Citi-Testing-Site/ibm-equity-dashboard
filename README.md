# IBM Equity Dashboard

A private, locally-run web application for tracking IBM equity compensation (RSUs, stock options, and ESPP).

## ⚠️ IMPORTANT LEGAL DISCLAIMER

**This application is for informational purposes only and does not constitute financial, legal, or tax advice.**

Before using this application or making any trading decisions:

1. ✅ Review IBM's insider trading policy and blackout window status
2. ✅ Verify if you are subject to SEC Section 16 reporting requirements
3. ✅ Obtain pre-clearance for trades if required by your role
4. ✅ Consult a tax professional for actual tax liability
5. ✅ Review Morgan Stanley at Work Terms of Service

**This application is read-only and never places trades or modifies your accounts.**

---

## 🎯 Features

### MVP (Current Version: 0.1.0)
- ✅ Manual Excel/CSV import from Morgan Stanley exports
- ✅ Live IBM stock price from yfinance
- ✅ Dashboard with summary tiles (total value, RSU value, options value, ESPP value)
- ✅ Holdings tables for RSUs, stock options, and ESPP purchases
- ✅ "What-if" price calculator for scenario modeling
- ✅ AES-128 encryption with passphrase protection
- ✅ Localhost-only (no cloud, no telemetry)

### Coming in v1.0
- 📊 Value-over-time chart
- 📅 Vesting timeline with upcoming events
- 💰 Tax withholding estimates
- 📄 Export to CSV
- ⚙️ Settings page (refresh interval, tax rate)

### Coming in v2.0
- 🤖 Automated browser scraping (optional, with user consent)
- 📈 Advanced analytics (IRR, cost basis, holding period)
- 💾 Encrypted backup/restore
- 🖥️ Desktop app (Tauri)

---

## 🚀 Quick Start

### Prerequisites

- **macOS** (tested on macOS 13+)
- **Python 3.11+** ([Download](https://www.python.org/downloads/))
- **Node.js 18+** ([Download](https://nodejs.org/))
- **Git** (pre-installed on macOS)

### Installation

1. **Clone or download this repository**
   ```bash
   cd ~/Desktop
   # If you haven't already, the project should be in ibm-equity-dashboard/
   cd ibm-equity-dashboard
   ```

2. **Set up the backend**
   ```bash
   cd backend
   
   # Create virtual environment
   python3 -m venv venv
   source venv/bin/activate
   
   # Install dependencies
   pip install -r requirements.txt
   
   # Copy environment file
   cp .env.example .env
   
   # Edit .env if needed (defaults are fine for local use)
   ```

3. **Set up the frontend**
   ```bash
   cd ../frontend
   
   # Install dependencies
   npm install
   ```

4. **Verify yfinance works**
   ```bash
   cd ../backend
   source venv/bin/activate
   python -c "import yfinance as yf; print(f'IBM Price: ${yf.Ticker(\"IBM\").info[\"currentPrice\"]}')"
   ```
   
   You should see: `IBM Price: $247.85` (or current price)

---

## 🏃 Running the Application

### Start Backend (Terminal 1)
```bash
cd ~/Desktop/ibm-equity-dashboard/backend
source venv/bin/activate
uvicorn app.main:app --reload --port 8000
```

You should see:
```
INFO:     Uvicorn running on http://127.0.0.1:8000
INFO:     Application startup complete.
```

### Start Frontend (Terminal 2)
```bash
cd ~/Desktop/ibm-equity-dashboard/frontend
npm run dev
```

You should see:
```
  VITE v5.0.11  ready in 523 ms

  ➜  Local:   http://localhost:3000/
  ➜  Network: use --host to expose
```

### Open the Application
1. Open your browser to **http://localhost:3000**
2. On first launch, you'll be prompted to create a passphrase
3. Import your Morgan Stanley Excel export (or use the sample data)
4. View your dashboard!

---

## 📊 Importing Your Data

### Option 1: Use Sample Data (Recommended for Testing)
The project includes `IBM_equity_spoof_data.xlsx` on your Desktop. This is synthetic data for testing.

1. In the app, click **"Import Data"**
2. Select `IBM_equity_spoof_data.xlsx`
3. Review the import summary
4. Click **"Confirm Import"**

### Option 2: Export from Morgan Stanley at Work
1. Log in to [Morgan Stanley at Work](https://www.morganstanley.com/spc/)
2. Navigate to **Holdings** or **Activity**
3. Click **"Export"** or **"Download"**
4. Save the Excel file
5. Import it into the app

**Expected Excel Structure:**
- Sheet: "RSU Grants" with columns: Grant Number, Grant Date, Units Granted, Vested Units, Unvested Units
- Sheet: "Stock Options" with columns: Grant Number, Strike Price, Vested, Outstanding
- Sheet: "ESPP Purchases" with columns: Purchase Date, Shares Purchased, Purchase Price

---

## 🔐 Security & Privacy

### Encryption
- **Database**: AES-128-GCM encryption via SQLCipher
- **Key Derivation**: Argon2id from your passphrase (memory-hard, GPU-resistant)
- **No Key Storage**: Passphrase required on every app start

### What's Encrypted
- ✅ All equity holdings (RSUs, options, ESPP)
- ✅ Vesting schedules
- ✅ Import history

### What's NOT Encrypted
- ❌ IBM stock price (public data)
- ❌ Application logs (contain no PII)

### Best Practices
1. **Choose a strong passphrase** (12+ characters, mixed case, numbers, symbols)
2. **Enable FileVault** (macOS full-disk encryption)
3. **Back up your database** regularly (see Backup section)
4. **Don't share your passphrase** with anyone
5. **Lock your screen** when stepping away

---

## 💾 Backup & Restore

### Manual Backup
```bash
# Copy the encrypted database
cp ~/.ibm-equity-dashboard/equity.db ~/Desktop/equity-backup-$(date +%Y%m%d).db
```

### Restore from Backup
```bash
# Stop the backend first (Ctrl+C in Terminal 1)
cp ~/Desktop/equity-backup-20260526.db ~/.ibm-equity-dashboard/equity.db
# Restart the backend
```

### Automated Backup (Optional)
Add to your crontab:
```bash
# Backup daily at 2 AM
0 2 * * * cp ~/.ibm-equity-dashboard/equity.db ~/Backups/equity-$(date +\%Y\%m\%d).db
```

---

## 📐 Calculation Formulas

All formulas are documented in [`docs/FORMULAS.md`](docs/FORMULAS.md).

### Key Formulas

**RSU Value:**
```
RSU Value = (Vested Units + Unvested Units) × Current Market Price
```

**Options Intrinsic Value:**
```
Intrinsic Value = MAX(0, Current Price - Strike Price) × Vested Options
```

**Tax Withholding Estimate:**
```
Tax Withheld = Gross Value × Tax Rate (default 37%)
Net Value = Gross Value - Tax Withheld
```

**⚠️ Tax estimates are for informational purposes only. Consult a CPA for actual tax liability.**

---

## 🛠️ Troubleshooting

### Backend won't start
```bash
# Check Python version
python3 --version  # Should be 3.11+

# Reinstall dependencies
cd backend
source venv/bin/activate
pip install --upgrade -r requirements.txt
```

### Frontend won't start
```bash
# Clear node_modules and reinstall
cd frontend
rm -rf node_modules package-lock.json
npm install
```

### "Cannot connect to backend" error
1. Verify backend is running on port 8000
2. Check for port conflicts: `lsof -i :8000`
3. Try restarting both backend and frontend

### yfinance not fetching prices
```bash
# Test yfinance directly
python -c "import yfinance as yf; print(yf.Ticker('IBM').info)"

# If it fails, check your internet connection
# yfinance scrapes Yahoo Finance, so it requires internet access
```

### Forgot passphrase
**There is no password recovery.** If you forget your passphrase, you must:
1. Delete the database: `rm ~/.ibm-equity-dashboard/equity.db`
2. Restart the app and create a new passphrase
3. Re-import your data

This is by design for security.

---

## 📚 Documentation

- [Architecture & Tech Stack](docs/ARCHITECTURE.md)
- [Threat Model & Security](docs/THREAT_MODEL.md)
- [Implementation Plan](docs/IMPLEMENTATION_PLAN.md)
- [Calculation Formulas](docs/FORMULAS.md)

---

## 🤝 Contributing

This is a personal-use application, but contributions are welcome!

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m "feat: add my feature"`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a Pull Request

---

## 📄 License

MIT License - See [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgments

- **yfinance** for free market data
- **FastAPI** for the excellent Python web framework
- **React** and **Vite** for the frontend
- **SQLCipher** for database encryption
- **Tailwind CSS** for styling

---

## 📞 Support

For issues or questions:
1. Check the [Troubleshooting](#-troubleshooting) section
2. Review the [documentation](docs/)
3. Open an issue on GitHub

---

## ⚖️ Legal

**This application is not affiliated with, endorsed by, or sponsored by IBM Corporation or Morgan Stanley.**

IBM and the IBM logo are trademarks of International Business Machines Corporation.

Morgan Stanley is a trademark of Morgan Stanley.

All trademarks are the property of their respective owners.

---

**Version**: 0.1.0-mvp  
**Last Updated**: 2026-05-26  
**Status**: Milestone 0 - Project Setup Complete