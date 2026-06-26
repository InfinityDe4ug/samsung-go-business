# Samsung Go Business — Payroll Provider Dashboard

Separate web application (React/TypeScript) for payroll provider partners to view aggregated, anonymized insights into Samsung Go Business adoption and usage patterns.

## Architecture Decision

**Separate Web App** (not integrated into `partner-demo.html`) because:

1. **Different Audience**: Partner dashboards require admin/business logic access; demo is for sales/marketing
2. **Different Data Access**: Partners need authenticated API access; demo is read-only visualization
3. **Separate Permissions**: Partner dashboard requires role-based access control; demo doesn't
4. **Scalability**: Dashboard can scale independently; demo remains lightweight
5. **Compliance**: Analytics data access is audited separately; demo is public-facing

---

## Overview

The Payroll Provider Dashboard is a React/TypeScript web application deployed at:
- **Production**: `https://partners.samsunggobusiness.com`
- **Staging**: `https://staging-partners.samsunggobusiness.com`

It provides:
- **Real-time Analytics**: Aggregated employee adoption metrics
- **Compliance Reporting**: CTR, SAR filings, KYC status
- **Operational Insights**: Deposit patterns, peak days, average amounts
- **Integration Status**: Employee enrollment progress, bulk upload history
- **Security Logs**: Access audit trails, suspicious activity

---

## Data Anonymization Strategy

### Aggregation Levels

All metrics are aggregated by:
- **Geography**: State-level only (never city, ZIP code, or address)
- **Industry**: NAICS industry classification (never company division or department)
- **Time Period**: Daily, weekly, monthly (never hourly)
- **Employee Count**: Rounded to nearest 50 (±25 employees)

### Examples of What IS Shown
```json
{
  "state": "Texas",
  "industry": "Technology Services (NAICS 5415)",
  "period": "2026-01-15 to 2026-01-21",
  "totalDeposits": 1250,
  "totalEmployeesOptedIn": 450,
  "averageDepositAmount": 1150.00
}
```

### Examples of What IS NOT Shown
- ❌ Individual employee names (only "Employee 1", "Employee 2", etc.)
- ❌ Specific company names (anonymized to "Company A", "Company B")
- ❌ Department names (only aggregated to industry)
- ❌ Individual account numbers (only last 4 digits, tokenized)
- ❌ Exact employee count (rounded to nearest 50)
- ❌ Hourly deposit patterns (only daily/weekly aggregates)
- ❌ City/ZIP code information (state-level only)

---

## Data Retention Policy

### Customer Transaction Data
- **Retention Period**: 12 months from transaction date
- **Purge**: Automatic after 12 months via scheduled job
- **Exception**: Data required for regulatory compliance (SAR/CTR) retained 7 years

### Aggregated Analytics Data
- **Retention Period**: Indefinite (no PII, fully anonymized)
- **Purge**: Never (historical trends important for business intelligence)

### Audit Logs
- **Retention Period**: 24 months
- **Purge**: Automatic after 24 months
- **Exception**: Logs involving suspicious activity retained 7 years for compliance

### Employee Profile Data (KYC)
- **Retention Period**: 12 months after employee opts out or terminates
- **Purge**: Automatic 12 months after opt-out
- **Encryption**: Always encrypted at rest; never exposed to dashboard

### API Access Logs (Partner Dashboard)
- **Retention Period**: 12 months
- **Purge**: Automatic after 12 months
- **Content**: Login times, IP addresses, queries (no customer data)

---

## Project Structure

