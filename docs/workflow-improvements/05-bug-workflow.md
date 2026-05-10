# 05 — Sibling `bug.md` workflow

## Problem

The current workflow is greenfield-shaped: idea → discovery → requirements → MVPs → plans → tasks → implementation → polish. Every phase assumes there is no existing code, no prior decisions, no production state.

After the app ships, the user finds a bug. They have two bad options today:

1. **Re-run the full workflow.** The workflow re-derives requirements, re-builds the MVP shape, re-plans, re-implements. This wastes time and may regress unrelated features.
2. **Fix the bug by hand.** Loses the ATDD discipline, the evaluation rubric, and the predictable shape that the workflow exists to provide.

Neither matches the actual shape of bug-fix work, which is small, targeted, regression-test-driven, and informed by the existing code rather than a blank slate.

## Proposed change

Add a sibling workflow file: `bug-fix-workflow.html`. Same article-per-task DOM structure, different phase set:

| Task | Phase | Goal |
|---|---|---|
| `B1` | Reproduction | Reproduce the bug locally. Capture the exact steps and observed-vs-expected behavior in `./docs/bugs/<bug-id>.md`. |
| `B2` | Failing test | Write the smallest acceptance test that fails because of the bug. Commit + push. |
| `B3` | Root cause | Identify the root cause. Document it in the bug file. No fix yet — the cause must be understood before patched. |
| `B4` | Fix | Implement the smallest change that turns the failing test green. Commit + push. |
| `B5` | Regression sweep | Run the full test suite. Any other failure is in scope (the fix must not regress). |
| `B6` | Evaluation | Run the same Implementation Evaluation Rubric against the changed files. Fix-on-find. |
| `B7` | Close-out | Update the bug file with the cause, fix, and test that pins it. |

Invocation: `/loop 1m bug-fix-workflow.html docs/bugs/<bug-id>.md`. The bug file replaces the idea file as the loop input.

## Tradeoff

- Two workflows to maintain instead of one. The Implementation Guidance is shared between them, which limits drift.
- The bug-fix workflow re-uses the evaluation rubric but does not re-run plan / task / MVP phases. This is correct for bug-fix shape but means the rubric must be checkable against a partial code change, not a full implementation.
- Bug triage (priority, severity, who-affects) is out of scope for the workflow. The workflow assumes a single bug-of-record; triage is upstream.

## Open questions

- Where does the bug file template live? Likely `./docs/bugs/_template.md` so a new bug file is a copy-and-fill.
- Does the bug workflow need its own phase-boundary approval gates (see [01](01-approval-gates.md))? Likely yes for B3 (root cause) — fixing without understanding the cause is the most common bug-fix anti-pattern.
- Feature-flag / hotfix variants — production-urgent fixes may need to bypass parts of the rubric. Probably out of scope for v1; revisit if it becomes a real need.
