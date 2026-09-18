# Agent team

Mona's Project Pulse dashboard will be built by a four-agent team coordinated through GitHub Copilot CLI in a Codespace.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 4.7 (Copilot) | Breaks the project into phases, delegates work to the specialists, manages dependencies and file ownership, and verifies the integrated result. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 (Copilot) | Researches the repository and relevant documentation, identifies requirements and risks, and produces an ordered implementation plan with file assignments and validation expectations. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (Copilot) | Implements the dashboard logic and runnable application support with explicit errors, deterministic behavior, and validation. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.1 Pro (Copilot) | Shapes the dashboard's UI/UX, accessibility, information hierarchy, responsive layout, status and priority treatments, and visual polish. | `.github/agents/designer.agent.md` |

The Orchestrator will ask the Planner for the implementation strategy, then assign non-overlapping work to the Coder and Designer in parallel where possible and sequence dependent or overlapping changes. GitHub Copilot CLI is the control point for invoking the agents and coordinating their work in the Codespace; the learner retains control of staging, commits, and pushes.

Orchestrator, Planner, Coder, and Designer.
The model assigned to each agent.
The responsibility of each agent.
The .github/agents/ file for each agent.
How the team will work together to build Project Pulse.
