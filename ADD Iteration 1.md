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

## Step 6: Sketch Views and Record Design Decisions

Layer View:




| Element | Responsibility |
| ----- | ----- |
| Presentation Client Side  | This layer contains modules that manage the user interface, handle student interactions, and support the conversational flow of AIDAP. These modules are responsible for rendering chat screens, dashboards, and navigation, supporting QA-5 (Usability). |
| Business Logic  | This layer contains modules that execute lightweight business logic on the client side, including preprocessing user messages, maintaining local session context, and performing validation before contacting the server. |
| Data  | This layer contains modules responsible for communication with the server, including formatting outbound requests, handling incoming responses, and maintaining connection reliability (supports QA-1 and QA-2). |
| Cross-Cutting  | This collection of modules covers security enforcement, local logging, temporary caching, and input sanitization on the client side. These elements support QA-4 (Privacy) and CON-1 (SSO). |
| UI Modules  | These modules render the student-facing interface, display conversational responses, and receive student queries. They support intuitive interaction and contribute directly to QA-5 (Usability). |
| UI Process Modules  | These modules guide the control flow of all user interactions, including managing chat sessions, handling conversational state, and coordinating UI transitions. |
| Client Business Modules  | These modules carry out operations that can be executed locally, such as basic academic rule checks, autocomplete, message preprocessing, and storing user context. They may also adapt server-side workflows for client display. |
| Business Entities  | These client-side models represent simplified versions of domain entities used to structure data displayed to the user. They are less detailed than server-side equivalents. |
| Communication Modules  | These modules consume the services exposed by the server-side API. They manage REST/GraphQL requests, message serialization, and error handling. They support CON-2 (API-only communication). |

# 

|  |  |
| ----- | :---- |
| Services Server Side  | This layer contains modules that expose services to the client application, including endpoints for academic queries, authentication, scheduling, and data retrieval. Supports CON-2 and QA-2. |
| Business Logic  | This layer contains modules that implement AIDAP’s core logic, such as academic advising workflows, requirement validation, AI reasoning flows, and program-specific rule processing. Supports QA-1, QA-3. |
| Data  | This layer contains modules responsible for data retrieval, transformation, and persistence. It connects to university systems, academic databases, and historical logs. Supports QA-4 and CON-6. |
| Cross-Cutting  | These modules include security enforcement, logging, monitoring, fault handling, and operational management. They ensure system reliability (QA-2) and privacy (QA-4). |
| Service Interfaces  | These modules define the REST/GraphQL API endpoints that the client consumes. They include request validation, schema definitions, and contract enforcement. |
| Business Modules  | These modules perform complex academic processing such as calculating program completion, validating prerequisites, analyzing academic history, and preparing AI-ready contextual data. |
| Business Entities  | These entities represent the full domain model. They are used by business logic modules to evaluate academic constraints. |
| DB Access Module  | This module is responsible for persistence of business entities and communication with university databases. It performs mapping, enforces data integrity, and shields upper layers from database structure. Supports CON-6. |
| Service Agents  | These modules connect to external systems such as the LMS, SIS, grade systems, and other academic services, supporting UC-1, UC-2, and QA-1. |

Deployment Diagram:  


| Node | Component | Responsibilities |
| ----- | ----- | ----- |
| Client Node | AIDAP Web Client | Provides the user interface for interacting with the system, sends requests to the server, displays processed data and reports. |
|  | Dashboard UI | Presents schedules, tasks, deadlines, analytics summaries, and recommendations directly to the user. Handles UI rendering and user interactions. |
|  | Client Analytics Module | Collects client-side usage data (page views, actions, preferences) to enhance insights and improve recommendations. |
|  | Local Cache Store | Stores temporary session data such as recently viewed tasks, schedules, or pre-fetched analytics to improve performance and reduce server calls. |
| Application Server Node | AIDAP Application | Central application logic responsible for processing client requests, coordinating between modules, and managing workflows. |
|  | Analytics Engine | Processes academic data, user patterns, and historical usage to generate insights and computed metrics used in reports. |
|  | Rule Evaluation | Applies predefined decision rules to produce recommendations or identify conflicts. |
|  | Reporting Controller | Generates structured reports for deadlines, usage analytics, academic workloads, and system insights. Formats results for client consumption. |
|  | API Endpoints | Provide REST endpoints for all system interactions (retrieving tasks, submitting updates, fetching analytics, generating reports). |
| Database Server Node | AIDAP Database | Stores all persistent data (users, tasks, schedules, analytics outputs, rules). Maintains integrity and supports transactions. |
|  | Report Storage | Stores previously generated reports, summaries, and analytics results for retrieval and historical comparison. |
|  | Analytics Data Tables | Hold processed metrics, usage logs, computed indicators, and AI-assisted data used by the analytics engine. |
| External Nodes | Time Server | Provides accurate, synchronized timestamps for scheduling, reminders, deadline validation, and logging. |
|  | Notification/Email Server | Sends system-generated emails and notifications such as reminders, alerts, scheduling warnings, and report delivery. |

## 

## 

## 

## 

## Step 7: Analysis of Design

| Not Addressed | Partially Addressed | Completely Addressed | Design Decisions Made During the Iteration |
| ----- | ----- | ----- | ----- |
|  |  |  UC-1 | The selected layered and service-oriented reference architectures establish the necessary modules that will support retrieving exam schedules. |
|  |  | UC-2 | The selected architectural structure establishes the components that will support LMS communication and announcement posting, including service agents and server-side business logic. |
|  | UC-4 |  | Cloud-native deployment and modular AI components support zero-downtime updates. However, CI/CD tooling and update orchestration have not yet been defined. |
|  | QA-1 |  | No specific performance-focused decisions have been made yet, as it is still necessary to identify the exact components involved in the performance-critical paths. |
|  |  | QA-2 | Use of a cloud-native three-tier deployment pattern and scalable backend servers improves availability. Auto-scaling and redundancy are implied but not yet fully detailed. |
|  |  | QA-3 | The modular AI subsystem and layered architecture identify elements that may be independently updated or replaced. These provide the foundation for modifiability. |
|  | QA-4 |  | No specific decisions fully address privacy beyond the initial introduction of SSO and RBAC. Detailed encryption, logging limits, and data-retention enforcement remain to be designed. |
|  | CON-1 |  | SSO integration at the presentation–application boundary allows the system to authenticate all clients through the university identity provider. Decisions regarding concurrency control have not yet been made. |
|  |  | CON-2 | The use of REST/GraphQL-based service architecture supports required communication constraints. The details of request schemas and endpoint definitions will be completed later. |
|  |  | CON-5 | Physically structuring the system using the three-tier cloud-native deployment pattern satisfies this constraint and prepares the system for scalable containerized deployment. |
| CON-6 |  |  | No specific decisions regarding data encryption standards or retention policies have yet been made at this stage. |
|  |  | CRN-1 | Selection of the reference architectures and the cloud-native deployment pattern has been completed. |
|  |  | CRN-2 | Technologies selected so far consider developer familiarity (REST APIs, containerized deployment, SSO). Additional technologies (e.g., model loading, AI versioning tools, university APIs) still need selection. |
| CRN-3 |  |  | No relevant decisions have been made to address long-term changes in cloud infrastructure or vendor migration strategies. |



