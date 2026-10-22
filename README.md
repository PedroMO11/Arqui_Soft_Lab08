# Arqui_Soft_Lab08

## Spotty — Automated Grading Platform on AWS Spot

**Case Study #11 · Software Architecture · UTEC 2026-II**
**Team:** _TBD_ · **Members:** _TBD_

Today the Software Architecture course grades its students' code submissions on a local server (a laptop), with one sandbox per student and workers (minions) that grade code and diagrams. The goal is to extend the system to every course, using only AWS Spot instances for sandboxes and workers.

---

## 1. System actors

| Actor | Type | Channel | Description |
|---|---|---|---|
| **Student** | Primary user | Mobile app | Submits their work (code, diagrams, models) and checks the status and grade of each submission. |
| **Professor** | Primary user | Desktop web | Creates assignments for their course by providing the rubric and how the student's submission should be evaluated, tracks grading progress and publishes grades no later than 2 days after the deadline. **Needs no technical knowledge and never configures infrastructure.** |
| **IT Administrator** | Support user | Desktop web / dashboards | Prepares the base image for each course and defines resources per course type (CPU, memory, GPU, max time). Manages the budget, Spot capacity and platform health. |
| **Worker / Minion** | Internal system | — | Runs the student's untrusted code inside an isolated sandbox and produces the evaluation result. |
| **AWS EC2 Spot** | External system | AWS API | Provides the worker instances. It can reclaim them with a 2-minute notice or run out of capacity for the requested type (especially GPU). |
| **University academic system** | External system | API | Receives the grades published by the professor. |
| **Push notification service** | External system | FCM / APNs | Delivers status notifications to the student's mobile app. |
| **LLM provider** | External system | API | Turns the professor's rubric into an executable evaluation and helps workers grade open-ended criteria (diagrams, code quality). |

---

## 2. Functional requirements

| ID | Actor | Requirement |
|---|---|---|
| **FR-01** | Student | The system must let the student upload a submission (code, diagrams or notebooks/models) for an open assignment, rejecting it if the deadline has passed or the maximum number of attempts was exceeded. |
| **FR-02** | Student | The system must let the student check from the mobile app the status of each submission (`QUEUED`, `RUNNING`, `RETRYING`, `GRADED`, `ERROR`), together with its grade and feedback, and notify every change through a push notification. |
| **FR-03** | Professor | The system must let the professor create an assignment through a guided form, providing only the deadline, the **rubric** (criteria and points) and **how to evaluate** the submission (for example, test cases with expected input and output, a reference solution or a checklist for diagrams). An LLM turns the rubric into an executable evaluation that the professor approves through a preview. Resources and the base image are assigned automatically according to the course type. |
| **FR-04** | System | The system must run and grade each submission in an isolated sandbox with the course's base image (with tests and, for open-ended criteria, the help of an LLM), and retry it automatically if AWS reclaims the instance, without the student having to resubmit. |
| **FR-05** | Professor | The system must show the professor the grading progress of each assignment (graded, queued, failed, estimated time) and let them publish the grades to the academic system. |

---

## 3. Non-functional requirements

| ID | Attribute | Requirement | Metric |
|---|---|---|---|
| **NFR-01** | Performance | Each submission must be graded within the P99 of its course type: **Programming ≤ 1 min**, **Architecture ≤ 10 min**, **AI ≤ 40 min** of execution. | P99 from submission to result, per course type |
| **NFR-02** | Scalability | The system must absorb the submission peak of the **last 30 minutes before the deadline** by scaling workers based on the queue, without large courses degrading small ones. | P99 per course during the deadline window |
| **NFR-03** | Reliability | No submission may be lost because of a Spot interruption (2-minute notice) or lack of capacity, and 100 % of submissions must be graded within **24 hours** of the deadline, leaving a buffer before the professor's **48-hour** limit. | `% jobs graded / jobs received = 100 %` · `% submissions graded within 24 h = 100 %` |
| **NFR-04** | Security | The student's code is **untrusted**: it must run without access to the network, credentials or other sandboxes, with hard limits on CPU, **memory** and time, so that inefficient code only affects its own execution. | 100 % of the malicious-code test suite blocked |
| **NFR-05** | Cost | Workers and sandboxes must run **only on AWS Spot instances**, using GPU only for the executions that need it. | % of worker compute on Spot = 100 % |

---

## 4. Constraints and assumptions

**Constraints from the case statement**

- Only **Spot** instances for sandboxes and workers. AWS can reclaim them with a **2-minute notice** or run out of capacity, and GPU instances are the scarcest.
- Each course has its own **base image of 3 to 5 GB**.
- Usage is **not constant**: almost all submissions arrive in the last 30 minutes before the deadline.
- Some students submit inefficient or memory-hungry code.
- Professors have at most **2 days** to deliver grades.

**Assumptions (to be confirmed with the professor)**

- ~10,000 students and ~300 sections with deliverables. Worst case: several assignments with the same deadline.
- The 2-day window starts at the assignment deadline.
- The Spot constraint only applies to workers and sandboxes; the API, database and queues can use managed services.
- The sandbox needs no internet access: dependencies are included in the course's base image.
- Deadlines are known in advance.
- The professor never configures infrastructure: IT prepares each course's base image and the platform assigns resources by course type (Programming, Architecture or AI).
