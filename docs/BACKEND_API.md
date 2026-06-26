# Samsung Go Business — Backend API Specification

Complete REST API specification for Samsung Go Business backend (Node.js/Express/TypeScript).

## API Overview

The backend provides REST endpoints for:
- Authentication & authorization
- Account management
- Transaction processing
- ACH integration
- Analytics & reporting (aggregated, anonymized)
- Payroll provider integration

**Base URL**: `https://api.samsunggobusiness.com/v1`

---

## Authentication

### JWT Token-Based Authentication

All endpoints (except `/auth/login` and `/auth/register`) require a valid JWT bearer token.

```
Authorization: Bearer <jwt_token>
```

### Token Structure

```typescript
interface JWTPayload {
  userId: string;
  email: string;
  accountId: string;
  iat: number;
  exp: number;
  aud: 'samsung-go-mobile' | 'payroll-provider-dashboard';
}
```

### Token Lifecycle
- **Access Token**: 1 hour validity
- **Refresh Token**: 30 days validity (stored secure, httpOnly cookie)
- **Automatic Refresh**: Client handles token refresh before expiration

---

## Endpoints

### Authentication (`/auth`)

#### **POST `/auth/register`**
Register a new user (employee).

**Request**
```json
{
  "email": "john.doe@example.com",
  "password": "SecurePassword123!",
  "firstName": "John",
  "lastName": "Doe",
  "ssn": "123-45-6789",
  "dateOfBirth": "1990-01-15"
}
```

**Response** (201 Created)
```json
{
  "userId": "user_123abc",
  "email": "john.doe@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "ref_456def...",
  "requiresMFA": true
}
```

**Errors**
- `400`: Invalid email or password format
- `409`: Email already registered
- `422`: KYC validation failed (invalid SSN, etc.)

---

#### **POST `/auth/login`**
Authenticate user and receive JWT tokens.

**Request**
```json
{
  "email": "john.doe@example.com",
  "password": "SecurePassword123!"
}
```

**Response** (200 OK)
```json
{
  "userId": "user_123abc",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "ref_456def...",
  "requiresMFA": true,
  "mfaRequired": true,
  "mfaMethods": ["sms", "authenticator"]
}
```

---

#### **POST `/auth/verify-mfa`**
Verify MFA code (SMS or authenticator app).

**Request**
```json
{
  "code": "123456",
  "method": "sms"
}
```

**Response** (200 OK)
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "ref_456def...",
  "requiresMFA": false
}
```

---

#### **POST `/auth/refresh`**
Refresh expired access token using refresh token.

**Request**
```json
{
  "refreshToken": "ref_456def..."
}
```

**Response** (200 OK)
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "ref_789ghi..."
}
```

---

#### **POST `/auth/logout`**
Invalidate current session and refresh token.

**Request**
```json
{
  "refreshToken": "ref_456def..."
}
```

**Response** (200 OK)
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

### Account Management (`/accounts`)

#### **GET `/accounts/{accountId}`**
Retrieve account details and current balance.

**Headers**
```
Authorization: Bearer <jwt_token>
```

**Response** (200 OK)
```json
{
  "accountId": "acc_123abc",
  "userId": "user_123abc",
  "accountNumber": "****1234",
  "routingNumber": "011000015",
  "balance": 2500.50,
  "currency": "USD",
  "status": "active",
  "createdAt": "2026-01-15T10:30:00Z",
  "kycStatus": "verified",
  "verificationMethod": "micro-deposits",
  "linkedAccounts": [
    {
      "accountId": "linked_456def",
      "bankName": "Chase Bank",
      "accountLast4": "1234",
      "accountType": "checking",
      "verified": true,
      "verifiedAt": "2026-01-15T10:30:00Z"
    }
  ],
  "splits": {
    "fee": 2.5,
    "allocations": [
      {
        "allocationId": "split_001",
        "linkedAccountId": "linked_456def",
        "percentage": 60,
        "isDefault": true
      },
      {
        "allocationId": "split_002",
        "linkedAccountId": "savings_789ghi",
        "percentage": 40,
        "isDefault": false
      }
    ]
  }
}
```

---

#### **PUT `/accounts/{accountId}/splits`**
Update split allocation for deposits.

