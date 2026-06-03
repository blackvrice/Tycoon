# Agent Instructions

## Scope

- These instructions apply to the entire repository.
- Follow them for Codex, Antigravity, Claude, and any other coding agent that reads this file.
- Direct user instructions in the current conversation take priority when they conflict with this file.

## Code Comments

- When adding or changing code, write comments for logic whose purpose, constraints, or side effects are not obvious from the code itself.
- Add or update comments when touching Unity scene wiring, prefabs, ScriptableObjects, runtime managers, save data, or UI bindings where future readers need context.
- Keep comments concise and useful. Do not add comments that merely repeat what the next line of code already says.
- If behavior changes, update stale comments in the same area as part of the change.

## Git Workflow

- Check `git status --short` before and after changes.
- Do not stage, commit, revert, or clean unrelated user changes.
- Stage only the files changed for the current task.
- Run the most relevant available verification before committing, such as tests, build checks, or at minimum `git diff --check` for the touched files.
- If verification cannot be run, explain why in the final response.
- After code or repository file changes are complete, create a commit with a short, clear message.
- Push the committed branch to its configured remote.
- If commit or push fails because of conflicts, authentication, missing remote configuration, or rejected updates, stop and report the exact blocker.

## Final Response

- Summarize what changed.
- Include the verification that was run and its result.
- Mention the commit and push result.
