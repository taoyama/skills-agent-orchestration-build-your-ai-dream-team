# Agent team

This repository defines a custom agent team used to build Mona's Project Pulse dashboard.

| Agent | Model | Responsibility | Definition |
|---|---|---|---|
| Orchestrator | Claude Opus 4.7 (copilot) | Coordinates the Planner, Coder, and Designer agents; breaks requests into phases, assigns non-overlapping file scopes, and reports final outcomes. Does not implement work itself. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 (copilot) | Researches the codebase, docs, and dependencies to produce implementation plans with ordered steps, file assignments, dependencies, and edge cases. Does not write code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code-oriented tasks (including Project Pulse app logic and support files like `.vscode/launch.json`) within the file scope assigned by the Orchestrator. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.1 Pro (copilot) | Handles UI/UX, accessibility, information architecture, and visual design for the Project Pulse dashboard, ensuring a polished, responsive interface. | `.github/agents/designer.agent.md` |

I am using GitHub Copilot CLI in a Codespace to orchestrate this work, invoking the Orchestrator agent to delegate tasks to the Planner, Coder, and Designer as needed.
