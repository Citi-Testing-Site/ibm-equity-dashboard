# IBM Equity Dashboard — Milestone-Based Implementation Plan

## 🎯 Overview

This plan breaks down the project into three shippable milestones, each delivering incremental value. Each milestone can be used independently, allowing you to stop at any point if your needs are met.

**Timeline Estimates:**
- **MVP**: 2-3 days (core functionality)
- **v1.0**: +2-3 days (polish and features)
- **v2.0**: +3-4 days (advanced features)

**Total**: ~7-10 days of focused development

---

## 📦 Milestone 0: Project Setup (Day 0)

**Goal**: Create project structure and verify all tools work

### Tasks:
1. Create project directory structure
2. Initialize Git repository
3. Set up Python virtual environment
4. Install backend dependencies (FastAPI, SQLCipher, etc.)
5. Initialize Node.js project
6. Install frontend dependencies (React, Vite, Tailwind)
7. Verify yfinance can fetch IBM price
8. Create `.gitignore` for sensitive files
9. Write initial README with setup instructions

### Deliverables:
- [ ] Project scaffolding complete
- [ ] Both frontend and backend start without errors
- [ ] Can fetch IBM price from yfinance
- [ ] Git repository initialized with first commit

### Success Criteria:
```bash
# Backend starts successfully
cd backend && uvicorn app.main:app --reload
# Output: "Application startup complete"

# Frontend starts successfully
cd frontend && npm run dev
# Output: "Local: http://localhost:3000"

# yfinance test works
python -c "import yfinance as yf; print(yf.Ticker('IBM').info['currentPrice'])"
# Output: 247.85 (or current price)
```

---

## 🚀 Milestone 1: MVP (Minimum Viable Product)

**Goal**: Core functionality - import data, view dashboard, see current value

**Target**: End of Day 3

### Features:
1. ✅ Manual Excel import (using your spoof data)
2. ✅ Live IBM price from yfinance
3. ✅ Basic dashboard with summary tiles
4. ✅ Holdings tables (RSUs, Options, ESPP)
5. ✅ Simple "what-if" price calculator
6. ✅ Basic encryption (passphrase on startup)

### Backend Tasks:

#### 1. Database Schema (4 hours)
- [ ] Design SQLite schema for RSUs, Options, ESPP, Vesting
- [ ] Implement SQLCipher encryption wrapper
- [ ] Create database initialization script
- [ ] Write CRUD operations for each table

**Schema:**
```sql
-- RSU Grants
CREATE TABLE rsu_grants (
    id INTEGER PRIMARY KEY,
    grant_number TEXT UNIQUE NOT NULL,
    grant_date DATE NOT NULL,
    units_granted INTEGER NOT NULL,
    grant_price REAL NOT NULL,
    vesting_schedule TEXT NOT NULL,
    vested_units INTEGER DEFAULT 0,
    unvested_units INTEGER NOT NULL,
    cancelled_units INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Stock Options
CREATE TABLE stock_options (
    id INTEGER PRIMARY KEY,
    grant_number TEXT UNIQUE NOT NULL,
    grant_date DATE NOT NULL,
    expiration_date DATE NOT NULL,
    options_granted INTEGER NOT NULL,
    strike_price REAL NOT NULL,
    vesting_schedule TEXT NOT NULL,
    vested INTEGER DEFAULT 0,
    unvested INTEGER NOT NULL,
    exercised INTEGER DEFAULT 0,
    outstanding INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- ESPP Purchases
CREATE TABLE espp_purchases (
    id INTEGER PRIMARY KEY,
    purchase_date DATE NOT NULL,
    offering_period TEXT NOT NULL,
    contribution REAL NOT NULL,
    purchase_price REAL NOT NULL,
    fmv_at_purchase REAL NOT NULL,
    shares_purchased INTEGER NOT NULL,
    shares_held INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Vesting Schedule
CREATE TABLE vesting_schedule (
    id INTEGER PRIMARY KEY,
    grant_number TEXT NOT NULL,
    tranche_number TEXT NOT NULL,
    vest_date DATE NOT NULL,
    units_or_options INTEGER NOT NULL,
    status TEXT NOT NULL, -- VESTED, SCHEDULED, CANCELLED
    estimated_fmv REAL,
    estimated_tax_withheld REAL,
    net_value REAL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (grant_number) REFERENCES rsu_grants(grant_number) 
        OR REFERENCES stock_options(grant_number)
);

-- Market Data Cache
CREATE TABLE market_data (
    id INTEGER PRIMARY KEY,
    ticker TEXT NOT NULL,
    price REAL NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- User Settings
CREATE TABLE settings (
    key TEXT PRIMARY KEY,
    value TEXT NOT NULL,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 2. Excel Parser (3 hours)
- [ ] Parse RSU Grants sheet
- [ ] Parse Stock Options sheet
- [ ] Parse ESPP Purchases sheet
- [ ] Parse Vesting Schedule sheet
- [ ] Validate data types and ranges
- [ ] Handle missing or malformed data gracefully

**Parser Logic:**
```python
# Pseudo-code for Excel parser
def parse_excel(file_path: str) -> ImportResult:
    workbook = openpyxl.load_workbook(file_path)
    
    # Parse RSUs
    rsu_sheet = workbook["RSU Grants"]
    rsus = []
    for row in rsu_sheet.iter_rows(min_row=5, values_only=True):
        if row[0] and row[0] != "TOTAL":
            rsus.append(RSUGrant(
                grant_number=row[0],
                grant_date=row[2],
                units_granted=row[4],
                # ... etc
            ))
    
    # Similar for options and ESPP
    # Validate all data
    # Return summary
