# 03 — Cache the MVPs

## Problem

MB1 and MF1 produce essentially the same scaffold every run. By design — the MVP is the *reference shape* the rest of the project is built against, and the Implementation Guidance pins it down precisely (Clean Architecture project split, MediatR wiring, sample command/query, `IAppDbContext`, JWT middleware, per-feature folders, Angular workspace with `api` / `components` / `domain` libraries, Material 3, BEM, one-type-per-file, one Playwright POM test).

Running MB1 + MF1 from scratch every time costs:

- Wall time — a non-trivial chunk of total run time.
- Variance — small drift in scaffolding choices ("did the MVP wire DI this way last time?") that the MB2 / MF2 evaluation pass then has to correct.
- Token budget — re-deriving a known-good shape from prose every run.

## Proposed change

Commit a `templates/` directory containing the accepted MVP shape:

```
templates/
  backend/                    # MB1's "accepted" output
    src/
      App.Api/
      App.Application/
        Features/
          Sample/
            CreateSample/
            GetSampleById/
        Behaviors/
        Common/
      App.Domain/
      App.Infrastructure/
    tests/
    appsettings.Development.json     # SQLEXPRESS connection string
  frontend/                   # MF1's "accepted" output
    apps/
      web/
    libs/
      api/
      components/
      domain/
    playwright.config.ts
```

MB1 becomes: `cp -R templates/backend/. backend/ && rename App → <ProjectName>`. MF1 becomes the equivalent for `frontend/`. MB2 / MF2 still run — they catch any drift that has crept into the templates since they were last regenerated.

A periodic re-generation task (every N runs, or on `Implementation Guidance` change) re-runs MB1 / MF1 *into* `templates/` to pick up new guidance.

## Tradeoff

The templates drift from the latest guidance unless they are regenerated. If the guidance gains a new rule and the template hasn't been re-generated, MB2 will catch it on the next real run (good) but the agent may apply the fix to the project copy without backporting to the template (bad — the next run pays the same cost again).

The template directory needs a `LAST_REGENERATED` marker (the guidance version it was generated against) so the workflow can detect staleness and trigger regeneration.

There's also a placeholder-substitution problem: `App.Api`, `App.Application`, etc. need to become `Acme.Api`, `Acme.Application`, etc. on copy. Either commit the templates with a literal placeholder string and `sed` it on copy, or keep the literal `App` and rename only project-file `RootNamespace` entries.

## Open questions

- Does the template live in this repo or in a separate `claude-create-app-templates` repo? Keeping it here makes the workflow self-contained; splitting it makes the template reusable across multiple workflow variants.
- How is regeneration triggered? Manual command, scheduled, or automatic on guidance hash change?
