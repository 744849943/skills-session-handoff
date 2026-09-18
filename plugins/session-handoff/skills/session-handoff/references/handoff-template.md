# Prepare Mode

Use this mode to capture a reliable execution checkpoint, not a general conversation summary.

## Gather evidence

Inspect the real project state in proportion to the task:

- applicable instruction files and project root;
- current revision, branch/worktree state, and relevant changes when version control exists;
- source artifacts, plans, decisions, outputs, and verification records that support the checkpoint;
- explicit human approvals or unresolved decisions.

Do not claim a check passed unless the available record identifies what ran and which state it covered. Mark old results `STALE` when later changes may invalidate them.

## Handoff contract

Produce one body with these sections. Use the user's language, but preserve the field labels where useful for later recovery.

```markdown
# Session Handoff

- Prepared at: <timestamp and timezone>
- Project root: <resolved path>
- Scope: <task being transferred>
- Persistence: <LATEST path + history path, or chat-only with reason>

## Current Objective and Status
<what is complete, incomplete, or blocked>

## Verification Gates
- Machine Gate: PASS | FAIL | NOT_RUN | STALE | UNKNOWN
  - Evidence: <source and observation>
- Human Confirmation Gate: CONFIRMED | PENDING | UNKNOWN
  - Evidence: <explicit approval source, or why absent>

## Evidence Map
| Claim | Current source | Observation |
|---|---|---|
| <decision-critical claim> | <path, revision, command result, or record> | <what it proves and any limit> |

## Decisions and Constraints
<settled choices, boundaries, and open questions>

## Next Action
<one safe action, its prerequisites, and expected result>

## Do Not Do
- <specific action that could cause damage, duplicate work, or bypass a gate>
```

Omit empty narrative, not required fields. If a fact cannot be established, write `UNKNOWN` and explain what evidence is missing.

## Persist

When safe and permitted:

1. Create `.handoff/` and `.handoff/history/` if needed.
2. Choose a collision-resistant timestamp such as `YYYYMMDDTHHMMSSZ` or the local equivalent with offset.
3. Write the completed body to `.handoff/LATEST.md` and `.handoff/history/<timestamp>.md` without changing its substance.
4. Re-read both files and confirm they match.

If persistence fails, return the complete body in chat, set `Persistence: chat-only`, and state the failure. Do not reduce it to a shorter summary.