```

#### 3. Market Data Service (2 hours)
- [ ] Implement yfinance wrapper
- [ ] Add caching (5-minute TTL)
- [ ] Handle API failures gracefully
- [ ] Background task for auto-refresh

**Market Service:**
```python
class MarketDataService:
    def __init__(self):
        self.cache = {}
        self.cache_ttl = 300  # 5 minutes
    
    async def get_current_price(self, ticker: str) -> float:
        if self._is_cache_valid(ticker):
            return self.cache[ticker]["price"]
        
        try:
            stock = yf.Ticker(ticker)
            price = stock.info["currentPrice"]
            self._update_cache(ticker, price)
            return price
        except Exception as e:
            # Return cached price if available, else raise
            if ticker in self.cache:
                return self.cache[ticker]["price"]
            raise MarketDataError(f"Failed to fetch {ticker}: {e}")
```

#### 4. API Endpoints (3 hours)
- [ ] `POST /api/auth/unlock` - Verify passphrase, decrypt database
- [ ] `POST /api/import/excel` - Upload and parse Excel file
- [ ] `GET /api/holdings/summary` - Total value, gains, breakdown
- [ ] `GET /api/holdings/rsus` - All RSU grants
- [ ] `GET /api/holdings/options` - All stock options
- [ ] `GET /api/holdings/espp` - All ESPP purchases
- [ ] `GET /api/market/current` - Current IBM price
- [ ] `POST /api/analytics/whatif` - Calculate values at hypothetical price

### Frontend Tasks:

#### 5. Project Setup (2 hours)
- [ ] Initialize Vite + React + TypeScript
- [ ] Configure Tailwind CSS
- [ ] Set up React Router
- [ ] Create Zustand store
- [ ] Configure Axios for API calls

#### 6. Unlock Screen (2 hours)
- [ ] Passphrase input form
- [ ] "Create new passphrase" flow for first-time users
- [ ] Error handling for wrong passphrase
- [ ] Loading state while decrypting

#### 7. Import Wizard (3 hours)
- [ ] File upload component
- [ ] Drag-and-drop support
- [ ] Progress indicator during parsing
- [ ] Import summary display (X RSUs, Y options imported)
- [ ] Error handling for invalid files

#### 8. Dashboard Layout (2 hours)
- [ ] Header with IBM logo and current price
- [ ] Navigation sidebar
- [ ] Main content area
- [ ] Responsive design (mobile-friendly)

#### 9. Summary Tiles (3 hours)
- [ ] Total Equity Value tile
- [ ] RSU Value tile (vested + unvested)
- [ ] Options Intrinsic Value tile
- [ ] ESPP Value tile
- [ ] Today's Gain/Loss tile (future feature, show $0 for MVP)
- [ ] Auto-refresh every 30 seconds

**Tile Component:**
```tsx
interface SummaryTileProps {
  title: string;
  value: number;
  subtitle?: string;
  trend?: "up" | "down" | "neutral";
}

