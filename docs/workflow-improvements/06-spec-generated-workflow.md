# 06 — Spec-generated workflow (alternative approach)

## Problem

The workflow today is a single hand-maintained HTML file with 22 `<article>` elements. Every change — new phase, new evaluation check, stack tweak — is a hand edit. This is fine while the workflow is one file targeting one stack, but it scales poorly:

- **Adding a stack** ([02](02-parameterize-stack.md)) means duplicating most of the file and editing the duplicate in 30 places. The next stack means duplicating again.
- **Adding approval gates** ([01](01-approval-gates.md)) means touching every phase-boundary article.
- **Adding a sibling workflow** ([05](05-bug-workflow.md)) means a second hand-maintained file with shared structure that will drift from the first.

When the workflow is the *output* of a maintenance process rather than the *input*, the right tool is generation, not editing.

## Proposed change

Replace the hand-maintained HTML with a small declarative spec that renders to HTML.

A sketch of `workflow.spec.yaml`:

```yaml
workflow:
  name: dotnet-angular-authenticated-full-stack
  invocation: /loop 1m {workflow} {idea}
  stack: dotnet-angular

phases:
  - id: discovery
    tasks: [D1, D2]
  - id: requirements
    tasks: [R1, R2]
    gate: approval        # see improvement 01
  - id: mvp
    parallel:
      backend: [MB1, MB2]
      frontend: [MF1, MF2]
    gate: approval
  - id: backend-impl
    tasks: [BP1, BP2, BT1, BT2, BI1]
    parallel-with: frontend-impl
  - id: frontend-impl
    tasks: [FP1, FP2, FT1, FT2, FI1]
    parallel-with: backend-impl
  - id: deploy
    tasks: [DP1]
  - id: polish
    tasks: [TP1, TP2, TP3, TP-UI]    # see improvement 04

tasks:
  D1:
    title: Discovery interview
    inputs: [idea]
    deliverable: ./docs/discovery/D1.md
    skill: discovery
  # ... etc

guidance:
  general: ./guidance/general.md
  stack:   ./stacks/dotnet-angular.md      # see improvement 02

evaluation_rubric:
  general: ./guidance/rubric-general.md
  stack:   ./stacks/dotnet-angular-rubric.md
```

A small renderer (Node script, ~200 lines) walks the spec and emits the same `<article data-task-id="…" data-status="…">` HTML the loop already consumes. The runtime contract with the loop does not change — only the source of truth does.

## Tradeoff

- **Indirection.** Today, "what does BT1 do?" is a Ctrl-F in one HTML file. After this change it is "find BT1 in the spec, follow its references to general guidance and stack guidance, then look at the renderer if the output looks wrong." More files, more layers.
- **Renderer is now load-bearing.** A bug in the renderer breaks every workflow run. The renderer needs its own tests.
- **Hand-tuning is harder.** When you want to make BI1 slightly different from FI1, the spec has to support that variation — either with per-task overrides or by giving up on the abstraction. The most common abstraction failure mode is "the spec couldn't express what I wanted, so I bypassed it" → drift.
- **Worth it only at scale.** With one workflow and one stack, the hand-maintained HTML is fine. The break-even is roughly two stacks or two workflow variants (greenfield + bug-fix).

## Decision criterion

Adopt this if and only if at least two of the following are true:

- Improvement [02 — Parameterize the stack](02-parameterize-stack.md) is being adopted.
- Improvement [05 — Sibling `bug.md` workflow](05-bug-workflow.md) is being adopted.
- The Implementation Guidance has been edited five or more times in a way that required parallel edits across multiple workflow files.

Otherwise, keep the hand-maintained HTML — it is the simplest thing that works.

## Open questions

- Renderer language — Node (matches the frontend tooling), Python (matches the broader scripting), or .NET (matches the backend stack)? Probably Node, because the output is HTML and the build is already in the JS/TS ecosystem.
- Does the spec also drive the README and docs index, or just the workflow HTML? Generating multiple artifacts from one spec is the strongest version of the argument; generating only the HTML is the weakest.
