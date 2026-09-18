---
name: session-handoff
description: Use when a Codex task must be paused for another conversation or resumed from a prior handoff, especially when project state, verification gates, or the next safe action must be re-established.
---

# Session Handoff

## Core principle

A handoff is an **anchor, not authority**. Preserve its intent, then verify decision-critical claims against the current project assets. Never silently choose the handoff or the workspace when they disagree.

## Choose a mode

| Situation | Mode | Required reference |
|---|---|---|
| Pause or transfer the current task | `prepare` | Read [references/handoff-template.md](references/handoff-template.md) |
| Resume from a prior conversation or handoff | `recover` | Read [references/recovery-template.md](references/recovery-template.md) |

If the request combines both, recover and obtain confirmation before preparing a new handoff.

## Invariants

- Treat the real project root, applicable instruction files, tracked/untracked changes, current revision, artifacts, and current verification records as primary evidence.
- Keep `Machine Gate` separate from `Human Confirmation Gate`. Automated success never implies human approval.
- Link every decision-critical claim through an `Evidence Map`; label unsupported claims as unknown rather than filling gaps from memory.
- Keep `Next Action` singular and safe. Record prerequisites and a `Do Not Do` list that prevents plausible damage or rework.
- Do not expand authority. Read-only inspection is allowed; external mutations, destructive actions, releases, or messages still require their ordinary authorization.

## Recover stop rule

The first recover pass is verification, not continuation. Inspect the live state, compare it with the anchor, output a `Recovery Verification Report`, classify it as exactly one of `MATCH`, `CONFLICT`, or `INSUFFICIENT_EVIDENCE`, then **STOP and wait for explicit user confirmation**. This applies even for `MATCH` and even when the user initially says to continue.

Do not execute the proposed next action, repair a conflict, rerun a potentially state-changing check, or manufacture missing evidence before that confirmation. If the user confirms, continue only within the confirmed scope.

## Persistence

In `prepare`, when the project is writable and persistence is permitted, write the same completed handoff to:

- `.handoff/LATEST.md`
- `.handoff/history/<timestamp>.md`

If either path cannot be written safely, provide the full handoff in chat and label it `chat-only` with the reason. Do not seek elevated access merely to persist it. V1 uses no watcher, daemon, background monitor, or automatic continuation.

## Quick checks

| Temptation | Required response |
|---|---|
| “The summary is obvious; chat is enough.” | Persist `LATEST + history` when safely possible. |
| “No contradiction means MATCH.” | Use `INSUFFICIENT_EVIDENCE` when proof is missing or stale. |
| “Machine PASS means approval.” | Keep the human gate pending or unknown without explicit evidence. |
| “The conflict is easy to resolve.” | Report it and stop; let the user choose. |

## Red flags

- Starting the next task before the recover report and confirmation
- Citing only the handoff instead of current assets
- Omitting the verdict, Evidence Map, Next Action, Do Not Do, or either gate
- Calling stale results current
- Producing two different handoff bodies for `LATEST` and history
