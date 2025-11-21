# AIDAP Architectural Design – ADD Iteration 2

## Step 1: Review Inputs

### Design Context

From Iteration 1, AIDAP has an overall architecture with:

- Rich client frontend (web / mobile / voice)
- Server-side layers:
  - Services
  - Business Logic
  - Data
  - Cross-cutting
- Cloud-native three-tier deployment (client, application server, database)

This iteration refines the **server-side application** to better satisfy performance, availability, and privacy requirements.

### Relevant Functional Requirements / Use Cases

- **UC-1: Retrieve Exam Schedule**  
  A student asks AIDAP for upcoming exams; backend must fetch schedule data and return within 2 seconds.

- **UC-2: Post Course Announcement**  
  A lecturer issues a conversational command; backend validates permissions and posts to the LMS via standard APIs.

- **UC-4: Deploy System Update**  
  A maintainer deploys a new version of the backend / AI without downtime and with safe rollback.

### Quality Attribute Drivers

- **QA-1 Performance**  
  Respond to student queries within **2 seconds** on average under normal load.

- **QA-2 Availability & Scalability**  
  At least **99.5% availability per month** and support for up to **5,000 concurrent users**, with automatic failover and backup.

- **QA-4 Privacy & Security**  
  Protect user data, enforce access control and retention, secure backup/restore, and comply with institutional policies.

### Constraints

- **CON-1** – Use **university SSO** for authentication.
- **CON-2** – External systems use **standard REST/GraphQL** APIs.
- **CON-5** – **Cloud-native deployment**: containerized and orchestrated.
- **CON-6** – Enforce **data retention and encryption standards**.

---

## Step 2: Establish Iteration Goal by Selecting Drivers

### Iteration 2 Goal

Refine the **AIDAP Server-Side Backend** (Services + Business Logic + Data + Cross-Cutting) so that:

- It can serve **UC-1** and **UC-2** within the **2-second response time** at normal load (QA-1).
- It can **scale and remain available** at **5,000 concurrent users with 99.5% uptime** (QA-2).
- It enforces **strict privacy and security** for student and lecturer data (QA-4, CON-1, CON-6).
- It remains **modifiably deployable**, supporting UC-4 and zero-downtime updates.

### Drivers for This Iteration

**Functional drivers**

- UC-1: Retrieve Exam Schedule  
- UC-2: Post Course Announcement  
- UC-4: Deploy System Update  

**Quality attribute drivers**

- QA-1 Performance  
- QA-2 Availability / Scalability  
- QA-4 Privacy / Security  

**Constraint drivers**

- CON-1 SSO  
- CON-2 REST/GraphQL integration  
- CON-5 Cloud-native  
- CON-6 Data retention & encryption  

---

## Step 3: Choose Elements of the System to Decompose

From Iteration 1, the server side is organized into:

- Services
- Business Logic
- Data
- Cross-Cutting

For Iteration 2 we select:

> **Element to Decompose:**  
> **AIDAP Server-Side Backend** (encompassing the Services, Business Logic, Data, and Cross-Cutting layers running in the Application Server Node).

---

## Step 4: Choose Design Concepts

1. **Modular Monolith Backend with Internal Service Modules**
2. **Ports-and-Adapters (Hexagonal) for Integrations**
3. **Read-Optimized Query Path with Caching**
4. **Asynchronous Processing for Non-Critical Tasks**
5. **Centralized Security & Privacy Facade**
6. **Stateless Backend Instances with Blue-Green / Rolling Deployments**

---

## Step 5: Instantiate Architectural Elements, Allocate Responsibilities, and Define Interfaces

### 5.1 Core Service Modules

### 5.1 Core Service Modules

| Module | Responsibilities | Interfaces |
|--------|------------------|------------|
| **API Gateway / Request Router** | - Terminates HTTPS connections<br>- Parses REST/GraphQL requests<br>- Routes requests to internal modules (`ExamService`, `AnnouncementService`, `AIOrchestrationService`)<br>- Calls `AuthService` + `SecurityPrivacyFacade` for validation | - `GET /api/exams/next`<br>- `GET /api/exams/schedule`<br>- `POST /api/courses/{courseId}/announcements` |
| **AuthService** | - Integrates with university SSO<br>- Validates tokens and retrieves user roles | - `validateToken(token): AuthContext`<br>- `getUserRoles(userId): RoleSet` |
| **Security & Privacy Facade** | - Centralizes authorization logic<br>- Performs redaction and data minimization<br>- Records audit logs for sensitive operations | - `checkAccess(authContext, action, resource): Decision`<br>- `filterResponse(authContext, rawData): RedactedData`<br>- `recordAuditEvent(authContext, action, resource)` |
| **ExamService** | - Handles UC-1 requests for upcoming exams<br>- Uses read-optimized repository + cache<br>- Ensures <2 sec response under normal load | - `getNextExamForStudent(studentId): ExamInfo`<br>- `getExamSchedule(studentId, timeRange): List<ExamInfo>` |
| **AnnouncementService** | - Handles UC-2<br>- Validates instructor permissions<br|- Uses `LMSAdapter` to post announcements | - `postAnnouncement(courseId, lecturerId, content): Result` |
| **AIOrchestrationService** | - Interprets natural-language queries<br>- Selects model versions and interacts with AI providers<br>- Generates final responses | - `interpretQuery(authContext, inputText): Intent+Params`<br>- `generateResponse(context, intentResult): Answer`<br>- `setActiveModel(modelId, version)` |

---

### 5.2 Integration & Data Modules

- `LMSAdapter`, `SISAdapter`, `CalendarAdapter`, `EmailAdapter`
- `ExamReadRepository`, `ExamWriteRepository`
- `ExamCache`, `CourseCache`
- `Monitoring & Telemetry Module`

---

## Step 6: Sketch Views and Record Design Decisions

### 6.1 Backend Module View

See:

### 6.2 Deployment View

See:

---

## Step 7: Analysis of Design

| Driver | Coverage | Explanation |
|--------|----------|-------------|
| UC-1 | Full | Optimized read path through cache + DB |
| UC-2 | Full | LMSAdapter + SecurityFacade handle permissions & integration |
| UC-4 | Partial | Supports blue-green but CI/CD details are future work |
| QA-1 | Partial/Strong | Caching, async tasks, horizontal scaling |
| QA-2 | Partial | Clustered backend, but DB/Cache failover still needed |
| QA-4 | Partial | Privacy Facade + audit logging; retention/crypto TBD |
| CON-1 | Full | AuthService |
| CON-2 | Full | REST/GraphQL boundaries |
| CON-5 | Full | Cloud-native deployment |
| CON-6 | Partial | Secure DB/Cache; policy details pending |

---
