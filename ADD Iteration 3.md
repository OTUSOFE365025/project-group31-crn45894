# AIDAP Architectural Design – ADD Iteration 3

## Step 1: Review Inputs

### Design Context

From Iteration 2, the **AIDAP Server-Side Backend** is structured as a modular monolith with:

- **Service modules**: `ExamService`, `AnnouncementService`, `AIOrchestrationService`
- **Cross-cutting modules**: `AuthService`, `Security & Privacy Facade`, `Monitoring & Telemetry`
- **Data modules**: `ExamReadRepository`, `ExamWriteRepository`, caches, adapters to LMS/SIS
- **Deployment**: horizontally scaled, stateless backend instances behind an API Gateway, with a shared SQL database and cache.

Iteration 2 ensured basic performance and availability for UC-1 and UC-2, and introduced a **Security & Privacy Facade**, but **QA-4 Privacy/Security** and **CON-6 retention/encryption** are only partially addressed.

This iteration focuses on **Security & Privacy + Data Protection**

### Relevant Functional Requirements / Use Cases

- **UC-1: Retrieve Exam Schedule**  
  - Ensure only the **right student** or authorized staff can see a given schedule.  
  - Enforce data minimization (only required fields) and secure queries.

- **UC-2: Post Course Announcement**  
  - Ensure only instructors / TAs with the correct role can post.  
  - All operations must be **audited** and **attributable** to a real identity from university SSO.

- **UC-4: Deploy System Update**  
  - Security configuration (policies, keys, secrets, retention rules) must **survive rolling / blue-green deployments** and be centrally managed.

### Quality Attribute Drivers

Primary:

- **QA-4 Privacy & Security**  
  Protect all student/lecturer data; enforce access control, auditability, retention, and encryption end-to-end.

Supporting:

- **QA-2 Availability & Scalability**  
  Security mechanisms (auth, authorization, encryption, audit) must **not become a single point of failure** and must scale for up to **5,000 concurrent users**.

- **QA-1 Performance**  
  Security, encryption, and auditing must be designed so UC-1 / UC-2 still meet **< 2s response time** under normal load.

These are classic **architecturally significant requirements (ASRs)** because they strongly shape the structures we choose.

### Constraints

- **CON-1 – University SSO**  
  All authentication must flow through the institution’s identity provider (OIDC/SAML).

- **CON-2 – REST/GraphQL APIs**  
  Security must be enforceable at these boundaries (e.g., tokens, scopes, CORS, schema-level authorization).

- **CON-5 – Cloud-Native, Containerized**  
  Security components must run as **separate deployable services or sidecars** where appropriate; secrets managed through cloud-native mechanisms.

- **CON-6 – Data Retention & Encryption**  
  Enforce institution-defined retention policies and encryption standards at rest and in transit.

---

## Step 2: Establish Iteration Goal by Selecting Drivers

### Iteration 3 Goal

Refine the **Security, Privacy & Data Protection Subsystem** so that:

- All access to student and lecturer data is **authenticated via SSO** and **authorized via centralized policies**.
- Sensitive data is **encrypted at rest and in transit**, with secure key management.
- The system supports **retention policies** (automatic deletion/anonymization) and **audit logging** for security-relevant events.
- These protections **scale** with the number of users and **do not violate** the 2-second response time goal for UC-1 / UC-2.
- Security configurations (policies, keys, retention rules) are **independent of application code deployments**, supporting UC-4.

### Drivers for This Iteration

**Functional drivers**

- UC-1: Retrieve Exam Schedule (secure access to exam data)  
- UC-2: Post Course Announcement (role-based access & audit)  
- UC-4: Deploy System Update (security config survives and supports safe rollout)

**Quality attribute drivers**

- QA-4: Privacy & Security (primary)  
- QA-2: Availability / Scalability (security mechanisms must stay available, support failover)  
- QA-1: Performance (security overhead must be controlled)

