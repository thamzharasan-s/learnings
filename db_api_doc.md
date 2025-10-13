# Async Task Orchestrator - DB & API Specification

## 1. Database Design

### Table: `tasks`

| Column           | Type        | Description                                        |
| ---------------- | ----------- | -------------------------------------------------- |
| `id`             | UUID        | Unique task identifier                             |
| `task_name`      | VARCHAR     | Logical name of the task (e.g., "ReportGenerator") |
| `status`         | ENUM        | PENDING / RUNNING / COMPLETED / FAILED             |
| `progress`       | INT         | Current progress in percentage (0–100)             |
| `attempts`       | INT         | Number of retries attempted                        |
| `max_attempts`   | INT         | Maximum allowed retries (default = 3)              |
| `payload`        | JSON / TEXT | Optional task data or parameters                   |
| `scheduled_at`   | TIMESTAMP   | Optional: when the task is scheduled to start      |
| `created_at`     | TIMESTAMP   | Task creation timestamp                            |
| `updated_at`     | TIMESTAMP   | Last status/progress update                        |
| `result_message` | TEXT        | Optional: output or error message                  |

### Table: `task_logs` (Optional for history)

| Column       | Type      | Description                       |
| ------------ | --------- | --------------------------------- |
| `id`         | BIGINT    | Auto-increment primary key        |
| `task_id`    | UUID      | Foreign key to `tasks.id`         |
| `status`     | ENUM      | Status at this log point          |
| `progress`   | INT       | Progress percentage at this point |
| `message`    | TEXT      | Any log message or error info     |
| `created_at` | TIMESTAMP | Log timestamp                     |

### Indexing & Tips

* Primary key on `tasks.id`
* Index on `status` for fast querying of running/pending tasks
* Index on `scheduled_at` for scheduled/delayed execution
* Compound index on `attempts` + `status` for retry queries

### ER Diagram (Conceptual)

```
┌───────────────┐       1-to-many      ┌───────────────┐
│    tasks      │────────────────────>│  task_logs    │
├───────────────┤                     ├───────────────┤
│ id (PK)       │                     │ id (PK)       │
│ task_name     │                     │ task_id (FK)  │
│ status        │                     │ status        │
│ progress      │                     │ progress      │
│ attempts      │                     │ message       │
│ max_attempts  │                     │ created_at    │
│ payload       │                     └───────────────┘
│ scheduled_at  │
│ created_at    │
│ updated_at    │
│ result_message│
└───────────────┘
```

### Example Queries

```sql
-- Get all running tasks
SELECT * FROM tasks WHERE status = 'RUNNING';

-- Get tasks scheduled to run now or in the past
SELECT * FROM tasks WHERE scheduled_at <= NOW() AND status = 'PENDING';

-- Insert new task
INSERT INTO tasks (id, task_name, status, progress, attempts, max_attempts, payload, scheduled_at, created_at, updated_at)
VALUES (UUID(), 'GenerateReport', 'PENDING', 0, 0, 3, '{"records":5000}', NOW(), NOW(), NOW());
```

---

## 2. API Specification

| Endpoint                    | Method | Description                                                                    |
| --------------------------- | ------ | ------------------------------------------------------------------------------ |
| `/api/tasks`                | POST   | Submit a new async task (optionally with `scheduled_at` for delayed execution) |
| `/api/tasks/{taskId}`       | GET    | Get details of a specific task including status, progress, and message         |
| `/api/tasks`                | GET    | List all tasks (filter optional: PENDING, RUNNING, COMPLETED, FAILED)          |
| `/api/tasks/retry/{taskId}` | POST   | Manually retry a failed task (increments retry count)                          |
| `/api/tasks/stats`          | GET    | Return metrics: success rate, avg runtime, tasks in progress                   |

### Optional Future APIs

| Endpoint                     | Method | Description                     |
| ---------------------------- | ------ | ------------------------------- |
| `/api/tasks/cancel/{taskId}` | POST   | Cancel a running or queued task |
| `/api/tasks/pause/{taskId}`  | POST   | Pause a scheduled/queued task   |
| `/api/tasks/resume/{taskId}` | POST   | Resume a paused task            |

### Sample cURL Requests

**Submit a Task:**

```bash
curl -X POST http://localhost:8080/api/tasks \
  -H 'Content-Type: application/json' \
  -d '{"taskName":"GenerateReport","payload":{"records":5000}}'
```

**Get Task Status:**

```bash
curl -X GET http://localhost:8080/api/tasks/<taskId>
```

**List All Tasks:**

```bash
curl -X GET http://localhost:8080/api/tasks
```

**Retry Failed Task:**

```bash
curl -X POST http://localhost:8080/api/tasks/retry/<taskId>
```

**Get Task Stats:**

```bash
curl -X GET http://localhost:8080/api/tasks/stats
```

