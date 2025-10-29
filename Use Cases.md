# Use Cases

| Use Case | Description | Associated Requirement ID |
|----------|-------------|----------------------------|
| **UC-1: Retrieve Exam Schedule**|A student interacts with AIDAP (via mobile app, website chat, or voice) to ask about upcoming exams. The system returns the nearest upcoming exam with date, time, and course details. | R1, R3, R4, R5, R6, RS1, RS2, RS8, RS9, RS10 |
| **UC-2: Post Course Announcement**|A lecturer requests AIDAP to post an announcement to a selected course. AIDAP validates instructor permissions, connects to the LMS API, and creates the announcement. The lecturer receives confirmation and may schedule automated reminders. | R1, R3, R4, R5, R6, R8, RL1, RL2, RL8 |
| **UC-3: View Class Analytics** | A lecturer asks AIDAP for a summary of student performance like average grade, attendance, engagement. AIDAP retrieves and aggregates data from the LMS to present analytics. | R1, R3, R5, RL3, RL6, RL7 |
| **UC-4: Deploy System Update** | A system maintainer deploys a software update using the automated CI/CD pipeline. AIDAP applies the update with zero downtime and logs deployment status. | R7, RM1, RM2, RM7 |
**UC-5: Monitor System Usage Analytics** | An administrator queries system usage statistics such as daily active users, most common query types, and peak hours. AIDAP retrieves analytics and displays a summarized report. | R1, R6, RA4, R7 |

<img width="466" height="728" alt="image" src="https://github.com/user-attachments/assets/42db3157-bd47-4a3e-a5f7-1eadab7d0eb5" />
