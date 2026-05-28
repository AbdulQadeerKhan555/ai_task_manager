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