**Request**
```json
{
  "allocations": [
    {
      "linkedAccountId": "linked_456def",
      "percentage": 70
    },
    {
      "linkedAccountId": "savings_789ghi",
      "percentage": 30
    }
  ]
}
```

**Response** (200 OK)
```json
{
  "success": true,
  "allocations": [
    {
      "allocationId": "split_001",
      "linkedAccountId": "linked_456def",
      "percentage": 70,
      "updated": true
    },
    {
      "allocationId": "split_002",
      "linkedAccountId": "savings_789ghi",
      "percentage": 30,
      "updated": true
    }
  ]
}
```

**Errors**
- `400`: Allocations don't sum to 100%
- `409`: One or more linked accounts are no longer verified
- `422`: Invalid account configuration

---

### Transactions (`/transactions`)

#### **GET `/transactions`**
Retrieve transaction history (paginated).

**Query Parameters**
```
?limit=20&offset=0&status=all&type=all&startDate=2026-01-01&endDate=2026-12-31
```

**Response** (200 OK)
```json
{
  "transactions": [
    {
      "transactionId": "txn_001",
      "accountId": "acc_123abc",
      "type": "deposit",
      "amount": 1500.00,
      "currency": "USD",
      "status": "settled",
      "source": "ach",
      "splits": [
        {
          "destinationAccount": "linked_456def",
          "amount": 900.00,
          "percentage": 60,
          "settled": true
        },
        {
          "destinationAccount": "savings_789ghi",
          "amount": 600.00,
          "percentage": 40,
          "settled": true
        }
      ],
      "fee": 37.50,
      "description": "Paycheck - Acme Corp",
      "timestamp": "2026-01-15T15:30:00Z",
      "settlementDate": "2026-01-17T00:00:00Z"
    }
  ],
  "pagination": {
    "total": 245,
    "limit": 20,
    "offset": 0,
    "hasMore": true
  }
}
```

---

#### **GET `/transactions/{transactionId}`**
Retrieve detailed transaction information.

**Response** (200 OK)
```json
{
  "transactionId": "txn_001",
  "accountId": "acc_123abc",
  "type": "deposit",
  "amount": 1500.00,
  "currency": "USD",
  "status": "settled",
  "source": "ach",
  "achDetails": {
    "achFileId": "achfile_001",
    "batchNumber": "001",
    "companyName": "Acme Corp",
    "employeeId": "EMP123456"
  },
  "splits": [
    {
      "splitId": "split_001",
      "destinationAccount": "linked_456def",
      "amount": 900.00,
      "status": "settled",
      "settlementDate": "2026-01-17T00:00:00Z"
    }
  ],
  "fee": 37.50,
  "feeBreakdown": {
    "processingFee": 2.5,
    "percentageOfDeposit": 2.5
  },
  "timestamp": "2026-01-15T15:30:00Z",
  "auditLog": [
    {
      "event": "deposit_received",
      "timestamp": "2026-01-15T15:30:00Z"
    },
    {
      "event": "splits_applied",
      "timestamp": "2026-01-15T15:35:00Z"
    }
  ]
}
```

---

#### **POST `/transactions/check-deposit`**
Submit a mobile check deposit.

**Request** (multipart/form-data)
```
front_image: <image_file>
back_image: <image_file>
amount: 500.00
```

**Response** (202 Accepted)
```json
{
  "transactionId": "txn_check_001",
  "status": "pending_review",
  "amount": 500.00,
  "estimatedCreditDate": "2026-01-17T00:00:00Z",
  "reviewStatus": "pending_ocr",
  "message": "Check deposit submitted. Review typically completes within 2 hours."
}
```

---

### ACH Integration (`/ach`)

#### **POST `/ach/deposits/process`**
Process incoming ACH deposit file (called by sponsor bank system, not by client).

**Request** (internal system only)
```json
{
  "achFileId": "achfile_001",
  "originatingBank": "011000015",
  "destinationBank": "021101108",
  "batchCount": 1,
  "entries": [
    {
      "transactionCode": "22",
      "receivingDFI": "021101108",
      "accountNumber": "999999999",
      "amount": 150000,
      "individualID": "EMP123456",
      "individualName": "John Doe",
      "originatingCompanyID": "ACMECORP01"
    }
  ]
}
```