function SummaryTile({ title, value, subtitle, trend }: SummaryTileProps) {
  return (
    <div className="bg-white rounded-lg shadow p-6">
      <h3 className="text-sm font-medium text-gray-500">{title}</h3>
      <p className="mt-2 text-3xl font-semibold text-gray-900">
        {formatCurrency(value)}
      </p>
      {subtitle && (
        <p className="mt-1 text-sm text-gray-600">{subtitle}</p>
      )}
    </div>
  );
}
```

#### 10. Holdings Tables (4 hours)
- [ ] RSU table with columns: Grant #, Date, Units, Vested, Unvested, Value
- [ ] Options table with columns: Grant #, Strike, Vested, Outstanding, Intrinsic Value
- [ ] ESPP table with columns: Purchase Date, Shares, Purchase Price, Current Value, Gain
- [ ] Sortable columns
- [ ] Responsive design (stack on mobile)

#### 11. What-If Calculator (2 hours)
- [ ] Price input field
- [ ] Real-time calculation (no API call)
- [ ] Display updated values for all holdings
- [ ] Reset button to return to current price

**What-If Logic:**
```typescript
function calculateWhatIf(price: number, holdings: Holdings) {
  const rsuValue = holdings.rsus.reduce((sum, rsu) => 
    sum + (rsu.vested_units + rsu.unvested_units) * price, 0
  );
  
  const optionsValue = holdings.options.reduce((sum, opt) => 
    sum + Math.max(0, price - opt.strike_price) * opt.vested, 0
  );
  
  const esppValue = holdings.espp.reduce((sum, purchase) => 
    sum + purchase.shares_held * price, 0
  );
  
  return {
    total: rsuValue + optionsValue + esppValue,
    rsuValue,
    optionsValue,
    esppValue
  };
}
```

### Testing & Documentation:

#### 12. MVP Testing (2 hours)
- [ ] Test Excel import with spoof data
- [ ] Verify all calculations match Excel formulas
- [ ] Test passphrase unlock/lock flow
- [ ] Test what-if calculator accuracy
- [ ] Cross-browser testing (Chrome, Safari, Firefox)

#### 13. MVP Documentation (2 hours)
- [ ] Update README with setup instructions
- [ ] Document Excel file format requirements
- [ ] Add screenshots of dashboard
- [ ] Write troubleshooting guide

### MVP Success Criteria:
- ✅ Can import `IBM_equity_spoof_data.xlsx` successfully
- ✅ Dashboard shows correct total equity value ($562,227.25)
- ✅ IBM price updates from yfinance
- ✅ What-if calculator shows correct values
- ✅ Passphrase protects database
- ✅ All tables display correct data

### MVP Limitations (Acceptable):
- ⚠️ No charts/graphs yet
- ⚠️ No vesting timeline view
- ⚠️ No tax estimates
- ⚠️ No historical tracking
- ⚠️ Basic UI styling (functional but not polished)

---

## 🎨 Milestone 2: v1.0 (Polish & Core Features)

**Goal**: Production-ready with charts, vesting timeline, and better UX

**Target**: End of Day 6

### New Features:
1. ✅ Value-over-time chart (Recharts)
2. ✅ Vesting timeline with upcoming events
3. ✅ Tax withholding estimates
4. ✅ Compliance disclaimer modal
5. ✅ Polished UI with animations
6. ✅ Export to CSV functionality
7. ✅ Settings page (refresh interval, tax rate)

### Backend Tasks:

#### 1. Historical Tracking (3 hours)
- [ ] Add `price_history` table
- [ ] Store daily snapshots of total value
- [ ] API endpoint for historical data
- [ ] Backfill with yfinance historical prices

**Schema:**
```sql
CREATE TABLE price_history (
    id INTEGER PRIMARY KEY,
    date DATE NOT NULL,
    ibm_price REAL NOT NULL,
    total_equity_value REAL NOT NULL,
    rsu_value REAL NOT NULL,
    options_value REAL NOT NULL,
    espp_value REAL NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(date)
);
```

#### 2. Tax Calculator (2 hours)
- [ ] Implement tax withholding formulas
- [ ] Support configurable tax rate (default 37%)
- [ ] Calculate estimated tax for upcoming vests
- [ ] API endpoint for tax estimates

**Tax Formula:**
```python
def calculate_tax_withholding(
    shares: int,
    fmv_at_vest: float,
    tax_rate: float = 0.37
) -> TaxEstimate:
    """
    Formula: Tax Withheld = Shares × FMV × Tax Rate
    
    For RSUs, employer typically withholds 22% federal + 15.3% FICA
    = 37.3% total (rounded to 37% for simplicity)
    
    User should consult tax professional for actual liability.
    """
    gross_value = shares * fmv_at_vest
    tax_withheld = gross_value * tax_rate
    net_value = gross_value - tax_withheld
    
    return TaxEstimate(
        gross_value=gross_value,
        tax_withheld=tax_withheld,
        net_value=net_value,
        shares_withheld=int(tax_withheld / fmv_at_vest)
    )
