# MCP Design Decisions: Tasks Server

This document tracks the architectural decisions for the Tasks MCP Server to ensure consistency across agents and deployment environments.

---

## 1. Transport & Deployment Shape

We will use **Streamable HTTP** as the transport layer, specifically in a **Stateless JSON** configuration. This is critical for our goal of deploying to cloud environments like Render, Railway, or Kubernetes.

### Sub-decision A: Transport
*   **Decision:** Streamable HTTP (not stdio).
*   **Rationale:** stdio is local-only. HTTP allows multiple agents across different network pods to connect to a single long-lived service.

### Sub-decision B: Response mode (stateless JSON vs SSE streaming)
*   **Decision:** Stateless JSON.
*   **Rationale:** 
    *   Stateless JSON follows a simple `POST /mcp` -> `JSON Response` pattern. 
    *   It removes the need for "session affinity" (sticky sessions), making horizontal scaling and rolling deployments trivial.
    *   While we lose "progress notifications" mid-call, our "tools-only" server (request -> work -> respond) does not require them. Matches the "simplest version" goal.

### Sub-decision D: Session IDs
*   **Decision:** No Sessions.
*   **Rationale:** Every request stands alone. This avoids the complexity of tracking which pod holds a specific session and ensures every pod can serve any request.

---

## 2. Tool Naming Convention
*   **Decision:** All tools will be prefixed with `tasks_`.
*   **Example:** `tasks_add_and_schedule`, `tasks_get_agenda`.

---

## 3. Tool Granularity
*   **Decision:** Prioritize **Agentic Workflow Tools** over atomic CRUD.
*   **Rationale:** Surfacing raw database operations to an agent is inefficient. We design tools that encapsulate end-to-end human intents (e.g., Capture, Review, Action) in a single call to minimize token usage and latency.

---

## 4. Context Management
*   **Decision:** Intent-based summarization (e.g., `get_agenda`) over raw list pagination.
*   **Rationale:** The server should handle filtering and sorting to return only high-signal information tailored for an LLM's context window.

---

## 5. Safety
*   **Decision:** Use MCP `destructiveHint` for task deletion or major state changes to trigger user confirmation.

---

# Task Management Tools & Data Model

This document defines the core data structure and the intent-based MCP tool interfaces for the Tasks Management System.

---

## 1. The Task Data Model

Each task will have the following properties:

| Property | Type | Description |
| :--- | :--- | :--- |
| `id` | `string` | Unique identifier (UUID). |
| `title` | `string` | Short summary of the task (Required). |
| `description` | `string` | Detailed notes (Optional). |
| `status` | `enum` | `todo`, `in_progress`, `completed`. |
| `due_at` | `string` | ISO 8601 timestamp for deadline (Optional). |
| `remind_at` | `string` | ISO 8601 timestamp for notification (Optional). |
| `created_at` | `string` | ISO 8601 timestamp of creation. |
| `updated_at` | `string` | ISO 8601 timestamp of last update. |

---

## 2. Intent-Based Tool Definitions

### `tasks_add_and_schedule`
Captures a new task and automatically schedules a reminder if a time is provided.
*   **Input:** `title` (required), `description` (optional), `remind_at` (optional).
*   **Logic:** Saves the task to the store and immediately registers a notification via the Notifications Service.
*   **Returns:** The newly created `Task` object.

### `tasks_get_agenda`
Retrieves a curated summary of tasks for immediate review.
*   **Input:** `date` (optional, defaults to today).
*   **Logic:** Returns a filtered list containing "Overdue" tasks and tasks "Due Today," sorted by priority/time.
*   **Returns:** A high-signal summary string or a list of `Task` objects.

### `tasks_mark_done`
Completes a task and cleans up associated resources.
*   **Input:** `id` (required).
*   **Logic:** Updates status to `completed` and automatically cancels any pending notifications for this task.
*   **Returns:** Success confirmation.

### `tasks_reschedule_or_edit`
Modifies a task and re-syncs any associated schedules.
*   **Input:** `id` (required), plus any of `title`, `description`, `remind_at`.
*   **Logic:** Updates the record and re-calculates/updates the notification trigger.
*   **Returns:** The updated `Task` object.

---

## 3. Storage Strategy (v1)
*   **Type:** In-memory (Global Python dictionary).
*   **Continuity:** Since we are using **Stateless JSON**, the in-memory store will be reset on every server restart. This is acceptable for the initial development phase.
