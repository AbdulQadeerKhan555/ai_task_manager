# AI Task Manager 🚀

An AI-native task management system and appointment booking assistant built with natural language at its core. This project leverages the **OpenAI Agents SDK** and the **Model Context Protocol (MCP)** to create a seamless, intelligent coordination experience.

## 🌟 Vision
The AI Task Manager is designed to be more than just a list. It's a personal coordinator that understands intent, manages schedules, and interacts with external tools (like Google Sheets and Notification services) to simplify your daily life.

## ✨ Core Features
- **Natural Language Orchestration:** Add, update, or find tasks simply by talking to the agent.
- **Intelligent Appointment Booking:** A specialized agent that handles bookings and records them directly to Google Sheets.
- **Automated Notifications:** Smart reminders for tasks with deadlines, handled via a dedicated FastAPI service.
- **AI-Native Architecture:** Built from the ground up using specialized agents rather than a monolithic logic block.

## 🏗️ Technical Architecture
The system is divided into four distinct layers:

1.  **Orchestration (OpenAI Agents SDK):** Manages conversation flow and agent handoffs.
2.  **Tools (MCP Server):** A Python-based server implementing the Model Context Protocol to provide standardized access to task data.
3.  **Services:** FastAPI-powered notifications and Google Sheets integration.
4.  **Interface:** A modern Next.js 16 frontend with secure session management via Better Auth.

## 🛠️ Tech Stack
- **Language:** Python 3.12+ (Backend), TypeScript (Frontend)
- **AI Framework:** OpenAI Agents SDK
- **Protocol:** Model Context Protocol (MCP)
- **Web Framework:** FastAPI (Services), Next.js 16 (UI)
- **Auth:** Better Auth
- **Infrastructure:** Designed for Render/Railway deployment

## 📜 The Constitution
This project follows a strict set of principles defined in [AGENTS.md](./AGENTS.md):
- **Test-Driven Development (TDD):** Red, Green, Refactor.
- **KISS & YAGNI:** Simplicity and focus over premature complexity.
- **Statelessness:** Cloud-native and horizontally scalable.

## 🚀 Getting Started (Coming Soon)
*Development is in progress. The initial project structure and "Constitution" have been established.*

---
**Maintained by:** [AbdulQadeerKhan555](https://github.com/AbdulQadeerKhan555)

