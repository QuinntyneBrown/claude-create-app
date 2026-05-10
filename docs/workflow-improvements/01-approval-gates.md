# 01 — Approval gates at phase boundaries

## Problem

The workflow today runs end-to-end without human input. A `/loop 1m` invocation will progress from Discovery → Requirements → MVPs → Plans → Tasks → Implementation in one continuous pass. If the requirements drift from what the human actually wanted — wrong scope, missing feature, misread of the idea — the divergence is not visible until the final diff. By that point every downstream task has amplified the original misread.

The evaluation tasks (R2, MB2, MF2, BP2, FP2, BT2, FT2) currently evaluate *internal consistency* (does this artifact comply with the guidance?) — not *intent fidelity* (is this still what the human wanted?).

## Proposed change

Add an explicit `awaiting-approval` status that the loop respects as a hard stop. Apply it at the phase-boundary evaluation tasks where intent drift would compound:

- **R2** — requirements approved before any MVP work.
- **MB2 / MF2** — MVP shape approved before plan work.
- **BP2 / FP2** — implementation plans approved before slice work.

Concretely:

- Add a new `data-status` value: `awaiting-approval`.
- The agent transitions an evaluation task to `awaiting-approval` once internal-consistency passes are clean.
- The loop's task-picker treats `awaiting-approval` as blocking for downstream tasks.
- The human flips the status to `done` (or back to `in-progress` with feedback notes appended to the evaluation file) to release the loop.

## Tradeoff

Less autonomy. The user has to babysit at three points instead of zero. For an experienced user with a well-scoped idea this is friction; for a vague idea it's the difference between catching drift in 2 minutes versus catching it in 60.

A weaker variant — "soft gate" — would have the agent post a one-paragraph summary at each boundary and continue after a fixed timeout if no objection lands. This trades determinism for less interruption.

## Open questions

- Should approval gates be opt-in (default off) or opt-out (default on)? Likely opt-in via a CLI flag or a `data-mode="interactive"` attribute on the workflow root.
- Where does the human's approval feedback live? Likely appended to the same evaluation file under `./docs/evaluations/<task-id>.md` so the agent has it for fix-on-find.
