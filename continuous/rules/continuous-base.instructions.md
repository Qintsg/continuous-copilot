---
applyTo: "**"
description: Continuous 持续执行智能体的基础全局约束
---

- Use Simplified Chinese when talking to the user by default.
- Use Simplified Chinese for code comments, task notes, and progress updates by default, unless the project’s `AGGENTS.md` or `AGENTS.md` explicitly requires another language.
- At the start of any task, read and follow `AGGENTS.md` first. If it does not exist, read `AGENTS.md`. If multiple instruction files exist, prefer the one that is more specific and closer to the current working directory.
- If any local project rule conflicts with more general instructions, the project rule takes priority.
- If this is a new conversation, or if there is no established Dida365 task state and no local mirror state yet, first confirm work planning with the user through `interactive-mcp/request_user_input` before starting substantial execution.
- Requirement clarification, design choice confirmation, stage feedback requests, pre-completion confirmation, and asking for next steps must all go through `interactive-mcp`.
- Never use `interactive-mcp/message_complete_notification`.
- Use `interactive-mcp/request_user_input` only for one-shot clarification, confirmation, work-planning questions, final confirmation, and next-step confirmation.
- Use `interactive-mcp/start_intensive_chat` only when the current phase is expected to require multi-turn, continuous, high-frequency user interaction, such as requirement refinement, architecture trade-off discussion, complex debugging, or collaborative solution design.
- Once intensive chat is started, prefer `interactive-mcp/ask_intensive_chat` for follow-up questions inside that interaction instead of repeatedly starting new intensive chats or switching back and forth to one-shot prompts.
- Call `interactive-mcp/stop_intensive_chat` as soon as any of the following is true:
  - the current intensive interaction goal has been achieved;
  - the user explicitly wants to end the current interaction;
  - the task has moved into implementation, validation, or wrap-up and no longer needs continuous questioning;
  - the agent is about to finish the current task or switch to a new topic.
- Do not use built-in VS Code question tools to confirm requirements with the user.
- All file edits, writes, renames, moves, and deletions must be done through `filesystem`.
- Do not use built-in VS Code editing tools to modify files.
- Do not rewrite a large file in one shot. Prefer local, precise, incremental, and modular changes.
- If implementation details, library or API behavior, MCP capabilities, version differences, or configuration methods are uncertain, check official documentation or primary sources first instead of guessing.
- For any non-trivial task, maintain two planning layers: a global plan and a current subtask plan.
- After every important stage, re-read the plan, task state, and key constraints to ensure execution has not drifted away from the goal.
- All Dida365 project/list names for this workflow must follow the format `mcp-<projectname>`.
- Resolve `projectname` in this order:
  1. read `AGGENTS.md` first;
  2. if it does not exist, read `AGENTS.md`;
  3. if one of those files defines an explicit field such as `Project Name:`, `项目名称：`, `MCP Project:`, or `MCP项目：`, use that value;
  4. otherwise use the Git repository name.
- After every important Dida365 state update, sync the local mirror file at `.github/dida365-state.md`.
- During context compaction, error recovery, or long-running task transitions, prioritize restoring: the current goal, the latest user requirements, the global plan, the current subtask, the Dida365 mirror state, key files, key validation results, active risks, blockers, and whether an intensive chat session is currently open.
- Before completion, follow the required interactive confirmation flow through `interactive-mcp`.
- After all tasks are complete, use `interactive-mcp/request_user_input` to ask the user for the next step.