```
dashboard/
├── src/
│   ├── pages/
│   │   ├── Dashboard.tsx              (Main landing page)
│   │   ├── Analytics/
│   │   │   ├── OverviewPage.tsx        (Key metrics summary)
│   │   │   ├── GeographyPage.tsx       (State-level breakdown)
│   │   │   ├── IndustryPage.tsx        (Industry classification)
│   │   │   ├── TimingPage.tsx          (Deposit timing patterns)
│   │   │   └── SplitDistributionPage.tsx (Split allocation patterns)
│   │   │
│   │   ├── Compliance/
│   │   │   ├── ReportingPage.tsx       (CTR, SAR, KYC status)
│   │   │   ├── DisputesPage.tsx        (Fraud disputes & resolution)
│   │   │   ├── SecurityPage.tsx        (Breach history, fraud attempts)
│   │   │   └── AuditLogPage.tsx        (Access audit trail)
│   │   │
│   │   ├── Integration/
│   │   │   ├── EnrollmentPage.tsx      (Employee signup status)
│   │   │   ├── BulkUploadPage.tsx      (Upload payroll data)
│   │   │   ├── UploadHistoryPage.tsx   (Past uploads)
│   │   │   └── SyncStatusPage.tsx      (Real-time sync status)
│   │   │
│   │   ├── Settings/
│   │   │   ├── ProfilePage.tsx         (Company profile)
│   │   │   ├── UsersPage.tsx           (User access management)
│   │   │   ├── IntegrationPage.tsx     (API keys, webhooks)
│   │   │   ├── NotificationsPage.tsx   (Alert preferences)
│   │   │   └── DataRetentionPage.tsx   (Explain retention policy)
│   │   │
│   │   └── Auth/
│   │       ├── LoginPage.tsx
│   │       ├── RegisterPage.tsx
│   │       ├── MFAPage.tsx
│   │       └── LogoutPage.tsx
│   │
│   ├── components/
│   │   ├── Layout/
│   │   │   ├── Header.tsx              (Logo, user menu, notifications)
│   │   │   ├── Sidebar.tsx             (Navigation menu)
│   │   │   ├── Footer.tsx              (Copyright, links)
│   │   │   └── Layout.tsx              (Wrapper component)
│   │   │
│   │   ├── Common/
│   │   │   ├── Card.tsx                (Generic card component)
│   │   │   ├── Button.tsx              (Styled buttons)
│   │   │   ├── Input.tsx               (Form input)
│   │   │   ├── Select.tsx              (Dropdown)
│   │   │   ├── DateRange.tsx           (Date range picker)
│   │   │   ├── Loading.tsx             (Spinner)
│   │   │   ├── Error.tsx               (Error message)
│   │   │   └── Modal.tsx               (Modal dialog)
│   │   │
│   │   ├── Analytics/
│   │   │   ├── MetricsCard.tsx         (KPI card with trend)
│   │   │   ├── AreaChart.tsx           (Time-series chart)
│   │   │   ├── BarChart.tsx            (Categorical comparison)
│   │   │   ├── PieChart.tsx            (Distribution)
│   │   │   ├── Table.tsx               (Data table with sorting)
│   │   │   └── ExportButton.tsx        (CSV/PDF export)
│   │   │
│   │   ├── Compliance/
│   │   │   ├── ReportingCard.tsx       (CTR/SAR summary)
│   │   │   ├── ComplianceTimeline.tsx  (Historical filings)
│   │   │   ├── FraudIndicator.tsx      (Fraud metrics badge)
│   │   │   └── AuditTable.tsx          (Access logs)
│   │   │
│   │   └── Integration/
│   │       ├── UploadForm.tsx          (CSV/JSON upload)
│   │       ├── UploadProgress.tsx      (Progress bar)
│   │       └── UploadResults.tsx       (Success/error summary)
│   │
│   ├── api/
│   │   ├── dashboardApi.ts             (Axios instance + helpers)
│   │   ├── analytics.api.ts            (Analytics endpoints)
│   │   ├── compliance.api.ts           (Compliance endpoints)
│   │   ├── integration.api.ts          (Integration endpoints)
│   │   ├── auth.api.ts                 (Auth endpoints)
│   │   └── interceptors.ts             (JWT refresh, error handling)
│   │
│   ├── hooks/
│   │   ├── useAuth.ts                  (Authentication context)
│   │   ├── useAnalytics.ts             (Analytics data fetching)
│   │   ├── useComplianceData.ts        (Compliance data fetching)
│   │   ├── useDateRange.ts             (Date range state management)
│   │   ├── usePagination.ts            (Pagination logic)
│   │   └── useLocalStorage.ts          (Persist preferences)
│   │
│   ├── context/
│   │   ├── AuthContext.tsx             (Auth state provider)
│   │   ├── NotificationContext.tsx     (Toast notifications)
│   │   └── ThemeContext.tsx            (Light/dark mode)
│   │
│   ├── types/
│   │   ├── analytics.types.ts          (Analytics DTOs)
│   │   ├── compliance.types.ts         (Compliance DTOs)
│   │   ├── integration.types.ts        (Integration DTOs)
│   │   ├── auth.types.ts               (Auth DTOs)
│   │   └── common.types.ts             (Common types)
│   │
│   ├── utils/
│   │   ├── formatters.ts               (Number, currency, date formatting)
│   │   ├── validators.ts               (Form validation)
│   │   ├── constants.ts                (API URLs, config)
│   │   ├── localStorage.ts             (Client-side storage)
│   │   └── logger.ts                   (Client-side logging)
│   │
│   ├── styles/
│   │   ├── index.css                   (Global styles)
│   │   ├── theme.css                   (Samsung One UI colors)
│   │   ├── components.css              (Component styles)
│   │   └── animations.css              (Transitions)
│   │
│   ├── App.tsx                         (Main app component)
│   ├── index.tsx                       (Entry point)
│   └── vite-env.d.ts
│
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── robots.txt
│
├── tests/
│   ├── pages/
│   │   ├── Analytics.test.tsx
│   │   └── Compliance.test.tsx
│   ├── components/
│   │   ├── MetricsCard.test.tsx
│   │   └── ExportButton.test.tsx
│   └── api/
│       └── dashboardApi.test.ts
│
├── .env.example
├── .env.local                          (Local dev secrets)
├── .gitignore
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

---

## Technology Stack

### Frontend
- **React 18**: UI framework
- **TypeScript**: Type safety
- **Vite**: Fast build tool
- **TailwindCSS**: Utility-first styling
- **React Query**: Server state management
- **Recharts**: Data visualization
- **React Router v6**: Client-side routing
- **Axios**: HTTP client

### Testing
- **Vitest**: Unit testing
- **React Testing Library**: Component testing
- **MSW**: Mock Service Worker (API mocking)

### Development
- **ESLint**: Code linting
- **Prettier**: Code formatting
- **Husky**: Git hooks

### Deployment
- **Vercel** or **AWS S3 + CloudFront**: Static hosting
- **GitHub Actions**: CI/CD pipeline

---

## Features

### 1. Analytics Overview
**Dashboard Page** — Real-time summary of key metrics:
- Total deposits (30-day, YTD)
- Total employees opted in
- Average/median deposit amount
- Average fee generated per deposit
- Split distribution (pie chart)
- Top states (bar chart)
- Top industries (bar chart)
- Trend line (deposits over time)

**Example Metric Card**:
```
┌─────────────────────────┐
│ Total Deposits (30d)    │
│                         │
│  $5,247,500             │
│  ↑ 12.3% vs last month  │
│                         │
│ [View Details →]        │
└─────────────────────────┘
```

### 2. Geographic Analysis
**Geography Page** — State-level breakdown:
- Table: State, # Deposits, Total $, Avg Deposit, % of Total
- US Map visualization (choropleth, color intensity by volume)
- Top 10 states by volume (bar chart)
- Geographic trends (line chart over time)
- Filtering by date range

### 3. Industry Analysis
**Industry Page** — NAICS industry classification:
- Table: Industry (NAICS code), # Deposits, Total $, Avg Deposit, % of Total
- Top 10 industries (bar chart)
- Industry trends (line chart)
- Industry selection filter

### 4. Timing & Patterns
**Timing Page** — Deposit timing analysis:
- Average deposits per day of week (bar chart)
- Peak days/times (heatmap, days × time-of-day)
- Average time from ACH receipt to split (metric)
- Settlement speed distribution (histogram)
- Trends over time (line chart)

### 5. Split Distribution
**Split Distribution Page** — How deposits are allocated:
- Average number of splits per deposit (metric)
- Top 3 split destinations:
  - Checking account (anonymized %)
  - Savings account (anonymized %)
  - Investment account (anonymized %)
- Split percentage distribution (scatter plot)
- Destination type breakdown (pie chart)

### 6. Compliance Reporting
**Reporting Page** — Regulatory compliance metrics:
- CTRs filed (count, date range)
- SARs filed (count, date range)
- KYC verifications (total, pending, declined)
- AML checks (count, pass/fail)
- Historical compliance timeline (interactive)

### 7. Disputes & Fraud
**Disputes Page** — Fraud and dispute management:
- Total disputes (count, status breakdown)
- Resolved disputes (count, avg resolution time)
- Pending disputes (table with details)
- Fraud attempts (count, success rate)
- Fraud prevention summary (metrics)

### 8. Security & Breach History
**Security Page** — Data security metrics:
- Data breaches (count, severity)
- Fraudulent transactions (count, loss amount)
- Fraud attempt rate (% of transactions)
- Encryption status (green/red indicator)
- Security timeline (historical events)
- Recent security alerts (table)

### 9. Audit Logs
**Audit Log Page** — Access audit trail:
- Table: Timestamp, User Email, Action, Resource, IP Address, Status
- Filtering by date, user, action type
- Suspicious activity highlighting
- Download audit report (CSV)

### 10. Integration Status
**Enrollment Page** — Employee adoption tracking:
- Total employees in payroll system (metric)
- Opted-in employees (metric, % adoption)
- Pending verification (count)
- Declined (count, reasons)
- Adoption trends (line chart over time)
- Comparison to industry benchmarks

### 11. Bulk Upload
**Bulk Upload Page** — Import employee payroll data:
- Drag-and-drop file upload (CSV or JSON)
- Template download (pre-formatted CSV)
- File validation before submission
- Upload progress bar
- Real-time sync status

**CSV Format**:
```csv
employee_id,email,first_name,last_name,ssn,dob,department,salary_band
EMP001,john@acme.com,John,Doe,***-**-6789,1990-01-15,Engineering,60k-80k
EMP002,jane@acme.com,Jane,Smith,***-**-4321,1992-06-20,Sales,50k-70k
```

### 12. Upload History
**Upload History Page** — Track past uploads:
- Table: Upload Date, File Name, Records Received, Processed, Failed, Status
- Download upload reports (CSV/JSON)
- Retry failed uploads
- View error details for failed records

### 13. Sync Status
**Sync Status Page** — Real-time sync health:
- Last sync timestamp
- Next scheduled sync
- Sync frequency (configurable)
- Success rate (%)
- Failed records (count, reasons)
- Sync logs (table)

### 14. Account Settings
**Profile Page** — Company information:
- Company name (read-only)
- Industry (read-only)
- Billing contact info
- Phone number
- Address (read-only)
- Federal Tax ID (partial, masked)

### 15. User Access Management
**Users Page** — Role-based access control:
- Table: User Email, Role, Last Login, Status
- Add new user (form)
- Edit user role
- Deactivate/reactivate user
- Roles:
  - **Admin**: All features + user management
  - **Analyst**: Analytics, compliance reports (read-only)
  - **Integrations Manager**: Uploads, sync management
  - **Auditor**: Audit logs, compliance (read-only)

### 16. Integration & API Keys
**Integration Page** — API configuration:
- Show/copy API key (masked until reveal)
- Regenerate API key
- Webhook URL (for real-time sync updates)
- Test webhook (send sample payload)
- API rate limits (show usage)
- Integration docs link

### 17. Notification Preferences
**Notifications Page** — Alert configuration:
- Email alerts: Daily summary, weekly report, security alerts
- In-app notifications: Real-time alerts
- Alert types: High deposit volume, fraud detected, sync failed, KYC declined
- Recipient emails (multiple)
- Frequency settings

### 18. Data Retention Info
**Data Retention Page** — Privacy documentation:
- Explanation of anonymization strategy
- Data retention timelines (table format)
- What data is collected (checklist)
- What data is NOT collected (checklist)
- GDPR/CCPA compliance info
- Data download/deletion request form

---

## API Integration

### Authentication Flow
```
User enters email/password
    ↓
