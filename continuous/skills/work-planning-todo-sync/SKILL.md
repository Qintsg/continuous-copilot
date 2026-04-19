---
name: work-planning-dida-sync
description: 在需要先确认工作规划时同步滴答清单状态的技能
user-invocable: false
---

# Work Planning and Dida365 Synchronization

## When to use this skill
Use this skill whenever any of the following is true:
- a new conversation has just started;
- the repository does not yet have a bound Dida365 project/list or local mirror state;
- the user has introduced a new non-trivial task;
- the user has changed the goal, priority, scope, or deliverables;
- the task has entered a new stage and the plan should be refreshed.

## Steps
1. Read `AGGENTS.md` first. If it does not exist, read `AGENTS.md`, README, the current local mirror state if present, and the repository structure.
2. Resolve the Dida365 project/list name:
   - first check for explicit fields such as `Project Name:`, `项目名称：`, `MCP Project:`, or `MCP项目：`;
   - if none exist, use the Git repository name;
   - always normalize the final name to `mcp-<projectname>`.
3. If this is a new conversation, or the remote state is not yet established, confirm work planning first.
4. Use `interactive-mcp/request_user_input` for one-shot planning questions.
5. If the planning phase clearly requires multiple continuous questions, start an intensive chat with `interactive-mcp/start_intensive_chat` and continue with `interactive-mcp/ask_intensive_chat`.
6. Convert the confirmed information into two planning layers:
   - global plan
   - current subtask plan
7. Create or update the corresponding `mcp-<projectname>` Dida365 project/list and task state.
8. Update the local mirror file `.github/dida365-state.md` so that it includes at least:
   - MCP project/list name
   - name source
   - whether the remote project/list is ready
   - current phase
   - current subtask
   - remaining unfinished item count
   - whether an intensive chat is still open
   - last sync timestamp
9. Re-read `.github/dida365-state.md` after every important stage and keep it aligned with the latest plan and remote state.
10. If the user changes requirements, re-confirm before updating the Dida365 state.
11. If an intensive chat was started for this phase, call `interactive-mcp/stop_intensive_chat` once the planning interaction is complete.
12. Before completion, check:
   - whether the remote Dida365 state reflects the final status
   - whether `.github/dida365-state.md` reflects the final status
   - whether any work items remain unresolved
   - whether the required interactive confirmation flow has been completed
   - whether there is no unclosed intensive chat session

## Additional constraints
- Do not mix general documentation and mirror state in the same file.
- Never use `interactive-mcp/message_complete_notification`.
- The Dida365 remote state and `.github/dida365-state.md` local mirror together form the task-state source of truth.
- If the project rules conflict with this skill, the project rules win.