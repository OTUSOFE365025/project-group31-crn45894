## Step 1: Review Inputs

**Design Purpose:**

AIDAP is a greenfield system intended to provide AI-driven academic assistance to students and faculty.  
 The purpose of the architectural design is to produce a sufficiently detailed structure that supports:

- conversational query handling,

- secure integration with university systems,

- high availability during peak periods,

- modifiable AI components, and

- compliance with privacy policies.

**Primary functional requirements:**

These are the main use cases relevant to the architectural design.

|  Use Case  | Description |
| ----- | ----- |
| UC-1: Retrieve Exam Schedule | A student interacts with AIDAP via mobile app, web chat, or voice interface to ask about upcoming exams. AIDAP retrieves and returns the nearest exam with full details. |
| UC-2: Post Course Announcement | A lecturer requests AIDAP to post an announcement to a course. AIDAP validates permissions, connects to the LMS API, posts the announcement, and confirms completion. |
| UC-4: Deploy System Update | A system maintainer deploys a system update through CI/CD. AIDAP applies the update with zero downtime and logs deployment results. |

## 

## Step 2: Establish Iteration goal by selecting drivers

### Goal:

Establish the overall system structure for AIDAP that:

* Supports the core student and instructor interaction flows (UC-1, UC-2)

* Allows updates and new AI models to be deployed with zero downtime (UC-4)

* Ensures secure access through SSO and strict privacy compliance

* Enables cloud-native deployment with scalable components to handle peak usage

* Keeps the AI model logic, data access, and external system integrations modular to support future expansion

# Drivers Selected for Iteration 1

### Functional drivers

* UC-1: Retrieve Exam Schedule  
   Core student-facing interaction; requires fast response times and reliable data access.

* UC-2: Post Course Announcement  
   Requires LMS integration, authentication, authorization, and consistent API communication.

* UC-4: Deploy System Update  
   High architectural impact due to modifiability and the requirement for zero downtime.

### Quality attribute drivers

* QA-2: Availability  
   Critical due to expected high traffic (e.g., registration week). Influences decisions on redundancy, auto-scaling, load balancing, and service decomposition.

* QA-3: Modifiability  
   High business and technical priority. Strong influence on modular architecture, plug-and-play AI models, CI/CD structure, and service boundaries.

* QA-1: Performance  
   The requirement to answer queries within 2 seconds shapes caching, API design, and system interaction patterns.

* QA-4: Privacy  
   Mandatory compliance requirement influencing data logging, storage, access control, encryption, and isolation of sensitive components.

### Constraint drivers

* CON-1: University SSO authentication  
   Determines identity management integration and influences gateway and backend design.

* CON-2: REST/GraphQL communication  
   Determines communication style and integration patterns with external systems.

* CON-5: Cloud-native deployment  
   Influences containerization, orchestration decisions, and service granularity.

* CON-6: Data retention & encryption standards  
   Drives how data is stored, encrypted, logged, and managed across components.

## 

# 

# 

## 

## Step 3: Choose Elements of the System to Decompose

At the start of Iteration 1, the entire AIDAP system is treated as a single black-box element:

### Element:

AIDAP System (root system)

Because the goal of this iteration is to establish the overall system structure, this root element is selected for decomposition.

## Step 4: Choosing Design Concepts

| Design Decisions | Rationale |
| ----- | ----- |
| Logically structure the system using a Three-Layered Architecture | A layered architectureseparates presentation, business logic, and data access responsibilities. This supports QA-3 Modifiability, since new AI models, external APIs, and UI channels can be changed independently. It supports QA-5 Usability by keeping interface logic isolated and easier to evolve. It supports CON-2, since external system communication is restricted to well-defined API boundaries. Privacy concerns (QA-4, CON-3, CON-4) are addressed by restricting sensitive operations to the application and data layers. |
| Physically structure the deployment using a Cloud-Native Three-Tier Deployment Pattern. | A three-tier deployment supports QA-2 Availability through load-balanced application servers and scalable infrastructure. It satisfies CON-5, which requires cloud-native deployment, and CON-6, which mandates controlled and encrypted data storage. The separation of tiers also simplifies future scalability decisions (CRN-3). |
| Use a Service Application Reference Architecture for AIDAP’s backend service layer. | Service-oriented applications expose functionality via APIs consumed by clients. This directly supports CON-2, which requires standardized REST/GraphQL API communication. It aligns with QA-1 Performance by enabling optimized, scalable API endpoints. It also supports QA-4 Privacy, allowing centralized enforcement of authentication and authorization. |
| Adopt a Modular AI Model Management Subsystem within the Application Layer. | This subsystem allows AI models to be added, replaced, or removed without affecting other system components, directly satisfying QA-3 Modifiability. It supports UC-4 (system updates with zero downtime). It also addresses CRN-1 by making technology choices flexible. |
| Implement SSO-based Authentication and Role-Based Access Control (RBAC) at the Presentation–Application boundary. | SSO satisfies CON-1 by using the university’s authentication system. RBAC ensures that private student data is only accessed by the authenticated student (CON-4, QA-4 Privacy). Combining both mechanisms isolates authentication logic and simplifies maintainability (CRN-2, CRN-4). |
| Logically structure the client part of the system using the Rich Client Application reference architecture | The Rich Client Application reference architecture supports the development of applications that run directly on the user’s device and provide an interactive, responsive interface. This approach is suitable for AIDAP because it allows the client to deliver smooth conversational interactions and fast response times, helping meet QA-1 Performance during typical student queries. The RCA model also supports adaptive and intuitive interfaces, which contributes to fulfilling QA-5 Usability, even though usability is not the main driver behind this decision. In addition, by keeping sensitive operations local to the authenticated session, this architecture assists in satisfying QA-4 Privacy and ensuring that student data is only accessible to authorized users in accordance with CON-4. Although rich client applications do not rely solely on browser execution, they can still interact seamlessly with backend services through standardized REST/GraphQL APIs, ensuring compliance with CON-2. |

## Step 5: Instantiate Architectural Elements, Allocate Responsibilities,and Define Interfaces

| Design Decisions and Location  | Rationale |
| :---- | ----- |
| Introduce a centralized Data Access module inside the Rich Client | This module abstracts all communication with the AIDAP server and hides protocol details from the rest of the client. It improves modifiability (QA-2) by localizing all networking concerns and supports UC-1, UC-4, and UC-7, which depend on reliable fetching and submitting of data. |
| Separate the UI into View and Controller components | Views focus only on presentation while Controllers manage user interactions. This separation makes UI updates easier and improves usability (QA-3) and modifiability (QA-2). It directly supports UC-1 and UC-5, where clear and consistent user interfaces are needed. |
| Instantiate a Client-Side Business Logic component | This component performs validation, data preparation, and rule checks before sending information to the server. It reduces unnecessary network calls and improves performance (QA-4). It supports UC-2 and UC-3, which require repeated user actions handled efficiently. |
| Define a Synchronization Interface between the Rich Client and server services | A dedicated interface formalizes request/response patterns, error codes, and retry behavior. It improves reliability (QA-1), especially under intermittent network conditions, and is essential for UC-7 where data submission must be dependable. |
| Use a standardized Representation Model for all data exchanged with the server | Representing data using structured objects or DTOs decouples the UI and business logic from server-specific formats. This supports QA-2 (modifiability) and ensures consistent handling of data across all use cases |
| Remove persistent local storage from the Rich Client | Data is not stored locally; the server remains the authoritative data source. This eliminates synchronization issues and improves reliability (QA-1). This decision supports UC-7, where the integrity and freshness of information are very important |