POST /api/auth/login
    ↓
Backend returns: { accessToken, refreshToken, user }
    ↓
Store tokens (accessToken in memory, refreshToken in secure cookie)
    ↓
Redirect to /dashboard
```

### API Endpoints Used

**Analytics**
- `GET /analytics/dashboard?startDate=...&endDate=...&granularity=daily`
- `GET /analytics/geographic?state=...&startDate=...`
- `GET /analytics/industry?industry=...&startDate=...`
- `GET /analytics/timing?startDate=...`
- `GET /analytics/split-distribution?startDate=...`

**Compliance**
- `GET /analytics/compliance?startDate=...&endDate=...`
- `GET /analytics/disputes?startDate=...&status=all`
- `GET /analytics/security?startDate=...`
- `GET /audit-logs?limit=50&offset=0&userId=...&action=...`

**Integration**
- `GET /payroll-provider/employees?limit=50&offset=0&status=all`
- `POST /payroll-provider/bulk-upload` (multipart/form-data)
- `GET /payroll-provider/bulk-uploads?limit=20&offset=0`
- `GET /payroll-provider/sync-status`

**Settings**
- `GET /payroll-provider/profile`
- `GET /payroll-provider/users?limit=50`
- `POST /payroll-provider/users` (add new user)
- `PUT /payroll-provider/users/{userId}` (update role)
- `GET /payroll-provider/api-keys`
- `POST /payroll-provider/api-keys/regenerate`

---

## Data Privacy & Anonymization (Dashboard-Specific)

### What IS Visible
✓ Aggregated metrics (sum, average, count)  
✓ State-level geography  
✓ NAICS industry code  
✓ Time-based trends (daily, weekly, monthly)  
✓ Rounded employee counts (nearest 50)  
✓ Account type (checking, savings, investment)  
✓ Percentage-based distributions  

### What IS NOT Visible
✗ Individual employee names or IDs  
✗ Specific company names (only "Company A")  
✗ Department or team names  
✗ Individual account numbers (only last 4 digits)  
✗ SSN or sensitive personal data  
✗ City, ZIP code, or neighborhood  
✗ Hourly transaction data  
✗ Individual transaction amounts (only aggregated)  
✗ IP addresses or device info (except in audit logs)  

### Differential Privacy
- Small random noise (±1-5%) added to counts
- Employee counts rounded to nearest 50
- Percentages reported to nearest 0.1%
- Salary ranges anonymized to bands (e.g., "50k-70k")

---

## Data Retention Implementation

### Database Strategy

**analytics_events Table** — Indexed by timestamp
```sql
-- Retention: 12 months, then automatic purge
SELECT * FROM analytics_events 
WHERE timestamp < NOW() - INTERVAL '12 months';