**Constraint drivers**

- CON-1: SSO integration  
- CON-2: REST/GraphQL boundaries  
- CON-5: Cloud-native deployment  
- CON-6: Retention & encryption standards

These drivers are the **focus set** for this iteration of ADD.

---

## Step 3: Choose Elements of the System to Decompose

From Iteration 2, we already identified the following server-side modules:

- **Cross-cutting / security-relevant**  
  - `AuthService`  
  - `Security & Privacy Facade`  
  - `Monitoring & Telemetry Module`

- **Data layer & integration**  
  - `ExamReadRepository`, `ExamWriteRepository`  
  - `ExamCache`, `CourseCache`  
  - `LMSAdapter`, `SISAdapter`, `CalendarAdapter`

- **Upstream entry points**  
  - `API Gateway / Request Router`

For Iteration 3 we select a **logical subsystem** that cuts across these modules:

> **Element to Decompose:**  
> **Security, Privacy & Data Protection Subsystem**  
> (comprising `AuthService`, `Security & Privacy Facade`, security-relevant parts of the Data layer, and their integration with the API Gateway and Monitoring.)

**Scope of this element**

- Everything that **decides “who can do what on which data, for how long, and with what guarantees”**, including:
  - Authentication integration with SSO (tokens, sessions).
  - Authorization & policy decision (RBAC / ABAC).
  - Data-level protection (encryption, minimization, retention & anonymization).
  - Security-relevant logging and monitoring.
- We **do not** decompose domain services (e.g., `ExamService`) in this iteration; we treat them as **clients** of this subsystem.

---

## Step 4: Choose Design Concepts

We now choose design concepts (patterns & tactics) that address the above drivers and shape this subsystem.

1. **Externalized Authentication via Identity Provider (IdP) + Token-Based Auth**

   - **Concept:**  
     Use OpenID Connect / OAuth2 with university SSO; `AuthService` becomes a thin adapter around the IdP. API Gateway validates tokens (signatures, expiry, scopes), and passes an `AuthContext` downstream.

   - **Why:**  
     - Satisfies **CON-1** by delegating identity proofing to the university.  
     - Keeps authentication logic **out of business services**, supporting **modifiability** and **security** (QA-4).  
     - Token validation is fast and cacheable, helping keep **latency < 2s** (QA-1).

2. **Centralized Policy-Based Access Control (PBAC / RBAC/ABAC)**

   - **Concept:**  
     Implement a **Policy Decision Point (PDP)** inside the `Security & Privacy Facade`, and **Policy Enforcement Points (PEPs)** at:
     - API Gateway (coarse-grained: “can this user call this endpoint?”)  
     - Service level (fine-grained: “can this user see this specific exam record/field?”).

   - **Why:**  
     - Encapsulates all authorization rules in a single, **policy-driven component**, reducing scattered checks and supporting **QA-4** and **modifiability**.  
     - Makes it easier to change rules (new roles, new data protection laws) without touching domain service code.

3. **Data Minimization & Response Filtering**

   - **Concept:**  
     `Security & Privacy Facade` and Data layer enforce **field-level filtering** based on user role and context. Services request “logical data sets” (e.g., `StudentExamView`), and the facade strips or masks sensitive fields (emails, grades, identifiers) when not needed.

   - **Why:**  
     - Directly supports **privacy-by-design** (QA-4) by limiting exposure of PII.  
     - Keeps the filtering logic centralized instead of being duplicated in each service.

4. **Transparent Data-at-Rest Encryption + Field-Level Encryption**

   - **Concept:**  
     - Use **database-level encryption** (TDE / encrypted volumes) for all data at rest.  
     - Additionally encrypt **high-sensitivity fields** (e.g., tokens, refresh tokens, key references) at the application layer using a **Key Management Service (KMS)** or secrets store.

   - **Why:**  
     - Satisfies **CON-6** and QA-4 by ensuring data remains protected even if storage is compromised.  
     - Combining transparent encryption with selective field-level encryption balances **security vs. performance** (QA-1).

