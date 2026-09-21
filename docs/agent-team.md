# Agent team

I will use GitHub Copilot CLI in a Codespace to orchestrate the following custom agent team for building Mona's Project Pulse dashboard:

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| **Orchestrator** | Claude Opus 4.7 (copilot) | Coordinates the specialist agents, breaks work into phases, manages file ownership and dependencies, and verifies that the integrated result works together. | `.github/agents/orchestrator.agent.md` |
| **Planner** | Claude Opus 4.7 (copilot) | Researches the repository, documentation, dependencies, edge cases, risks, and validation needs, then produces an actionable implementation plan without writing code. | `.github/agents/planner.agent.md` |
| **Designer** | Gemini 3.1 Pro (copilot) | Leads UI/UX, accessibility, information architecture, interaction flow, visual hierarchy, responsive behavior, and polished Project Pulse dashboard styling. | `.github/agents/designer.agent.md` |
| **Coder** | GPT-5.5 (copilot) | Implements assigned code, keeps behavior explicit and testable, creates required runnable-app support configuration, and validates the changes. | `.github/agents/coder.agent.md` |

The Orchestrator will start with the Planner, then coordinate the Designer and Coder in phases that respect their file scopes and dependencies.