-- Daily scheduled job (Kubernetes CronJob or Lambda)
DELETE FROM analytics_events 
WHERE timestamp < NOW() - INTERVAL '12 months';
```

**audit_logs Table** — Indexed by created_at
```sql
-- Retention: 24 months
DELETE FROM audit_logs 
WHERE created_at < NOW() - INTERVAL '24 months'
  AND event_type NOT IN ('security_incident', 'fraud_detected');
```

**transaction_data Table** — Encrypted, indexed by settled_date
```sql
-- Retention: 12 months from settlement
DELETE FROM transaction_data 
WHERE settled_date < NOW() - INTERVAL '12 months'
  AND NOT EXISTS (
    SELECT 1 FROM compliance_holds 
    WHERE transaction_id = transaction_data.id
  );
```

**aggregated_metrics Table** — No PII, retained forever
```sql
-- No purge; historical trends important
-- Data is already anonymized, so retention = indefinite
```

### Retention Cron Jobs

**Daily (2 AM UTC)**:
- Purge analytics events older than 12 months
- Purge API logs older than 12 months

**Weekly (Sunday 3 AM UTC)**:
- Purge audit logs older than 24 months
- Purge transaction data older than 12 months (not under hold)

**Monthly (1st, 3 AM UTC)**:
- Verify all data retention policies
- Send retention report to compliance team

### Example Kubernetes CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: data-purge-daily
spec:
  schedule: "0 2 * * *"  # 2 AM UTC daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: purge-job
            image: node:18-alpine
            command:
            - sh
            - -c
            - |
              node scripts/purge-old-data.js \
                --table=analytics_events \
                --retention=12months \
                --verify
          restartPolicy: OnFailure
```

