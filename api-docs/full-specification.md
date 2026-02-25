This is a comprehensive breakdown of the **Bill Connect API** endpoints, categorized by module. This structure aligns strictly with the "Bill Connect" requirements, ensuring support for the **Admin Governance**, **Biller Operations**, **Customer Experience**, and **Security** layers.

### **Base URL:** `https://api.billconnect.com/api/v1`

---

### **1. Authentication & Security Module (Global)**
*Handles Login, MFA, and Session Management.*

| Method | Endpoint | API Name | Description | Security |
| :--- | :--- | :--- | :--- | :--- |
| **POST** | `/auth/login` | User Login | Authenticates Admin or Biller staff via Email/Password. Returns JWT Access & Refresh Tokens. | 🔓 Public |
| **POST** | `/auth/refresh` | Refresh Token | Generates a new Access Token using a valid Refresh Token. | 🔓 Public |
| **POST** | `/auth/logout` | Logout | Invalidates the current session/token (adds to Redis blacklist). | 🔒 Secured |
| **POST** | `/auth/mfa/generate` | Generate MFA | Generates a QR Code (TOTP secret) for 2FA setup (Google Authenticator). | 🔒 Secured |
| **POST** | `/auth/mfa/verify` | Verify MFA | Verifies the 6-digit code to enable 2FA or complete login. | 🔒 Secured |
| **POST** | `/auth/password/reset-request` | Forgot Password | Sends a password reset link to the registered email. | 🔓 Public |
| **POST** | `/auth/password/reset` | Reset Password | Sets a new password using the token received in email. | 🔓 Public |

---

### **2. Administrator Module (SVL Staff)**
*Focus: Governance, Biller Onboarding, Financial Config, and System Health.*

| Method | Endpoint | API Name | Description | Security |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/admin/dashboard` | Admin Dashboard | Fetches global KPIs: TTV (Total Transaction Volume), Active Billers, Revenue. | 🔒 Admin |
| **POST** | `/admin/users` | Create Admin User | Invites a new internal SVL staff member (sends email invite). | 🔒 Super Admin |
| **GET** | `/admin/audit-logs` | System Audit Log | Searchable log of all system actions (Who did What and When). | 🔒 Auditor/Admin |
| **GET** | `/admin/billers` | List Billers | View all billers with filters (Pending, Active, Suspended). | 🔒 Admin |
| **POST** | `/admin/billers/onboard` | Onboard Biller | Initiates the 6-step wizard. Saves Business Info, Directors, and Bank details. | 🔒 Admin |
| **PUT** | `/admin/billers/{id}/status` | Approve/Reject | Updates Biller status (e.g., `PENDING` → `ACTIVE` or `REJECTED`). | 🔒 Admin |
| **GET** | `/admin/billers/{id}/kyc` | View KYC Docs | Generates secure Presigned URLs to view uploaded KYC documents. | 🔒 Admin |
| **PUT** | `/admin/billers/{id}/fees` | Configure Fees | Sets Platform Fee, GCT, and Convenience Fee rules for a specific Biller. | 🔒 Admin |
| **POST** | `/admin/api-keys` | Generate Partner Key | Creates a new API Key for third-party partners (e.g., Payment Gateways). | 🔒 Admin |
| **PUT** | `/admin/api-keys/{id}/revoke` | Revoke Partner Key | Immediately disables a partner's API access. | 🔒 Admin |
| **GET** | `/admin/reports/settlement` | Settlement Report | Generates a CSV showing net payable amounts to billers. | 🔒 Admin |

---

### **3. Biller Module (Partner Companies)**
*Focus: Customer Management, Billing Cycles, and Reconciliation.*

| Method | Endpoint | API Name | Description | Security |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/biller/dashboard` | Biller Dashboard | Real-time view of Outstanding Receivables, Collections, and Overdue Bills. | 🔒 Biller |
| **POST** | `/biller/customers` | Create Customer | Manually creates a single customer profile (Account No, Name, Phone). | 🔒 Biller |
| **POST** | `/biller/customers/import` | Bulk Import Cust. | Uploads a CSV file to create/update customers in bulk (Async Job). | 🔒 Biller |
| **GET** | `/biller/customers` | List Customers | Searchable list of customers belonging to this Biller. | 🔒 Biller |
| **POST** | `/biller/bills` | Create Bill | Manually creates an invoice (supports line items like "Service Charge"). | 🔒 Biller |
| **POST** | `/biller/bills/import` | Bulk Import Bills | Uploads a CSV to generate thousands of bills at once. | 🔒 Biller |
| **GET** | `/biller/bills` | Bill History | View bills with status filters (Unpaid, Paid, Overdue). | 🔒 Biller |
| **GET** | `/biller/payments` | Payment History | Master list of all received payments with breakdown (Gross, Net, Fees). | 🔒 Biller |
| **GET** | `/biller/payments/{id}` | Txn Details | Detailed view of a specific transaction including tax/fee split. | 🔒 Biller |
| **PUT** | `/biller/settings/notifications`| Update Templates | Customize email subject/body for payment reminders. | 🔒 Biller |

