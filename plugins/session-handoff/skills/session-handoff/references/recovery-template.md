# Recover Mode

Use this mode before continuing work from a handoff, prior conversation, pasted summary, or `.handoff/LATEST.md`.

## Verify before acting

1. Locate the intended project root and applicable instructions.
2. Read the handoff as an anchor. Extract its objective, gates, evidence claims, next action, and prohibitions.
3. Inspect the current project assets that can confirm or refute each decision-critical claim. Prefer current files and version-control state over narrative memory.
4. Compare anchor and live state without fixing either one.
5. Produce the report below and stop for confirmation.

Use read-only inspection on this first pass. If fresh verification would write files, contact external systems, change state, or take material time, list it as proposed evidence collection rather than running it.

## Verdict rules

Apply these in order; the first matching rule determines the overall verdict:

1. `CONFLICT`: A reliable current asset contradicts any decision-critical anchor claim, or authoritative anchors disagree in a way that changes the next action. One such contradiction makes the overall verdict `CONFLICT`, even when other claims merely lack evidence.
2. `INSUFFICIENT_EVIDENCE`: No decision-critical claim is contradicted, but required evidence is absent, unverifiable, ambiguous, or stale enough that safe continuation cannot be justified.
3. `MATCH`: Every decision-critical anchor claim is supported by current evidence, no material contradiction exists, and both gates are accurately represented.

Silence is not agreement. A missing approval is not a human PASS. A passing run for an older revision is not a current machine PASS.

## Recovery Verification Report contract

```markdown
# Recovery Verification Report

- Project root: <resolved path>
- Handoff anchor: <source>
- Verdict: MATCH | CONFLICT | INSUFFICIENT_EVIDENCE
- Scope checked: <what was and was not inspected>

## State Comparison
| Anchor claim | Current evidence | Finding |
|---|---|---|
| <claim> | <path, revision, or record> | MATCH | CONFLICT | UNKNOWN |

## Verification Gates
- Machine Gate: PASS | FAIL | NOT_RUN | STALE | UNKNOWN
  - Evidence: <current source and limits>
- Human Confirmation Gate: CONFIRMED | PENDING | UNKNOWN
  - Evidence: <explicit source or missing record>

## Evidence Map
| Decision-critical fact | Source | What it proves / cannot prove |
|---|---|---|
| <fact> | <current asset> | <bounded interpretation> |

## Next Action
<one proposed safe action; do not execute it yet>

## Do Not Do
- <action forbidden until the conflict, gap, or gate is resolved>

## Human Confirmation Gate
Confirm whether I should proceed with `<proposed action>` under the verified scope above.
```

## Example classification

If the anchor says tests passed at revision `abc123`, the project is now at `def456`, and no newer run exists, classify the relevant claim `UNKNOWN`, set `Machine Gate: STALE`, and use `INSUFFICIENT_EVIDENCE`. Do not infer failure, rerun automatically, or call the state a match.

After emitting the report, stop. Do not append work product from the proposed next action.
