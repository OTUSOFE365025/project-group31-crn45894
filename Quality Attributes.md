# Quality Attributes
| ID | Quality Attribute Scenario | Associated Use Case |
|----|-----------------------------|---------------------|
| **QA-1 (Performance)**|When a student queries their next exam, AIDAP shall respond within 2 seconds under normal load conditions, ensuring seamless conversational interaction. *(RS10)* | UC-1 |
| **QA-2 (Availability)**|During high-traffic periods (e.g., registration week), AIDAP remains accessible with ≥99.5% uptime per month, employing auto-scaling and failover strategies. *(RS11, RA6)* | UC-1, UC-2 |
| **QA-3 (Modifiability)**|A new AI model is introduced to the system to support a language that the system did not have without affecting the other system components and without downtime. *(RS4, RM1, RM5)* | UC-4 |
| **QA-4 (Privacy)**|When handling user queries, AIDAP ensures that all logs and stored responses comply with privacy regulations. *(RS8, RA5)* | UC-1 |
| **QA-5 (Usability)**|When a new user interacts with AIDAP, the interface gudes them intuitively using conversational cues an context-aware responses. *(RS12)* | UC-1 |
