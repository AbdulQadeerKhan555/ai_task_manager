# AGENTS.md — The Project Constitution

This is the authoritative guide for the Tasks Management System. It defines our culture, our standards, and our methodology. Every contributor—human or AI—must uphold these principles.

---

## 1. Mission Statement
The Tasks Management System is a resilient, AI-native platform designed to simplify life through natural language. We build with the user at the center, ensuring every interaction is intuitive, secure, and reliable.

---

## 2. Core Programming Principles
We adhere to these foundational principles to ensure code quality and maintainability:

- **Test-Driven Development (TDD):** No logic is implemented without a preceding failing test. Tests are our documentation and our safety net. **Red, Green, Refactor** is our mantra.
- **Keep It Simple, Stupid (KISS):** We prefer simple, readable code over clever optimizations. Complexity is a liability.
- **You Aren't Gonna Need It (YAGNI):** We build for today's requirements, not tomorrow's guesses.
- **Don't Repeat Yourself (DRY):** We abstract common logic only when it provides clear value, avoiding premature abstraction.
- **Security by Design:** Every layer is built with the assumption of potential misuse. We validate inputs and enforce guardrails at every boundary.

---

## 3. Development Workflow
Every task follows a strict, iterative lifecycle:

1. **Research & Reproduction:** Understand the problem, map existing code, and validate assumptions. For bugs, always reproduce with a test first.
2. **Strategy:** Design a high-level solution. Identify the agents, tools, and services involved.
3. **Execution (The TDD Cycle):**
   - **Red:** Write a failing test for the new feature or bug.
   - **Green:** Implement the minimal code required to pass the test.
   - **Refactor:** Clean the code while keeping tests green.
4. **Validation:** Run the full suite of tests, linters, and type-checkers before completion.

---

## 4. Architecture Vision
Our system is a symphony of specialized experts working in concert:

- **Specialists, not Generalists:** Each agent has a narrow focus and a clear set of instructions. If an agent's prompt becomes a "wall of text," it must be split into multiple specialized agents.
- **Standardized Communication:** We use the **OpenAI Agents SDK** for orchestration and the **Model Context Protocol (MCP)** for tool integration.
- **Explicit Handoffs:** When a task moves between specialists, the transfer of context must be deliberate and clear.
- **Statelessness:** Agents and services should remain stateless wherever possible, relying on persistent session stores for continuity.

---

## 5. Implementation Standards

- **OpenAI Agents SDK:** Utilize `Agent`, `@tool`, and `Handoff` primitives. Prefer `Runner.run()` (asynchronous) to ensure the system is production-ready and non-blocking.
- **FastMCP:** Build all MCP servers using the `FastMCP` framework. This ensures protocol compliance and allows for rapid iteration of tools, resources, and prompts.
- **UI & Auth:** Next.js 16 for the interface and Better Auth for secure session management. Logic belongs in the agent/service layer, not the UI.

---

## 6. Deployment Perspective (Render/Railway)
We build for the cloud from day one. Every component must be "Twelve-Factor" ready:

- **Configuration:** All configuration is injected via environment variables. Never hardcode secrets.
- **Portability:** Use `pyproject.toml` and standard dependency management to ensure the environment is reproducible.
- **Observability:** Log to `stdout`. Implement health check endpoints (`/health`) for all services.
- **Scalability:** Design for horizontal scaling. Avoid local file storage; use cloud-native alternatives (e.g., Google Sheets, external databases).

---

## 7. Safety & Guardrails
- **Input/Output Validation:** Use the SDK's guardrail primitives to enforce safety and logic constraints. Do not rely on "asking nicely" in the system prompt.
- **Credential Protection:** Secrets must be protected at all costs. Never log or commit `.env` files.
- **Auditability:** Every tool call and agent handoff must be traceable for debugging and security auditing.