### Purge Script (Node.js)

```typescript
// scripts/purge-old-data.ts
import { pool } from '../src/database';

async function purgeOldData(table: string, retentionMonths: number) {
  const retentionDate = new Date();
  retentionDate.setMonth(retentionDate.getMonth() - retentionMonths);
  
  const query = `
    DELETE FROM ${table}
    WHERE created_at < $1
    AND NOT EXISTS (
      SELECT 1 FROM compliance_holds 
      WHERE resource_table = $2 
      AND resource_id = ${table}.id
    )
    RETURNING id, created_at;
  `;
  
  try {
    const result = await pool.query(query, [retentionDate, table]);
    console.log(`Purged ${result.rowCount} records from ${table}`);
    
    // Log purge event for audit trail
    await pool.query(
      `INSERT INTO purge_logs (table_name, record_count, purge_date)
       VALUES ($1, $2, NOW())`,
      [table, result.rowCount]
    );
  } catch (error) {
    console.error(`Error purging ${table}:`, error);
    throw error;
  }
}

// Run purge
purgeOldData('analytics_events', 12)
  .then(() => console.log('Purge completed'))
  .catch(err => process.exit(1));
```

---

## Deployment

### Build & Deploy (Vercel)
```bash
# .vercelignore
node_modules/
.env.local
*.md
tests/

# vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "env": {
    "VITE_API_BASE_URL": "@vite_api_base_url"
  },
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://api.samsunggobusiness.com/v1/:path*"
    }
  ],
  "headers": [
    {
      "source": "/:path*",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        }
      ]
    }
  ]
}
```