5. **Retention & Anonymization Engine (Policy-Driven Batch Jobs)**

   - **Concept:**  
     - Introduce a `DataRetentionService` that periodically scans data (e.g., by policy tags or timestamps) and either **deletes** or **anonymizes** records.  
     - Policies (e.g., “keep exam logs for X months”) are stored in configuration and interpreted by this engine.

   - **Why:**  
     - Implements **CON-6** and privacy requirements **independently** of domain logic — services just tag data with purpose/ownership.  
     - Isolates retention behavior so it can be adapted if institutional policies change (supports **modifiability**).

6. **Security-Focused Logging & Audit Trail (Cross-Cutting Concern)**

   - **Concept:**  
     - All sensitive operations (`postAnnouncement`, `viewExamSchedule`, admin actions) emit **structured security events** (who/what/when/from where) into a dedicated **append-only audit log** (e.g., log stream, separate DB, or SIEM integration).  
     - Use **correlation IDs** propagated from the API Gateway.

   - **Why:**  
     - Supports QA-4 (accountability, non-repudiation) and aids **incident response**.  
     - Centralized logs make it easier to detect suspicious patterns without changing services.

7. **High-Availability Security & Data Components**

   - **Concept:**  
     - Deploy `AuthService` integration, PDP, and audit logging in **replicated instances** behind the same load balancer as the main backend.  
     - Use **replicated DB** (primary + replicas or multi-AZ) and **clustered cache** for any security-critical data (token blacklists, session state if any).

   - **Why:**  
     - Ensures security components don’t become a single point of failure, supporting **QA-2** (99.5% availability) while still enforcing security rules.

8. **Security as a Dedicated Layer / Module Boundary**

   - **Concept:**  
     - Treat the **Security & Privacy Subsystem** as its own “layer” in the architecture (logically between API Gateway and domain services, and alongside data access).  
     - Services must depend on **abstract security interfaces**, not on concrete IdP or DB details (aligns with **Dependency Inversion**).

   - **Why:**  
     - Reduces coupling and makes it possible to switch IdPs, change policy engine, or move to a different KMS without rewriting domain modules.

# Step 5: Instantiate Architectural Elements, Allocate Responsibilities, and Define Interfaces

## 5.1 Security, Privacy & Data Protection Modules

