# Agent team

To build Mona's Project Pulse dashboard, I will use a four-agent custom team defined under the repository agent folder.

- Orchestrator
	- Target model: Claude Opus 4.7 (copilot)
	- Responsibility: Coordinates the workflow across specialist agents, sequences phases, and validates that integrated outputs fit together.
	- Definition: .github/agents/orchestrator.agent.md

- Planner
	- Target model: Claude Opus 4.7 (copilot)
	- Responsibility: Researches the codebase and requirements, identifies dependencies and risks, and produces an implementation plan with execution order and parallelization guidance.
	- Definition: .github/agents/planner.agent.md

- Coder
	- Target model: GPT-5.5 (copilot)
	- Responsibility: Implements application logic and supporting runnable-app configuration in assigned files, with explicit behavior and validation.
	- Definition: .github/agents/coder.agent.md

- Designer
	- Target model: Gemini 3.1 Pro (copilot)
	- Responsibility: Owns UI/UX quality for the dashboard, including structure, accessibility, visual clarity, responsive behavior, and Project Pulse styling expectations.
	- Definition: .github/agents/designer.agent.md

This work is orchestrated using GitHub Copilot CLI in a Codespace, with the Orchestrator delegating scoped tasks to Planner, Coder, and Designer.