**Response** (200 OK)
```json
{
  "achFileId": "achfile_001",
  "processedEntries": 1,
  "failedEntries": 0,
  "totalAmount": 1500.00,
  "status": "processed",
  "deposits": [
    {
      "depositId": "deposit_001",
      "accountId": "acc_123abc",
      "amount": 1500.00,
      "status": "split_pending"
    }
  ]
}
```

---

#### **POST `/ach/deposits/{depositId}/settle`**
Initiate settlement of splits to linked accounts (internal, called by transaction service).

**Response** (200 OK)
```json
{
  "depositId": "deposit_001",
  "settledSplits": 2,
  "totalSettled": 1500.00,
  "achFilesGenerated": 1,
  "status": "settling"
}
```

---

### Analytics (`/analytics`)

#### **GET `/analytics/dashboard`** (Payroll Provider)
Retrieve aggregated, anonymized insights.

**Headers**
```
Authorization: Bearer <payroll_provider_jwt_token>
```

**Query Parameters**
```
?startDate=2026-01-01&endDate=2026-12-31&granularity=daily
```

**Response** (200 OK)
```json
{
  "period": {
    "startDate": "2026-01-01",
    "endDate": "2026-12-31"
  },
  "aggregated": {
    "totalDeposits": 5000000,
    "totalEmployeesOptedIn": 45000,
    "averageDepositAmount": 1250.00,
    "medianDepositAmount": 1000.00,
    "totalFeeGenerated": 125000.00,
    "averageFeePerDeposit": 25.00
  },
  "splitDistribution": {
    "averageSplitCount": 2.3,
    "topDestinations": [
      {
        "destinationType": "checking_account",
        "percentage": 55,
        "anonymized": true
      },
      {
        "destinationType": "savings_account",
        "percentage": 30,
        "anonymized": true
      },
      {
        "destinationType": "investment_account",
        "percentage": 15,
        "anonymized": true
      }
    ]
  },
  "timing": {
    "averageDepositPerDay": 13698.63,
    "peakDayOfWeek": "Friday",
    "averageTimeFromACHReceiptToSplit": "2.3 hours"
  },
  "quality": {
    "successRate": 99.8,
    "failedDeposits": 10,
    "failureReasons": {
      "invalid_account": 5,
      "compliance_block": 3,
      "system_error": 2
    }
  },
  "compliance": {
    "fraudiousTransactions": 0,
    "kycVerifiedEmployees": 45000,
    "pendingVerification": 200,
    "declined": 15
  },
  "timeseries": [
    {
      "date": "2026-01-01",
      "deposits": 45,
      "amount": 56250.00,
      "averageAmount": 1250.00
    }
  ]
}
```

---

#### **GET `/analytics/compliance`** (Payroll Provider)
Compliance and regulatory metrics.

**Response** (200 OK)
```json
{
  "reporting": {
    "ctrs_filed": 8,
    "sars_filed": 0,
    "kyc_verifications": 45000,
    "aml_checks": 45000
  },
  "disputes": {
    "totalDisputes": 12,
    "resolved": 10,
    "pending": 2,
    "averageResolutionDays": 3.5
  },
  "security": {
    "dataBreaches": 0,
    "fraudAttempts": 23,
    "fraudSuccessRate": 0,
    "encryptionStatus": "full"
  }
}
```

---

### Payroll Provider Integration (`/payroll-provider`)

#### **GET `/payroll-provider/employees`** (Payroll Provider Dashboard)
List employees who have elected Samsung Go (paginated, anonymized).

**Query Parameters**
```
?limit=50&offset=0&status=all
```

**Response** (200 OK)
```json
{
  "employees": [
    {
      "employeeId": "EMP123456",
      "employeeName": "John D.",
      "department": "Engineering",
      "status": "opted_in",
      "accountStatus": "active",
      "totalDeposits": 15,
      "totalAmount": 18750.00,
      "lastDepositDate": "2026-01-15",
      "kycStatus": "verified"
    }
  ],
  "pagination": {
    "total": 45000,
    "limit": 50,
    "offset": 0,
    "hasMore": true
  }
}
```

---

#### **POST `/payroll-provider/bulk-upload`**
Bulk upload payroll provider enrollment data.

