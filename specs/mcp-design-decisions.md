# MCP Design Decisions: Tasks Server

This document tracks the architectural decisions for the Tasks MCP Server to ensure consistency across agents and deployment environments.

---

## 1. Transport Mechanism: SSE (Server-Sent Events)

### Decision
We will use **SSE** as the primary transport mechanism for the Tasks MCP server.

### Rationale
*   **Remote Accessibility:** Unlike `stdio`, which requires the agent to run the server as a local subprocess, SSE allows the server to run as a web service (e.g., on Render, Railway, or Kubernetes).
*   **Multi-Agent Support:** Multiple agents (Tasks Manager, Appointment Agent, etc.) can connect to the same server URL simultaneously.
*   **Cloud Native:** It follows standard HTTP patterns, making it easier to manage behind load balancers and within containerized environments.

### Implementation (Simplest Version)
*   The server will run using **FastAPI** or **Starlette**.
*   It will expose an `/sse` endpoint for the MCP connection.
*   **Note:** No authentication will be implemented in the initial version to maintain simplicity.

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