| **Module**                                       | **Responsibilities**                                                                                                                                                                                  | **Interfaces**                                                                                                                                                                                                                               |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AuthAdapter (IdP Integration)**                | - Integrates with university SSO (OIDC/SAML)<br>- Validates JWT tokens (signatures, expiry, scopes)<br>- Fetches user identity and group/role claims                                                  | - `validateToken(jwt): AuthContext`<br>- `refreshToken(refreshToken): TokenPair`<br>- `fetchUserInfo(authContext): UserProfile`                                                                                                              |
| **Security & Privacy Facade**                    | - Central entry point for authorization & privacy decisions<br>- Performs coarse + fine-grained checks<br>- Applies role/policy rules, redaction, and minimization<br>- Emits structured audit events | - `authorize(authContext, action, resource): Decision`<br>- `filterResponse(authContext, data, viewType): RedactedData`<br>- `onSensitiveAction(authContext, action, resource, outcome)`<br>- `getSecurityContext(request): SecurityContext` |
| **PolicyDecisionService (PDP)**                  | - Evaluates RBAC/ABAC/PBAC authorization rules<br>- Supports endpoint-level and record-level policy checks<br>- Retrieves rules from the PolicyStore                                                  | - `decide(authContext, action, resourceAttrs): Decision`<br>- `explainDecision(id): PolicyTrace`                                                                                                                                             |
| **PolicyStore**                                  | - Stores all policies, roles, and retention configurations<br>- Supports versioning and staged activation                                                                                             | - `getActivePolicies(): PolicySet`<br>- `getPolicyById(policyId): Policy`<br>- `listPoliciesByScope(scope): List<Policy>`                                                                                                                    |
| **DataProtectionService (Crypto / KMS Adapter)** | - Encrypts/decrypts sensitive fields<br>- Uses external KMS / secret manager<br>- Handles key rotation and key alias management                                                                       | - `encrypt(field, plaintext): Ciphertext`<br>- `decrypt(field, ciphertext): Plaintext`<br>- `getKeyMetadata(field): KeyInfo`                                                                                                                 |
| **DataRetentionService**                         | - Executes scheduled deletion/anonymization based on retention policies<br>- Scans DB for expired or tagged data models<br>- Generates retention and policy-compliance reports                        | - `runRetentionCycle(policyId): RetentionReport`<br>- `tagRecord(recordId, policyTag)`<br>- `previewImpact(policyId): ImpactSummary`                                                                                                         |
| **AuditLogService**                              | - Stores structured security audit events<br>- Ensures immutable, append-only logging<br>- Correlates events using request IDs                                                                        | - `recordEvent(authContext, action, resource, outcome, metadata)`<br>- `searchEvents(filter): List<AuditEvent>`                                                                                                                              |
| **SecurityEventMonitor**                         | - Reads audit log stream for anomaly detection<br>- Issues alerts for suspicious behavior patterns                                                                                                    | - `evaluateHealth(): SecurityHealthStatus`<br>- `runDetectionRules(): List<Alert>`                                                                                                                                                           |
| **TokenRevocationStore (Optional)**              | - Maintains a list of invalidated tokens<br>- Used for forced logout or emergency access blocks                                                                                                       | - `isRevoked(tokenId): boolean`<br>- `revokeToken(tokenId, reason)`                                                                                                                                                                          |

---

## 5.2 Interactions with Existing Backend Modules

### API Gateway / Request Router

**Responsibilities**

* Validates JWT using `AuthAdapter`
* Builds `AuthContext` and attaches to request
* Performs coarse-grained authorization using the `Security & Privacy Facade`
* Forwards authorized requests to domain services with an attached `SecurityContext`

**Operational Flow (UC-1 / UC-2)**

1. Receive HTTPS request with token
2. `AuthAdapter.validateToken(jwt)` → produces `AuthContext`
3. `SecurityFacade.authorize(AuthContext, action, resource)` at API boundary
4. If allowed, forward to the domain service

---

# Step 5: Instantiate Architectural Elements, Allocate Responsibilities, and Define Interfaces

## 5.1 Security, Privacy & Data Protection Modules

