# Workflow Improvements

Candidate improvements to the autonomous workflow described in `dotnet-angular-authenticated-full-stack-workflow.html`. Each improvement is recorded as its own document with the problem, the proposed change, and the main tradeoff. Nothing here has been adopted yet — these are decision inputs, not decisions.

## Context

The workflow is currently invoked as `/loop 1m <workflow.html> <idea.md>`. Every minute the loop fires, picks the first ready task, executes it against the Implementation Guidance, and (for evaluation tasks) applies fix-on-find. End-to-end this produces a tested, working .NET + Angular full-stack app from an idea description.

This directory captures levers for tuning that pipeline and one alternative approach to the whole structure.

## Improvements (incremental)

1. [Approval gates at phase boundaries](01-approval-gates.md) — pause between phases so human intent doesn't drift unnoticed through downstream tasks.
2. [Parameterize the stack](02-parameterize-stack.md) — separate the stack-agnostic phase structure from the .NET/Angular-specific rules so the workflow can target other tech.
3. [Cache the MVPs](03-cache-mvps.md) — commit a `templates/` scaffold so MB1/MF1 stop re-creating the same shape every run.
4. [Real feedback loop from the running app](04-app-feedback-loop.md) — boot the stack, screenshot at responsive breakpoints, and audit against the design mocks.
5. [Sibling `bug.md` workflow](05-bug-workflow.md) — a short reproduce → failing test → fix → regression workflow for post-ship bugs.

## Alternative approach (structural)

6. [Spec-generated workflow](06-spec-generated-workflow.md) — stop hand-maintaining 22 `<article>` elements; render the workflow HTML from a small declarative spec.

## Related

- [`../workflow-decision-trees/README.md`](../workflow-decision-trees/README.md) — how the agent picks the next task and where the workflow file's state model breaks down at high iteration counts.