```

#### 3. Export Functionality (2 hours)
- [ ] Export holdings to CSV
- [ ] Export vesting schedule to CSV
- [ ] Export price history to CSV
- [ ] Optionally encrypt exports with separate passphrase

### Frontend Tasks:

#### 4. Value Chart (4 hours)
- [ ] Line chart showing equity value over time
- [ ] Toggle between 1M, 3M, 6M, 1Y, All time
- [ ] Hover tooltip with date and value
- [ ] Responsive design
- [ ] Loading skeleton while fetching data

**Chart Component:**
```tsx
import { LineChart, Line, XAxis, YAxis, Tooltip, ResponsiveContainer } from 'recharts';

function ValueChart({ data }: { data: PriceHistory[] }) {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <LineChart data={data}>
        <XAxis dataKey="date" />
        <YAxis />
        <Tooltip formatter={(value) => formatCurrency(value)} />
        <Line 
          type="monotone" 
          dataKey="total_equity_value" 
          stroke="#2563eb" 
          strokeWidth={2}
        />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

#### 5. Vesting Timeline (4 hours)
- [ ] Timeline view of upcoming vests
- [ ] Group by month/quarter
- [ ] Show estimated value and tax withholding
- [ ] Highlight next vest date
- [ ] Filter by grant type (RSU, Options)

**Timeline Component:**
```tsx
function VestingTimeline({ schedule }: { schedule: VestingEvent[] }) {
  const upcoming = schedule.filter(e => e.status === "SCHEDULED");
  const nextVest = upcoming[0];
  
  return (
    <div className="space-y-4">
      {upcoming.map(event => (
        <div 
          key={event.id}
          className={cn(
            "border-l-4 pl-4 py-2",
            event.id === nextVest.id ? "border-blue-500" : "border-gray-300"
          )}
        >
          <div className="flex justify-between">
            <div>
              <p className="font-medium">{event.grant_number}</p>
              <p className="text-sm text-gray-600">
                {formatDate(event.vest_date)}
              </p>
            </div>
            <div className="text-right">
              <p className="font-semibold">{formatCurrency(event.net_value)}</p>
              <p className="text-sm text-gray-600">
                {event.units_or_options} units
              </p>
            </div>
          </div>
        </div>
      ))}
    </div>
  );
}
```

#### 6. Compliance Disclaimer (2 hours)
- [ ] Modal that shows on first launch
- [ ] Checkbox: "I understand and agree"
- [ ] Link to IBM insider trading policy
- [ ] Persistent footer with disclaimer link

**Disclaimer Text:**
```
⚠️ IMPORTANT LEGAL NOTICE

This application is for informational purposes only and does not constitute 
financial, legal, or tax advice.

Before making any trading decisions:
1. Check IBM's insider trading policy and blackout window status
2. Verify if you are subject to SEC Section 16 reporting requirements
3. Obtain pre-clearance if required by your role
4. Consult a tax professional for actual tax liability
5. Review Morgan Stanley at Work Terms of Service

This application is read-only and does not place trades or modify your accounts.

By using this application, you acknowledge that you are solely responsible 
for compliance with all applicable laws and regulations.
```

#### 7. Settings Page (3 hours)
- [ ] Configure price refresh interval (1min, 5min, 15min)
- [ ] Set tax withholding rate (default 37%)
- [ ] Toggle auto-lock (future feature)
- [ ] Change passphrase
- [ ] Export/import settings

#### 8. UI Polish (4 hours)
- [ ] Add loading skeletons for all data fetches
- [ ] Smooth transitions and animations
- [ ] Error boundaries for graceful failures
- [ ] Toast notifications for actions
- [ ] Dark mode support (optional)

### v1.0 Success Criteria:
- ✅ Chart displays historical equity value
- ✅ Vesting timeline shows next 12 months
- ✅ Tax estimates match Excel formulas
- ✅ Disclaimer shown and acknowledged
- ✅ Can export all data to CSV
- ✅ Settings persist across sessions
- ✅ UI feels polished and professional

---

## 🚀 Milestone 3: v2.0 (Advanced Features)

**Goal**: Power-user features and automation

**Target**: End of Day 10

### New Features:
1. ✅ Automated browser scraping (Playwright) for Morgan Stanley
2. ✅ Multi-currency support (for international employees)
3. ✅ Performance-based RSU modeling (multipliers)
4. ✅ Dividend tracking and projections
5. ✅ Advanced analytics (IRR, cost basis, holding period)
6. ✅ Backup/restore with encryption
7. ✅ Desktop app wrapper (Tauri)

### Backend Tasks:

#### 1. Playwright Automation (6 hours)
- [ ] Implement headless browser login to Morgan Stanley
- [ ] Navigate to Holdings page
- [ ] Extract data from DOM
- [ ] Handle MFA prompts
- [ ] Respect rate limits (1 request per minute)
- [ ] Store credentials securely (OS keychain)

**⚠️ WARNING**: This may violate Morgan Stanley ToS. User must review and accept risk.

**Automation Flow:**
```python
async def scrape_morgan_stanley(username: str, password: str) -> Holdings:
    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=True)
        page = await browser.new_page()
        
        # Login
        await page.goto("https://www.morganstanley.com/spc/...")
        await page.fill("#username", username)
        await page.fill("#password", password)
        await page.click("#login-button")
        
        # Wait for MFA (if required)
        await page.wait_for_selector("#mfa-code", timeout=60000)
        # User must enter MFA code manually
        
        # Navigate to Holdings
        await page.goto("https://www.morganstanley.com/spc/holdings")
        
        # Extract data
        rsus = await page.query_selector_all(".rsu-row")
        # ... parse DOM
        
        await browser.close()
        return holdings
```

#### 2. Multi-Currency Support (3 hours)
- [ ] Add currency conversion API (exchangerate-api.com)
- [ ] Store user's preferred currency
- [ ] Convert all values on display
- [ ] Handle historical exchange rates

#### 3. Advanced Analytics (4 hours)
- [ ] Calculate IRR (Internal Rate of Return) for each grant
- [ ] Track cost basis for tax purposes
- [ ] Calculate holding period for capital gains
- [ ] Project future dividends

**IRR Formula:**
```python
def calculate_irr(grant: RSUGrant, current_price: float) -> float:
    """
    IRR = (Current Value / Grant Value) ^ (1 / Years) - 1
    
    Example:
    - Granted 100 RSUs at $130 on 2021-04-15
    - Current price: $247.85 on 2026-05-26
    - Years: 5.12
    - IRR = (24785 / 13000) ^ (1/5.12) - 1 = 13.8% annualized
    """
    grant_value = grant.units_granted * grant.grant_price
    current_value = grant.units_granted * current_price
    years = (datetime.now() - grant.grant_date).days / 365.25
    
    if years == 0 or grant_value == 0:
        return 0.0
    
    irr = (current_value / grant_value) ** (1 / years) - 1
    return irr
```

#### 4. Backup/Restore (3 hours)
- [ ] Export encrypted database to file
- [ ] Import from encrypted backup
- [ ] Verify backup integrity (checksum)
- [ ] Automatic daily backups (optional)

### Frontend Tasks:

#### 5. Analytics Dashboard (4 hours)
- [ ] IRR chart by grant
- [ ] Cost basis table
- [ ] Dividend projection chart
- [ ] Tax lot optimizer (which shares to sell first)

#### 6. Automation Settings (3 hours)
- [ ] Configure Morgan Stanley credentials (stored in OS keychain)
- [ ] Schedule automatic imports (daily, weekly)
- [ ] View import history and logs
- [ ] Manual trigger for immediate import

#### 7. Desktop App (Tauri) (6 hours)
- [ ] Wrap web app in Tauri
- [ ] System tray icon
- [ ] Auto-start on login
- [ ] Native file picker for imports
- [ ] Code signing for macOS

**Tauri Benefits:**
- Single executable, no terminal needed
- Native performance
- Smaller bundle size than Electron
- Better security (no Node.js in frontend)

### v2.0 Success Criteria:
- ✅ Can automatically import from Morgan Stanley (with user consent)
- ✅ Analytics show IRR and cost basis
- ✅ Backup/restore works reliably
- ✅ Desktop app runs as native application
- ✅ All features from MVP and v1.0 still work

---

## 📋 Development Workflow

### Daily Standup (Self-Check):
1. What did I complete yesterday?
2. What will I work on today?
3. Any blockers or questions?

### Testing Strategy:
- **Unit tests**: All calculation functions (pytest for backend, Vitest for frontend)
- **Integration tests**: API endpoints with test database
- **E2E tests**: Critical user flows (Playwright)
- **Manual testing**: Use spoof data for every feature

### Git Workflow:
```bash
# Feature branches
git checkout -b feature/excel-parser
# ... make changes
git commit -m "feat: implement Excel parser for RSU grants"
git checkout main
git merge feature/excel-parser

# Milestone tags
git tag -a v0.1-mvp -m "MVP: Core functionality complete"
git tag -a v1.0 -m "v1.0: Charts and vesting timeline"
git tag -a v2.0 -m "v2.0: Automation and advanced analytics"
```

### Code Review Checklist:
- [ ] All calculations match documented formulas
- [ ] No hardcoded credentials or API keys
- [ ] Error handling for all external calls
- [ ] TypeScript types are strict (no `any`)
- [ ] Security best practices followed
- [ ] Documentation updated

---

## 🎯 Definition of Done (Each Milestone)

### MVP:
- [ ] All MVP features implemented and tested
- [ ] Can import spoof data successfully
- [ ] Dashboard displays correct values
- [ ] README has setup instructions
- [ ] No critical bugs

### v1.0:
- [ ] All v1.0 features implemented and tested
- [ ] Charts render correctly
- [ ] Tax estimates are accurate
- [ ] UI is polished and responsive
- [ ] User documentation complete

### v2.0:
- [ ] All v2.0 features implemented and tested
- [ ] Automation works (with user consent)
- [ ] Desktop app builds and runs
- [ ] Backup/restore tested
- [ ] Full documentation including compliance guide

---

## 🚦 Risk Mitigation

### Technical Risks:
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| SQLCipher installation issues | Medium | High | Provide Docker alternative |
| yfinance API changes | Low | Medium | Add fallback to Alpha Vantage |
| Excel schema variations | High | Medium | Flexible parser with validation |
| Playwright browser issues | Medium | Low | Manual import always available |

### Legal Risks:
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Morgan Stanley ToS violation | Medium | High | Warn user, make automation opt-in |
| Insider trading violation | Low | Critical | Prominent disclaimer, no trade execution |
| Tax calculation errors | Medium | Medium | Disclaimer: "estimates only, consult CPA" |

---

## 📚 Documentation Deliverables

### For Each Milestone:
1. **README.md** - Setup and usage instructions
2. **CHANGELOG.md** - What's new in this version
3. **API.md** - Backend API documentation (auto-generated by FastAPI)
4. **FORMULAS.md** - All calculation formulas with examples
5. **TROUBLESHOOTING.md** - Common issues and solutions

### Final Documentation:
1. **USER_GUIDE.md** - Complete user manual with screenshots
2. **DEVELOPER_GUIDE.md** - How to contribute or extend
3. **COMPLIANCE.md** - Legal disclaimers and IBM policy guidance
4. **BACKUP.md** - Backup and restore procedures

---

## ✅ Next Steps

Please review this implementation plan and let me know:

1. Does the milestone breakdown make sense?
2. Are the time estimates realistic for your schedule?
3. Should we start with Milestone 0 (project setup)?
4. Any features you want to prioritize or deprioritize?

Once approved, I can switch to **Code mode** to begin implementation, or we can discuss any adjustments to the plan first.