| **Module**                                       | **Responsibilities**                                                                                                                                                                                  | **Interfaces**                                                                                                                                                                                                                               |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AuthAdapter (IdP Integration)**                | - Integrates with university SSO (OIDC/SAML)<br>- Validates JWT tokens (signatures, expiry, scopes)<br>- Fetches user identity and group/role claims                                                  | - `validateToken(jwt): AuthContext`<br>- `refreshToken(refreshToken): TokenPair`<br>- `fetchUserInfo(authContext): UserProfile`                                                                                                              |
| **Security & Privacy Facade**                    | - Central entry point for authorization & privacy decisions<br>- Performs coarse + fine-grained checks<br>- Applies role/policy rules, redaction, and minimization<br>- Emits structured audit events | - `authorize(authContext, action, resource): Decision`<br>- `filterResponse(authContext, data, viewType): RedactedData`<br>- `onSensitiveAction(authContext, action, resource, outcome)`<br>- `getSecurityContext(request): SecurityContext` |
| **PolicyDecisionService (PDP)**                  | - Evaluates RBAC/ABAC/PBAC authorization rules<br>- Supports endpoint-level and record-level policy checks<br>- Retrieves rules from the PolicyStore                                                  | - `decide(authContext, action, resourceAttrs): Decision`<br>- `explainDecision(id): PolicyTrace`                                                                                                                                             |
| **PolicyStore**                                  | - Stores all policies, roles, and retention configurations<br>- Supports versioning and staged activation                                                                                             | - `getActivePolicies(): PolicySet`<br>- `getPolicyById(policyId): Policy`<br>- `listPoliciesByScope(scope): List<Policy>`                                                                                                                    |
| **DataProtectionService (Crypto / KMS Adapter)** | - Encrypts/decrypts sensitive fields<br>- Uses external KMS / secret manager<br>- Handles key rotation and key alias management                                                                       | - `encrypt(field, plaintext): Ciphertext`<br>- `decrypt(field, ciphertext): Plaintext`<br>- `getKeyMetadata(field): KeyInfo`                                                                                                                 |
| **DataRetentionService**                         | - Executes scheduled deletion/anonymization based on retention policies<br>- Scans DB for expired or tagged data models<br>- Generates retention and policy-compliance reports                        | - `runRetentionCycle(policyId): RetentionReport`<br>- `tagRecord(recordId, policyTag)`<br>- `previewImpact(policyId): ImpactSummary`                                                                                                         |
| **AuditLogService**                              | - Stores structured security audit events<br>- Ensures immutable, append-only logging<br>- Correlates events using request IDs                                                                        | - `recordEvent(authContext, action, resource, outcome, metadata)`<br>- `searchEvents(filter): List<AuditEvent>`                                                                                                                              |
| **SecurityEventMonitor**                         | - Reads audit log stream for anomaly detection<br>- Issues alerts for suspicious behavior patterns                                                                                                    | - `evaluateHealth(): SecurityHealthStatus`<br>- `runDetectionRules(): List<Alert>`                                                                                                                                                           |
| **TokenRevocationStore (Optional)**              | - Maintains a list of invalidated tokens<br>- Used for forced logout or emergency access blocks                                                                                                       | - `isRevoked(tokenId): boolean`<br>- `revokeToken(tokenId, reason)`                                                                                                                                                                          |

---

## 5.2 Interactions with Existing Backend Modules

### API Gateway / Request Router

**Responsibilities**

* Validates JWT using `AuthAdapter`
* Builds `AuthContext` and attaches to request
* Performs coarse-grained authorization using the `Security & Privacy Facade`
* Forwards authorized requests to domain services with an attached `SecurityContext`

**Operational Flow (UC-1 / UC-2)**

1. Receive HTTPS request with token
2. `AuthAdapter.validateToken(jwt)` → produces `AuthContext`
3. `SecurityFacade.authorize(AuthContext, action, resource)` at API boundary
4. If allowed, forward to the domain service

---

## Step 6: Sketch Views and Record Design Decisions
In Iteration 3 we sketch **focused, security-centric views**:

- **6.1 Security Component View** – main security modules + how they relate.
- **6.2 UC-1 Security Flow (Sequence)** – how “Retrieve Exam Schedule” goes through security.
- **6.3 Security Deployment View** – where security runs in the cloud-native deployment.
  
---

### 6.1 Security Component View (Condensed)

**Purpose:** Show how the **API Gateway**, **Security & Privacy Subsystem**, **Domain Services**, and **Data/Infra** relate.

**Diagram 6.1 – Security, Privacy & Data Protection – Component View**  

<img width="2008" height="1045" alt="image" src="https://github.com/user-attachments/assets/b80a84ea-b1f9-48a8-bcac-cf34716c2657" />

- All domain services **depend on `Security & Privacy Facade`**, not directly on IdP or KMS.
- Policy decisions and storage are **centralized** in a `PolicyEngine` (PDP + PolicyStore).
- Crypto is **abstracted** behind `DataProtectionService` and a **KMS**.
- `DataRetentionService` and `AuditLogService` are **decoupled** from the request path and can evolve independently.

---

### 6.2 UC-1 Security Flow – Retrieve Exam Schedule