**Request** (multipart/form-data)
```
file: <csv_or_json_file>
```

**CSV Format**
```
employee_id,email,first_name,last_name,ssn,dob
EMP001,john@acme.com,John,Doe,123-45-6789,1990-01-15
EMP002,jane@acme.com,Jane,Smith,987-65-4321,1992-06-20
```

**Response** (202 Accepted)
```json
{
  "uploadId": "upload_001",
  "status": "processing",
  "recordsReceived": 2,
  "estimatedCompletionTime": "2026-01-15T16:45:00Z"
}
```

---

## Error Handling

All error responses follow this format:

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Human-readable error message",
    "details": {
      "field": "email",
      "reason": "Invalid email format"
    },
    "requestId": "req_123abc",
    "timestamp": "2026-01-15T15:30:00Z"
  }
}
```

### Common Error Codes

| Code | HTTP Status | Meaning |
|------|-------------|---------|
| `INVALID_REQUEST` | 400 | Malformed request |
| `UNAUTHORIZED` | 401 | Missing or invalid JWT token |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `CONFLICT` | 409 | Resource already exists |
| `RATE_LIMITED` | 429 | Too many requests |
| `KYC_FAILED` | 422 | KYC validation failed |
| `INTERNAL_ERROR` | 500 | Server error |

---

## Rate Limiting

- **Default**: 100 requests per minute per user
- **Burst**: Up to 200 requests per minute (short-term spikes)
- **Headers**:
  ```
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 87
  X-RateLimit-Reset: 1705333800
  ```

---

## Data Privacy & Anonymization

### Customer Data
- **PII**: Encrypted at rest, masked in logs
- **Account Numbers**: Tokenized; only last 4 digits shown to user
- **SSN**: Encrypted; never logged or exposed
- **Transaction Details**: Timestamped, immutable, audited

### Analytics Data
- **Aggregated**: Sum/average/percentile across all employees
- **Anonymized**: No way to link metrics back to individuals
- **Differential Privacy**: Small random noise added to counts (±1-5%)
- **Retention**: Aggregated data kept 7 years; individual records 7 years (compliance)

---

## Database Schema (PostgreSQL)

### Users Table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  ssn_encrypted VARCHAR(255),
  date_of_birth DATE,
  kyc_status ENUM('pending', 'verified', 'failed'),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Accounts Table
```sql
CREATE TABLE accounts (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  account_number_encrypted VARCHAR(255),
  routing_number VARCHAR(9),
  balance DECIMAL(12, 2),
  status ENUM('active', 'frozen', 'closed'),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Transactions Table
```sql
CREATE TABLE transactions (
  id UUID PRIMARY KEY,
  account_id UUID REFERENCES accounts(id),
  type ENUM('deposit', 'withdrawal', 'transfer'),
  amount DECIMAL(12, 2),
  status ENUM('pending', 'settled', 'failed'),
  source ENUM('ach', 'check_deposit', 'transfer'),
  description VARCHAR(255),
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (account_id, timestamp)
);
```

### Split Allocations Table
```sql
CREATE TABLE split_allocations (
  id UUID PRIMARY KEY,
  account_id UUID REFERENCES accounts(id),
  linked_account_id UUID,
  percentage DECIMAL(5, 2),
  is_default BOOLEAN,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Analytics Events Table
```sql
CREATE TABLE analytics_events (
  id BIGSERIAL PRIMARY KEY,
  event_type VARCHAR(100),
  account_id UUID REFERENCES accounts(id),
  event_data JSONB,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX (event_type, timestamp)
);
```

---

## Deployment & Infrastructure

### Environment Variables
```
DATABASE_URL=postgresql://user:pass@localhost/samsung-go-prod
JWT_SECRET=<secret_key>
JWT_REFRESH_SECRET=<refresh_secret>
ACH_PROCESSING_KEY=<ach_key>
SPONSOR_BANK_API_URL=https://api.sponsorbank.com
STRIPE_API_KEY=<stripe_key>
FIREBASE_PROJECT_ID=samsung-go-business
LOG_LEVEL=info
NODE_ENV=production
```

### Docker Deployment
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

---

**Status**: API specification complete  
**Next**: Dashboard implementation documentation
