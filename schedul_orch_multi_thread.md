# 🧩 Project Outline: **Async Task Orchestrator**

## 📘 1. Project Overview

The **Async Task Orchestrator** is a Spring Boot–based backend service that manages and executes background tasks asynchronously.
It allows clients to **submit jobs**, such as file processing, report generation, or notifications, which are executed in **parallel threads** with **progress tracking**, **retry mechanisms**, and **status monitoring**.

This project demonstrates real-world **multithreading and concurrency** concepts using **Java and Spring Boot**.

---

## 🎯 2. Objectives

* Learn and implement **multithreading concepts** in a Spring Boot application.
* Build a **scalable, concurrent job execution system**.
* Support **task scheduling**, **retry on failure**, and **status tracking**.
* Provide REST APIs for **task submission, progress, and results**.
* Optionally visualize job statuses through a simple web UI.

---

## 🧠 3. Core Concepts & Skills Practiced

| Concept                      | Description                                                                          |
| ---------------------------- | ------------------------------------------------------------------------------------ |
| **Multithreading**           | Using `ThreadPoolExecutor`, `Callable`, and `Runnable` for concurrent task execution |
| **Asynchronous Programming** | Using `@Async` and `CompletableFuture` for non-blocking operations                   |
| **Thread Safety**            | Managing concurrent state updates using `ConcurrentHashMap` and synchronization      |
| **Scheduling**               | Handling delayed execution and retries using `ScheduledExecutorService`              |
| **Persistence**              | Storing task metadata (status, progress, timestamps) in a relational DB              |
| **Monitoring**               | Exposing REST endpoints for live job tracking                                        |
| **Scalability**              | Future integration with Kafka for distributed task execution                         |

---

## ⚙️ 4. Tech Stack

| Layer                | Technology                                                                      |
| -------------------- | ------------------------------------------------------------------------------- |
| **Language**         | Java 17+                                                                        |
| **Framework**        | Spring Boot 3.x                                                                 |
| **Concurrency APIs** | `ThreadPoolExecutor`, `ScheduledExecutorService`, `CompletableFuture`, `@Async` |
| **Database**         | H2 (dev) / MySQL (prod)                                                         |
| **ORM**              | Spring Data JPA                                                                 |
| **Build Tool**       | Maven                                                                           |
| **Testing**          | JUnit 5, Mockito                                                                |
| **(Optional)**       | Kafka / RabbitMQ (for future scaling)                                           |
| **(Optional)**       | React or Thymeleaf (for task dashboard)                                         |

---

## 🧱 5. High-Level Architecture

```
                ┌───────────────────────┐
                │    Client (REST API)   │
                └──────────┬────────────┘
                           │
                POST /api/tasks
                           │
                           ▼
        ┌────────────────────────────────────┐
        │        Task Orchestrator API       │
        │  - Validate & store task details   │
        │  - Assign unique Task ID           │
        │  - Submit to ThreadPoolExecutor    │
        └───────────┬────────────────────────┘
                    │
                    ▼
          ┌────────────────────────────┐
          │   TaskExecutorService      │
          │  - Executes task in thread │
          │  - Updates progress/status │
          │  - Handles retries         │
          └────────────┬───────────────┘
                       │
                       ▼
             ┌───────────────────────┐
             │     Database (H2)     │
             │ task_id | status | ...│
             └───────────────────────┘

```

---

## 🔍 6. Key Functionalities

### ✅ Phase 1: Core Async Execution

* `/api/tasks` → Accepts task creation requests.
* Generates a unique task ID.
* Submits the job to an executor thread pool.
* Returns task ID immediately (non-blocking).

### ✅ Phase 2: Progress Tracking

* Task updates progress and state (RUNNING → COMPLETED / FAILED).
* `/api/tasks/{id}` → Returns live status, start time, end time, and progress %.

### ✅ Phase 3: Retry & Scheduling

* Implement automatic retry for failed tasks (e.g., 3 attempts).
* Use `ScheduledExecutorService` for delayed re-execution.

### ✅ Phase 4: Monitoring & Metrics

* `/api/tasks` → List all running and completed tasks.
* `/api/tasks/stats` → Returns summary metrics (success rate, avg runtime).
* (Optional) Integrate Spring Boot Actuator endpoints.

### ✅ Phase 5 (Optional): Dashboard UI

* Simple React/Thymeleaf frontend showing all tasks, statuses, and progress bars.

---

## 🧩 7. Database Design (Draft)

**Table: `tasks`**

| Column           | Type      | Description                            |
| ---------------- | --------- | -------------------------------------- |
| `id`             | UUID      | Unique task identifier                 |
| `task_name`      | VARCHAR   | Logical name (e.g., “ReportGenerator”) |
| `status`         | ENUM      | PENDING / RUNNING / COMPLETED / FAILED |
| `progress`       | INT       | Progress in percentage                 |
| `attempts`       | INT       | Retry count                            |
| `created_at`     | TIMESTAMP | When task was submitted                |
| `updated_at`     | TIMESTAMP | Last update                            |
| `result_message` | VARCHAR   | Optional output or error message       |

---

## 🧩 8. Packages Structure

```
com.thamizharasan.asynctaskorchestrator
 ├── controller
 │     └── TaskController.java
 ├── service
 │     ├── TaskService.java
 │     └── TaskExecutorService.java
 ├── model
 │     └── Task.java
 ├── repository
 │     └── TaskRepository.java
 ├── config
 │     └── ExecutorConfig.java
 ├── exception
 │     └── TaskNotFoundException.java
 ├── dto
 │     └── TaskRequest.java / TaskResponse.java
 └── AsyncTaskOrchestratorApplication.java
```

---

## 🧩 9. APIs (Planned)

| Endpoint                | Method | Description                  |
| ----------------------- | ------ | ---------------------------- |
| `/api/tasks`            | `POST` | Submit a new async task      |
| `/api/tasks/{id}`       | `GET`  | Get task details by ID       |
| `/api/tasks`            | `GET`  | List all tasks               |
| `/api/tasks/retry/{id}` | `POST` | Manually retry a failed task |
| `/api/tasks/stats`      | `GET`  | View task statistics summary |

---

## 🧠 10. Example Use Case (Demo)

* Client sends:

  ```json
  POST /api/tasks
  {
    "taskType": "GENERATE_REPORT",
    "payload": { "records": 5000 }
  }
  ```
* Response:

  ```json
  {
    "taskId": "c3f1b6d9-9e7e-4b1b-a76a-5db6a54b1a2d",
    "status": "SUBMITTED"
  }
  ```
* Task runs in background.
* Client polls `/api/tasks/{id}` to track status.

---

## 🧩 11. Future Enhancements

* Add **priority queue** (run high-priority tasks first).
* Integrate **Kafka** for distributed execution.
* Implement **pause/resume** functionality.
* Add **WebSocket push notifications** for task status updates.
* Implement **authentication (Spring Security)**.

---

## 📈 12. Expected Learning Outcomes

* Mastery of **Java concurrency** within a Spring Boot ecosystem.
* Hands-on experience designing **thread-safe systems**.
* Understanding of **real-world async workflows** (used in job schedulers, report generators, background processors).
* Strong **portfolio project** for resume/interviews demonstrating practical backend engineering depth.

