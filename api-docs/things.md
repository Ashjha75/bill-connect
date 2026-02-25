This is a **from-scratch technical blueprint** for building the Bill Connect API. It details the exact **Tools (Tech Stack)** you need to install and the **Sequential Component Architecture** (what you will build, layer by layer).

---

### 🛠️ Part 1: The Tech Stack (The "Arsenal")

These are the specific libraries and tools you will use in your `pom.xml` (Maven) or `build.gradle`.

| Category | Tool / Library | Why we are using it? |
| :--- | :--- | :--- |
| **Core Framework** | **Spring Boot 3.5+** (Java 21 LTS) | Access to Virtual Threads, latest security patches, and long-term support. |
| **Database** | **PostgreSQL** | Best for relational financial data (ACID compliant) and JSONB support. |
| **ORM** | **Spring Data JPA (Hibernate)** | To interact with the DB using Java Objects (Entities). |
| **Migrations** | **Flyway** | Version control for your database (V1__Init.sql, V2__Add_Biller.sql). |
| **Security** | **Spring Security 6** | The standard for AuthN/AuthZ. |
| **Tokens** | **JJWT (Java JWT)** | To generate and parse JSON Web Tokens securely. |
| **Caching/NoSQL** | **Redis (via Redisson)** | For caching, Rate Limiting, and Distributed Locks (prevents double payments). |
| **Boilerplate** | **Lombok** | Removes Getters/Setters/Constructors code. |
| **Mapping** | **MapStruct** | **Crucial.** Converts Entities $\leftrightarrow$ DTOs efficiently at compile time. |
| **Storage** | **AWS SDK (S3)** | To store KYC documents and bulk CSV files securely. |
| **Validation** | **Hibernate Validator** | For `@NotNull`, `@Email`, `@Size` annotations on DTOs. |
| **Testing** | **JUnit 5 + Mockito + Testcontainers** | For Unit tests and Integration tests with a real Dockerized DB. |
| **Docs** | **SpringDoc OpenAPI (Swagger)** | Auto-generates API documentation and UI. |

---

### 🏗️ Part 2: The Structural Sequence (The "Blueprint")

This is the order in which the system is architected and built.

#### 1. 📂 Domain Models (The Data Layer)
*This is the foundation. We define what our data looks like.*
*   **BaseEntity:** An abstract class with ID, `createdAt`, `updatedAt` (using JPA Auditing).
*   **User Entity:** `email`, `password_hash`, `role` (Enum: ADMIN, BILLER, CUSTOMER).
*   **Biller Entity:** `businessName`, `status` (PENDING/ACTIVE), `kycDocUrl`.
*   **Bill Entity:** `amount`, `status` (PAID/UNPAID), `dueDate`, `customerId`.
*   **Transaction Entity:** `txnId`, `amount`, `fee`, `netAmount`, `paymentGatewayResponse`.

#### 2. 🛡️ Security Configuration (The "Fortress")
*We build this early so every API is secure by default.*
*   **SecurityFilterChain:** Disables basic auth, enables CORS, sets session to STATELESS.
*   **JwtAuthenticationFilter:** Intercepts every request, extracts the "Bearer Token," validates signature, and sets the User Context.
*   **PasswordEncoder:** `BCryptPasswordEncoder` (Strength 12) for hashing passwords.
*   **CorsConfiguration:** Allows frontend (React/Angular) to talk to the backend.

#### 3. 🚨 Global Error Handling (The "Safety Net")
*Instead of crashing with a stack trace, we return clean JSON.*
*   **@ControllerAdvice:** Catches exceptions globally.
*   **ErrorResponse DTO:**
    ```json
    {
      "timestamp": "2024-02-25T10:00:00",
      "status": 400,
      "error": "Validation Failed",
      "message": "Email cannot be empty",
      "path": "/auth/login"
    }
    ```
*   **Custom Exceptions:** `ResourceNotFoundException`, `PaymentFailedException`, `UnauthorizedAccessException`.

#### 4. 🔑 Authentication APIs (The "Entry Point")
*   **Login Service:** Accepts Email/Pass $\to$ Checks DB $\to$ Generates Access (15m) & Refresh (7d) Tokens.
*   **MFA Service:** Uses Google Authenticator logic (TOTP) to generate QR codes.
*   **Redis Whitelist:** Stores Refresh Tokens in Redis to allow "Logout" (by removing them).

#### 5. ⚙️ Admin & Governance APIs
*   **Biller Onboarding:** A generic file upload service (S3) for KYC docs.
*   **State Machine:** Logic to transition Biller from `SUBMITTED` $\to$ `APPROVED`.
*   **Fee Configuration:** Storing generic JSON rules for how much % to charge per biller.

#### 6. ⚡ Biller Operations (Performance Heavy)
*   **Async Processing:**
    *   **Tool:** Spring `@Async` + `CompletableFuture`.
    *   **Logic:** When Biller uploads a 50MB CSV:
        1. Save file to S3.
        2. Return "202 Accepted" immediately.
        3. Background thread parses CSV and inserts rows in batches (batch size: 1000).
*   **MapStruct:** Converts the CSV rows into `CustomerDTO` and then `CustomerEntity`.

#### 7. 💳 Customer & Payment (The "Core Logic")
*   **Bill Fetching:**
    *   **Caching:** `@Cacheable(value = "bills", key = "#accountNo")`.
    *   **Logic:** Check Redis first. If missing, check DB, then save to Redis.
*   **Payment Execution (Idempotency):**
    *   **Redis Lock:** `redisson.getLock("bill:" + billId)`.
    *   **Transaction:** `@Transactional`.
    *   **Flow:** Lock Bill $\to$ Validate Status $\to$ Call Gateway $\to$ Update DB $\to$ Unlock.

#### 8. 🔍 AOP & Auditing (The "Watcher")
*   **Aspect Oriented Programming (AOP):**
    *   **@LogExecutionTime:** Measures how long APIs take.
    *   **@AuditLog:** Intercepts sensitive methods (like `approveBiller`) and saves a record to the `audit_logs` table (Who, What, IP Address).

#### 9. 🧪 Testing Strategy
*   **Unit Tests:** Test Services using `Mockito` (Mock the DB).
*   **Integration Tests:** Test Controllers using `Testcontainers`. Spin up a real PostgreSQL Docker container during the test to ensure SQL queries actually work.

---

### 🚀 Summary: The Development Sequence

1.  **Setup:** Initialize Spring Boot with dependencies.
2.  **Database:** Create `User`, `Role`, `Biller` tables via Flyway.
3.  **Security:** Implement JWT Logic and Filter Chain.
4.  **Base APIs:** Build Auth (Login/Register).
5.  **Admin Logic:** Build CRUD for Billers + S3 Upload.
6.  **Biller Logic:** Implement CSV Import + Async Jobs.
7.  **Customer Logic:** Implement Bill Fetch + Redis Caching.
8.  **Payment Logic:** Implement Idempotent Payment + Distributed Locking.
9.  **Refine:** Add AOP Logging, Global Error Handling, and Swagger Docs.
10. **Test:** Write JUnit tests for critical paths.