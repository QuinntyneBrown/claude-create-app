# 02 — Parameterize the stack

## Problem

Every project produced by this workflow gets the same stack: .NET 8 + Angular + SQL Server Express + MediatR + FluentValidation + Material 3 + Playwright + JWT local-user auth. That is the right shape for an authenticated CRUD line-of-business app. It is the wrong shape for:

- A static marketing site (no backend, no auth, wants Astro/Next.js + a CMS).
- An ML / data demo (wants a Python backend, Streamlit or React frontend, no SQL).
- A CLI tool (wants a single repo, no frontend at all).
- A native mobile app (wants Expo/React Native or SwiftUI, not Angular).
- A real-time / collaborative app (wants WebSockets first-class, possibly an event store, possibly Postgres + LISTEN/NOTIFY).

Forcing every idea through the .NET + Angular mold either rejects the idea or produces a clumsy fit.

## Proposed change

Separate the workflow into two layers:

1. **Stack-agnostic meta-workflow.** The phase structure: Discovery → Requirements → Mocks → MVPs → Plans → Tasks → Slices → Polish. The evaluation cadence: every plan and every implementation gets a fix-on-find evaluation pass. The vertical-slice discipline. None of this is .NET-specific.

2. **Per-stack overlay.** The Backend, Frontend, Validation, Authentication, Library Structure, and parts of the Implementation Evaluation Rubric. Today these live inline in the single workflow file. Lift them into stack-specific files:

   - `stacks/dotnet-angular.md` — the current rules.
   - `stacks/nextjs-postgres.md` — Next.js + Prisma + Postgres + NextAuth + Playwright.
   - `stacks/python-fastapi-react.md` — FastAPI + SQLAlchemy + React + Vite + Pytest + Playwright.
   - `stacks/static-astro.md` — Astro + Markdown content collections + Tailwind + no backend.

The workflow runner picks a stack overlay based on the idea (a Discovery-phase decision) or based on an explicit `--stack` flag.

## Tradeoff

More upfront design — both the meta-workflow and at least two stack overlays must exist before this is useful. The single-file simplicity ("read this HTML, do what it says") becomes "read these two files, do what they say". The `data-task-id` registry needs a stack-id namespace to avoid collisions if stack overlays add their own tasks.

In return, the workflow stops being a .NET + Angular generator and becomes a generic full-stack-app generator parameterized by tech choice. That is the difference between a tool and a framework.

## Open questions

- Does the stack overlay get to add or remove phases? E.g., a static site has no backend MVP — does that mean MB1 and MB2 are skipped, or does the meta-workflow declare phases as optional?
- How does the evaluation rubric compose? Some rubric checks are stack-agnostic (no `TODO`, no empty bodies, ATDD discipline); others are stack-specific (Clean Architecture layering, MediatR CQS). The composition rule matters.
- Where does the stack decision happen — Discovery (D1/D2) or before Discovery? Argues for a `D0 — Pick the stack` task that the human approves.