**Purpose:** Show how security is enforced *end-to-end* for **UC-1: Retrieve Exam Schedule**.

**Diagram 6.2 – UC-1: Retrieve Exam Schedule – Security Flow**  

<img width="1216" height="738" alt="image" src="https://github.com/user-attachments/assets/e9718177-416e-4c19-8e90-3d3d381221f2" />

- **Two tiers of authorization**:
  - At the **API Gateway** (coarse-grained, endpoint-level).
  - Within the **ExamService** (fine-grained, resource-level).
- Decryption via `DataProtectionService` occurs **only after** a successful authorization decision.
- `filterResponse` in the Security & Privacy Facade enforces **data minimization** before results leave the backend.
- All sensitive actions (viewing schedules, posting announcements, etc.) are recorded through `AuditLogService` for **accountability and traceability**.

---

### 6.3 Security Deployment View (Condensed)

**Purpose:** Show where security components live in the **cloud-native deployment**.

**Diagram 6.3 – Security & Privacy – Deployment View (Condensed)**  

<img width="1679" height="578" alt="image" src="https://github.com/user-attachments/assets/dae5b039-992a-4e55-954e-a1fc72c3cd44" />

- Security logic (`Security & Privacy` components) is **co-located** with the backend in the same Kubernetes cluster, but modeled as a **separate container** for clarity and modifiability.
- **DataRetentionService** and **SecurityEventMonitor** run as **background jobs**, so they **do not impact** the latency of UC-1 / UC-2.
- **AIDAP DB**, **Audit Log Store**, and **KMS** are shared infrastructure services with:
  - **TLS in transit** and  
  - **encryption at rest**,  
  satisfying **CON-5 (cloud-native containerized)** and **CON-6 (retention & encryption standards)**.
- This deployment supports **horizontal scaling** of API Gateway, backend, and security components, contributing to **QA-2 (availability & scalability)** while maintaining **QA-4 (privacy & security)**.

---

## Step 7: Analysis of Design

| Driver                               | Coverage         | Explanation                                                                                                                                                                                              |
| ------------------------------------ | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **UC-1: Retrieve Exam Schedule**     | Full             | Authorization occurs at API + service level; response filtering ensures privacy; encrypted fields are only decrypted after approval; audit logs capture access.                                          |
| **UC-2: Post Course Announcement**   | Full             | Instructor roles validated by policies; unauthorized posts blocked; every announcement event logged for traceability.                                                                                    |
| **UC-4: Deploy System Update**       | Partial / Strong | Security policy configuration, encryption keys, and retention rules are externalized and persist across deployments. CI/CD specifics remain future work.                                                 |
| **QA-4: Privacy & Security**         | Strong           | Central Security Facade, PDP, field-level encryption, structured auditing, and retention engine provide robust privacy-by-design coverage. Remaining work includes detailed incident-response workflows. |
| **QA-2: Availability & Scalability** | Partial / Strong | Security components are horizontally scalable (AuthAdapter, PDP replicas, audit log pipeline). IdP, KMS, and DB assumed to have HA configurations but not detailed here.                                 |
| **QA-1: Performance (<2s)**          | Acceptable       | Token validation is lightweight; PDP evaluations are fast; crypto is applied selectively; overall overhead remains small enough to meet UC-1/UC-2 timing constraints.                                    |
| **CON-1: SSO**                       | Full             | All authentication flows use the institution’s identity provider (OIDC/SAML).                                                                                                                            |
| **CON-2: REST/GraphQL**              | Full             | Security and authorization applied cleanly at all API boundaries.                                                                                                                                        |
| **CON-5: Cloud-Native Deployment**   | Full             | All security modules run as scalable containerized services, matching cloud-native constraints.                                                                                                          |
| **CON-6: Retention & Encryption**    | Partial / Strong | Transparent storage encryption + field-level encryption + automated retention policies implemented. Further institution-specific compliance rules required.                                              |

---
