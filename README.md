# AI Task Manager 🚀

An AI-native task management system that helps users manage tasks, reminders, and appointment-style workflows through natural conversation.

This project is built on the principles of **agentic orchestration**, where specialized AI agents coordinate through strict boundaries to deliver a reliable, modular user experience.

## 🌟 Product Direction
The system provides a clear, reliable, and explicit conversational interface. We prioritize deterministic behavior: time, date, identity, and destructive actions are never guessed; they are explicitly clarified by the AI before execution.

## 🏗️ Architecture & Boundaries
The system is divided into clear operational domains:

1.  **Tasks Manager Agent:** The primary user-facing orchestrator. Owns intent parsing, clarification, routing, and final user confirmation.
2.  **Tasks MCP Server:** The exclusive backend service for all task mutations. Agents do not mutate the database directly; they communicate via the Model Context Protocol (MCP).
3.  **Appointment Booking Agent:** A specialized agent routed to for booking workflows. It handles scheduling but **never** mutates tasks directly.
4.  **Notifications API:** A standalone service triggered only after valid task mutations succeed.

## 🛠️ Tech Stack & Defaults
- **Language:** Python 3.12+ (Backend/Agents), TypeScript (Frontend)
- **Package Manager:** `uv` (Exclusive tool for dependency resolution and environments)
- **AI Framework:** OpenAI Agents SDK
- **Protocol:** Model Context Protocol (MCP) (via `FastMCP`)
- **Infrastructure:** Kubernetes-first (Containers via `ghcr.io`, stateless design)

## 📜 The Constitution
Every design and implementation decision in this repository is governed by our project constitution. 

👉 **[Read AGENTS.md](./AGENTS.md)** before contributing. 

Key principles include:
- **Test-First Default:** Write failing tests covering expected behavior before implementing.
- **Docs-First Development:** Verify SDK/Framework behavior against official docs. Outdated assumptions are bugs.
- **Kubernetes From The Start:** All services must be stateless, configurable via environment variables, and equipped with health probes.
- **Always Verify:** Plausible output is not verified output. Prove it works empirically.

## 🚀 Getting Started (Coming Soon)
*Development is currently in the foundational Planning and Research phase.*

---
**Maintained by:** [AbdulQadeerKhan555](https://github.com/AbdulQadeerKhan555)
