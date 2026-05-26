# AGENTS.md — The Project Constitution

This is the authoritative guide for the Tasks Management System. It defines our culture, our standards, and our methodology. Every contributor—human or AI—must uphold these principles.

---

## 1. Mission Statement
The Tasks Management System is a resilient, AI-native platform designed to simplify life through natural language. We build with the user at the center, ensuring every interaction is intuitive, secure, and reliable.

---

## 2. Core Programming Principles
We adhere to these foundational principles to ensure code quality and maintainability:

- **Verification First:** **Always verify what you are doing.** Never assume success. Empirical evidence (logs, tests, status codes) is the only source of truth.
- **Test-Driven Development (TDD):** No logic is implemented without a preceding failing test. **Red, Green, Refactor** is our mantra.
- **Keep It Simple, Stupid (KISS):** We prefer simple, readable code over clever optimizations.
- **You Aren't Gonna Need It (YAGNI):** Build for today's requirements, not tomorrow's guesses.
- **Security by Design:** Validate every input and enforce guardrails at every boundary.

---

## 3. The Creator's Workflow
Every task follows a strict **Research -> Plan -> Execute -> Verify** lifecycle. We do not skip steps.

1.  **Research:** Map the codebase, read relevant docs, and identify dependencies.
2.  **Plan (Plan Mode):** For all non-trivial tasks, draft a plan and obtain alignment before writing code. Define the "Definition of Done."
3.  **Execute (TDD Cycle):**
    - **Red:** Write a failing test.
    - **Green:** Implement the minimal code to pass.
    - **Refactor:** Clean and optimize while keeping tests green.
4.  **Verify:** Rigorously validate the output using automated tests, manual checks, and status verifications. **Validation is non-negotiable.**

---

## 4. Architecture & Specialized Agents
Our system is a symphony of specialized experts working in concert:

- **Specialists, not Generalists:** Each agent has a narrow focus. If a prompt grows too large, split the agent.
- **Context Management:** Isolate workstreams to prevent context pollution. Keep session history lean and focused.
- **Standardized Communication:** Use the **OpenAI Agents SDK** and **MCP** for all tool and data interactions.
- **Explicit Handoffs:** Transfers of context between agents must be deliberate and recorded.

---

## 5. Implementation Standards
- **Tech Stack Mandates:** Use **Python 3.12+** for all backend logic and **uv** as the primary package manager for performance and reliability.
- **OpenAI Agents SDK:** Utilize `Agent`, `@tool`, and `Handoff`. Use `Runner.run()` (async) for production readiness.
- **FastMCP:** Build all MCP servers using the `FastMCP` framework for rapid, compliant tool development.
- **Self-Evolving Documentation:** Every correction or lesson learned must be encoded back into `AGENTS.md` or `GEMINI.md` to prevent regressions.

---

## 6. Deployment Perspective (Render/Railway)
- **Twelve-Factor Ready:** Configuration via environment variables, stateless services, and `stdout` logging.
- **Health Checks:** Implement `/health` endpoints for all services.
- **Scalability:** Design for horizontal scaling and cloud-native persistence.

---

## 7. Safety & Auditability
- **SDK Guardrails:** Use primitives for input/output validation.
- **Credential Protection:** Secrets are never logged or committed.
- **Traceability:** Every tool call and handoff must be traceable for auditing and debugging.

