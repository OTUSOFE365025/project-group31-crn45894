# SOFE3650F25 Project Repository Template
The objective of this project is to demonstrate a methodological set of steps in the design of a software
architecture for a set of requirements listed in Appendix A. The expected design approach to take is
the Attribute Driven Design (ADD).

## Deliverables
The expectation is to submit a set of artifacts that demonstrates the ADD steps as applied to the design and implementation of an AI-Powered Digital Assistant Platform.

## Phase 1
Use cases, quality attributes and constraint requirements of the project need to be defined based on the requirements that are provided below.
Expected artifacts are:
- Use Case models
- Quality Attributes for the application
- Architectural Concerns
- Business Drivers

## Phase 2

## Appendix A: Project Requirements
AI-Powered Digital Assistant Platform (AIDAP)

The **AI-Powered Digital Assistant Platform (AIDAP)** provides a conversational interface for students, faculty, and administrators to interact with institutional data such as course schedules, deadlines, announcements, and academic analytics. The assistant integrates with external university systems (LMS, registration, calendars, and mail) and uses AI to deliver contextual answers.

---

## Stakeholders

| Symbol | Stakeholder | Description |
|---------|--------------|--------------|
| **S** | Students | End users who query academic and campus information. |
| **L** | Lecturers | Provide course-related content and respond to academic queries. |
| **A** | Administrators | Maintain institutional data, integrations, and policies. |
| **M** | System Maintainer | Responsible for deployment, monitoring, and upgrades. |
| **D** | Data Source Systems | External systems such as LMS, Registration, Calendar, and Email servers. |

---

## General Requirements

| ID | Requirement |
|----|--------------|
| **R1** | The system shall provide conversational access to institutional data and services. |
| **R2** | The system shall store historical interactions for personalization. |
| **R3** | The system shall integrate with existing data sources (registration, LMS, calendars). |
| **R4** | The system shall support both text and voice interaction modes. |
| **R5** | The system shall use AI models to interpret natural-language queries. |
| **R6** | The system shall generate responses using both stored knowledge and live data. |
| **R7** | The system shall be deployable as a cloud-native, scalable service. |
| **R8** | The system shall protect user data and comply with institutional privacy policies. |

---

## Requirements of Students

| ID | Requirement |
|----|--------------|
| **RS1** | The system shall allow students to ask academic or administrative questions (e.g., “When is my next exam?”). |
| **RS2** | The system shall notify students of deadlines, schedule changes, and announcements. |
| **RS3** | The system shall allow students to access personalized dashboards summarizing upcoming events and performance indicators. |
| **RS4** | The system shall support multi-language queries and responses. |
| **RS5** | The system shall learn from previous conversations to improve response relevance. |
| **RS6** | The system shall allow students to change preferences for notifications and language. |
| **RS7** | The system shall provide secure authentication through the institution’s single sign-on (SSO). |
| **RS8** | The system shall ensure that student-specific data are visible only to the authenticated user. |
| **RS9** | The system shall be accessible on mobile, web, and voice-assistant devices. |
| **RS10** | The system shall respond to queries within 2 seconds on average under normal load. |
| **RS11** | The system shall remain available 99.5% of the time per month. |
| **RS12** | The system shall have an intuitive UI consistent with conversational design best practices. |
| **RS13** | The system shall allow export of calendar events to personal calendars. |
| **RS14** | The system shall support offline cache of recent responses for limited connectivity. |

---

## Requirements of Lecturers

| ID | Requirement |
|----|--------------|
| **RL1** | The system shall allow lecturers to publish or update course materials accessible to students through the assistant. |
| **RL2** | The system shall enable lecturers to post announcements via conversational commands. |
| **RL3** | The system shall allow lecturers to view summarized class analytics (grades, attendance, engagement). |
| **RL4** | The system shall enable lecturers to schedule automated reminders (e.g., assignment deadlines). |
| **RL5** | The system shall allow lecturers to manage access rights for teaching assistants. |
| **RL6** | The system shall allow lecturers to query the assistant for aggregated statistics (e.g., average GPA by course). |
| **RL7** | The system shall notify lecturers of system-detected anomalies (e.g., sudden drop in participation). |
| **RL8** | The system shall ensure that only authorized lecturers can modify course data. |

---

## Requirements of Administration

| ID | Requirement |
|----|--------------|
| **RA1** | The system shall allow administrators to manage institutional integrations (LMS, registration, calendars). |
| **RA2** | The system shall allow administrators to define global policies (data retention, response logging). |
| **RA3** | The system shall allow administrators to broadcast campus-wide announcements via the assistant. |
| **RA4** | The system shall allow administrators to monitor system usage and generate analytics reports. |
| **RA5** | The system shall ensure compliance with institutional security and privacy regulations. |
| **RA6** | The system shall provide high availability with automatic fail-over and backup recovery. |
| **RA7** | The system shall support scalability to handle up to 5,000 concurrent users. |

---

## Requirements of System Maintainer

| ID | Requirement |
|----|--------------|
| **RM1** | The system shall allow maintainers to deploy updates with zero downtime using continuous deployment pipelines. |
| **RM2** | The system shall provide monitoring dashboards (health, latency, errors). |
| **RM3** | The system shall support configuration of AI model versions and API keys. |
| **RM4** | The system shall log performance metrics for model accuracy and latency. |
| **RM5** | The system shall be easily extensible to integrate new AI services or external data sources. |
| **RM6** | The system shall allow secure backup and restore of user and configuration data. |
| **RM7** | The system shall support role-based access for maintenance operations. |

---

## Requirements of Data Source Systems

| ID | Requirement |
|----|--------------|
| **RD1** | The system shall synchronize data with connected university systems at configurable intervals. |
| **RD2** | The system shall use standard APIs (REST or GraphQL) for interoperability. |
| **RD3** | The system shall handle failures in data source availability gracefully (retry and recovery). |
| **RD4** | The system shall maintain data integrity and consistency across systems. |