### Environment Variables (.env.local)
```
VITE_API_BASE_URL=https://api.samsunggobusiness.com/v1
VITE_LOG_LEVEL=info
VITE_ANALYTICS_ID=G-XXXXXXXXXX  # Google Analytics
VITE_SENTRY_DSN=https://key@sentry.io/project  # Error tracking
```

---

## Security & Compliance

### Authentication
- OAuth 2.0 with Google/Microsoft (SSO)
- MFA required for all users
- Session timeout: 30 minutes inactivity
- JWT with short expiry (1 hour)

### Data Protection
- All API calls over HTTPS (TLS 1.3)
- Request signing to prevent tampering
- Rate limiting (100 requests/minute per user)
- CORS restricted to `*.samsunggobusiness.com`

### Audit Trail
- Every API call logged (user, action, timestamp, IP)
- Suspicious activity flagged (brute force attempts, unusual access)
- Audit logs retained 24 months
- Monthly compliance audit

### Compliance
- GDPR-compliant (data retention, right to delete)
- CCPA-compliant (opt-out, data transparency)
- SOC 2 Type II certified
- Annual security audit

---

## Monitoring & Alerts

### Performance Metrics
- Page load time (target: < 2 seconds)
- API response time (target: < 500ms)
- Uptime (target: 99.9%)

### Error Tracking (Sentry)
- Uncaught exceptions
- API errors (4xx, 5xx)
- Network errors
- Performance issues

### Alerts
- API down (page down)
- Error rate > 1% (Slack notification)
- Suspicious login (email alert)
- Bulk upload failure (email to integrations manager)

---

## Testing Strategy

### Unit Tests (Vitest)
- Formatters (currency, date)
- Validators (form input)
- Utils (localStorage, constants)

### Component Tests (React Testing Library)
- MetricsCard (renders data correctly)
- Table (sorting, pagination)
- ExportButton (CSV generation)

### Integration Tests
- Login flow
- Analytics data fetching
- Bulk upload flow

### E2E Tests (Cypress)
- Full user journey (login → analytics → export)
- Error handling (network fail, 401)
- Data persistence (local storage)

---

**Status**: Payroll Provider Dashboard architecture complete  
**Next**: Deploy backend API and integrate both systems
