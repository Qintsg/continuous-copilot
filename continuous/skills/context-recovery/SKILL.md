---
name: context-recovery
description: 在上下文压缩或错误恢复后恢复任务主线的技能
user-invocable: false
---

# Context Recovery

## When to use this skill
Use this skill whenever any of the following is true:
- context compaction has occurred;
- the conversation has become long and focus may be degrading;
- the session is being resumed after interruption;
- an error occurred and the task thread needs to be rebuilt;
- the main agent suspects the current execution state is drifting away from the plan.

## Recovery order
1. Re-read `AGGENTS.md`. If it does not exist, re-read `AGENTS.md`.
2. Re-read `.github/dida365-state.md`.
3. Re-establish the current goal, the user’s latest requirements, and the key constraints.
4. Restore the global plan and current subtask plan.
5. Identify completed work, unfinished work, and the next action.
6. Confirm key modified files, key validation results, current risks, and blockers.
7. Confirm whether an intensive chat session is still open; if it is no longer needed, close it promptly with `interactive-mcp/stop_intensive_chat`.
8. If major uncertainty still remains after recovery, decide whether user confirmation is needed.
9. If necessary, pull the remote Dida365 state again and resync the local mirror.

## Desired outcome
After recovery, the main agent should again be able to answer clearly:
- what it is doing now;
- why it is doing it;
- what has already been done;
- what the next step is;
- whether the Dida365 state needs to be updated;
- whether user confirmation is required;
- whether any intensive chat session is still open.

## Additional constraints
- Do not resume blind editing during recovery.
- Restore task state first, then continue.
- If the plan is outdated or inconsistent with the current state, update the plan and task state before moving on.
- Never use `interactive-mcp/message_complete_notification`.