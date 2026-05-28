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
*   **Example:** `tasks_create`, `tasks_list`.

---

## 3. Tool Granularity
*   **Decision:** Prioritize **Atomic Tools** (CRUD operations) first for maximum agent flexibility.

---

## 4. Context Management
*   **Decision:** Mandatory pagination and filtering to prevent context window overflow.

---

## 5. Safety
*   **Decision:** Use MCP `destructiveHint` for task deletion to trigger user confirmation.

---

# Task Management Tools & Data Model

This document defines the core data structure and the MCP tool interfaces for the Tasks Management System.

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
| `created_at` | `string` | ISO 8601 timestamp of creation. |
| `updated_at` | `string` | ISO 8601 timestamp of last update. |

---

## 2. Tool Definitions

### `tasks_create`
Adds a new task to the system.
*   **Input:** `title` (required), `description` (optional), `due_at` (optional).
*   **Returns:** The newly created `Task` object.

### `tasks_list`
Retrieves a list of tasks with filtering.
*   **Input:** 
    *   `status`: Filter by status (optional).
    *   `limit`: Max tasks to return (default: 50).
    *   `offset`: Number of tasks to skip (for pagination).
*   **Returns:** An array of `Task` objects.

### `tasks_get`
Retrieves a single task by its ID.
*   **Input:** `id` (required).
*   **Returns:** The `Task` object or an error if not found.

### `tasks_update`
Modifies an existing task.
*   **Input:** `id` (required), and any of `title`, `description`, `status`, `due_at`.
*   **Returns:** The updated `Task` object.

### `tasks_delete`
Removes a task from the system.
*   **Input:** `id` (required).
*   **Annotations:** `destructiveHint: true`.
*   **Returns:** A success confirmation message.

---

## 3. Storage Strategy (v1)
*   **Type:** In-memory (Global Python dictionary).
*   **Continuity:** Since we are using **Stateless JSON**, the in-memory store will be reset on every server restart. This is acceptable for the initial development phase.
