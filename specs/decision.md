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
*   **Example:** `tasks_capture`, `tasks_review`.

---

## 3. Tool Granularity: The 5-Intent Model
*   **Decision:** Model the unit of user work, not the unit of database mutation.
*   **Rationale:** We use five intent-based tools that cover ~everything a human actually does with a task manager. This minimizes token usage and failure points by finishing a coherent piece of work in one call.

---

## 4. Context Management
*   **Decision:** Intent-based summarization (via `tasks_review`) over raw list pagination.
*   **Rationale:** The server filters and sorts data to return high-signal summaries (e.g., "Overdue", "Today"), keeping the agent's context window lean.

---

## 5. Safety
*   **Decision:** Use MCP `destructiveHint` for `tasks_remove` to trigger user confirmation.

---

# Task Management Tools & Data Model

This document defines the core data structure and the intent-based MCP tool interfaces for the Tasks Management System.

---

## 1. The Task Data Model

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

## 2. The 5-Intent Tool Definitions

### `tasks_capture`
**Intent:** "Remind me to call mom tomorrow at 5"
*   **Input:** `title` (required), `description` (optional), `remind_at` (optional).
*   **Logic:** Saves the task and schedules a notification in one step.

### `tasks_review`
**Intent:** "What's on my plate today?" / "What's overdue?"
*   **Input:** `query` (optional - e.g., "today", "overdue", "all").
*   **Logic:** Returns a curated, LLM-friendly summary of relevant tasks.

### `tasks_modify`
**Intent:** "Move the dentist to Wednesday and rename it"
*   **Input:** `id` (required), `title` (optional), `due_at` (optional), `remind_at` (optional).
*   **Logic:** Updates the record and re-syncs any associated notification triggers.

### `tasks_resolve`
**Intent:** "I finished the report" / "Skip the gym"
*   **Input:** `id` (required), `resolution` (default: "completed").
*   **Logic:** Marks the task done and cancels any pending notifications.

### `tasks_remove`
**Intent:** "Delete that grocery task - I don't need it"
*   **Input:** `id` (required).
*   **Annotations:** `destructiveHint: true`.
*   **Logic:** Permanently deletes the task and its schedule.

---

## 3. Storage Strategy (v1)
*   **Type:** In-memory (Global Python dictionary).
*   **Continuity:** Reset on server restart (acceptable for initial development).
