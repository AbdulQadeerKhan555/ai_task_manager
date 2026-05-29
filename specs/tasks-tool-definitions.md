# Technical Specification: 5-Intent Task Tools

This section defines the precise technical schemas and internal logic for the Tasks MCP Server.

---

## 1. The Task Schema (Pydantic Model)

```json
{
  "type": "object",
  "properties": {
    "id": { "type": "string", "format": "uuid" },
    "title": { "type": "string" },
    "description": { "type": "string" },
    "status": { "type": "string", "enum": ["todo", "in_progress", "completed"] },
    "due_at": { "type": "string", "format": "date-time" },
    "remind_at": { "type": "string", "format": "date-time" },
    "created_at": { "type": "string", "format": "date-time" },
    "updated_at": { "type": "string", "format": "date-time" }
  },
  "required": ["id", "title", "status", "created_at", "updated_at"]
}
```

---

## 2. Tool Definitions & Logic

### `tasks_capture`
**Intent:** Capture a new task and optional schedule.
*   **Input Schema:**
    ```json
    {
      "type": "object",
      "properties": {
        "title": { "type": "string", "description": "The task summary" },
        "description": { "type": "string", "description": "Optional details" },
        "remind_at": { "type": "string", "format": "date-time", "description": "Optional reminder time (ISO 8601)" }
      },
      "required": ["title"]
    }
    ```
*   **Internal Logic:** 
    1. Generate UUID.
    2. Set `status` to `todo`.
    3. If `remind_at` is provided, trigger the Notification Service.
    4. Save to `tasks_db`.

### `tasks_review`
**Intent:** Get a high-signal overview of tasks.
*   **Input Schema:**
    ```json
    {
      "type": "object",
      "properties": {
        "query": { "type": "string", "enum": ["today", "overdue", "all"], "default": "today" }
      }
    }
    ```
*   **Internal Logic:** 
    1. If `today`: Filter tasks where `status != completed` AND (`due_at` or `remind_at` is today).
    2. If `overdue`: Filter tasks where `status != completed` AND (`due_at` < now).
    3. Sort by `due_at` ascending.

### `tasks_modify`
**Intent:** Update task details or reschedule.
*   **Input Schema:**
    ```json
    {
      "type": "object",
      "properties": {
        "id": { "type": "string", "format": "uuid" },
        "title": { "type": "string" },
        "description": { "type": "string" },
        "due_at": { "type": "string", "format": "date-time" },
        "remind_at": { "type": "string", "format": "date-time" }
      },
      "required": ["id"]
    }
    ```
*   **Internal Logic:** 
    1. Find task by ID (Return 404 if missing).
    2. Patch only provided fields.
    3. If `remind_at` changed, cancel old notification and schedule new one.
    4. Update `updated_at`.

### `tasks_resolve`
**Intent:** Complete or skip a task.
*   **Input Schema:**
    ```json
    {
      "type": "object",
      "properties": {
        "id": { "type": "string", "format": "uuid" },
        "resolution": { "type": "string", "enum": ["completed", "skipped"], "default": "completed" }
      },
      "required": ["id"]
    }
    ```
*   **Internal Logic:** 
    1. Update `status` to `completed` (or skip).
    2. **Crucial:** Cancel any pending notifications associated with this task ID.

### `tasks_remove`
**Intent:** Delete a task permanently.
*   **Input Schema:** `{"id": {"type": "string", "format": "uuid"}}`
*   **Internal Logic:** 
    1. Remove from `tasks_db`.
    2. Cancel all pending notifications.
*   **Annotation:** `destructiveHint: true`.

---

## 3. Storage Strategy (v1)
*   **Type:** In-memory (Global Python dictionary `tasks_db: dict[str, Task]`).
*   **Thread Safety:** Use `asyncio.Lock()` for all write operations.