---

### **4. Customer Module (The Payer)**
*Focus: Bill Discovery, Payment Execution, and History.*

| Method | Endpoint | API Name | Description | Security |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/public/billers` | Directory | Lists all available/active Billers (logos, names, categories). | 🔓 Public |
| **POST** | `/customer/bills/fetch` | Fetch Bill | Retreives pending bills using Biller ID and Account Number (Identifier). | 🔓 Public/Guest |
| **POST** | `/customer/payments/preview`| Calc. Fees | Returns the total to pay + fee breakdown (Validation Step before paying). | 🔓 Public/Guest |
| **POST** | `/customer/payments/pay` | **Pay Bill** | Executes the payment. **Idempotent**. Updates Bill Status to `PAID`. | 🔓 Public/Guest |
| **GET** | `/customer/payments/{txnId}` | Receipt | Fetches the payment confirmation/receipt details after success. | 🔓 Public/Guest |
| **GET** | `/customer/history` | My History | (If logged in) Returns past payments made by this user. | 🔒 Customer |

---

### **5. Utility & File Management (Shared)**
*Handles file storage and reference data.*

| Method | Endpoint | API Name | Description | Security |
| :--- | :--- | :--- | :--- | :--- |
| **POST** | `/files/upload/presigned` | Get Upload URL | Generates a temporary AWS S3 Presigned URL for secure file uploading. | 🔒 Secured |
| **GET** | `/files/{fileId}/presigned` | Get Download URL | Generates a temporary URL to view a secure document (e.g., KYC). | 🔒 Admin/Biller |
| **GET** | `/reference/banks` | List Banks | Returns list of supported banks/branches for settlement setup. | 🔓 Public |
| **GET** | `/reference/categories` | Biller Categories | Returns categories (Utilities, Telecom, Loans) for the directory. | 🔓 Public |

---

### **6. Partner API (External Systems)**
*For Kiosks or Third-Party Apps integrating with Bill Connect.*

| Method | Endpoint | API Name | Description | Security |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | `/partner/validate-account` | Validate Account | Checks if a customer account exists and returns current balance. | 🔑 API Key |
| **POST** | `/partner/post-payment` | Post Payment | External system posts a payment. Backend acts as Authoritative Engine. | 🔑 API Key |
| **POST** | `/partner/void-transaction` | Void Payment | Reverses a transaction (if within the allowed time window). | 🔑 API Key |

---

### **Summary of Security Levels**
1.  **🔓 Public:** Accessible by anyone (Rate-limited).
2.  **🔒 Secured:** Requires a valid `Bearer Token` (JWT).
3.  **🔒 Role (Admin/Biller):** Requires JWT *plus* specific role claim in the token.
4.  **🔑 API Key:** Requires `X-API-KEY` header (for machine-to-machine communication).