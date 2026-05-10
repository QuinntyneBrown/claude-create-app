# 04 — Real feedback loop from the running app

## Problem

Today the workflow's verification is:

- `dotnet build` clean.
- `ng build` clean.
- Unit tests pass.
- Playwright POM tests pass.
- Implementation Evaluation Rubric clean.

All of these verify that the code is well-structured and the tests pass. **None of them verify that the app looks right.** A Playwright test asserting `expect(page.getByRole('button', { name: 'Save' })).toBeVisible()` passes whether the Save button is the right color, the right size, in the right place, on the right background, with the right typography, at the right responsive breakpoint.

The mock files under `./docs/mocks/` are the design-of-record. Today they are an input to FP1 / FT1 / FI1 (planning and slice tasks reference them) but they are never compared against the running output.

## Proposed change

Add a TP-phase task — call it `TP-UI` — that:

1. Boots the backend MVP and frontend MVP locally.
2. Walks every route in the routing plan.
3. Captures screenshots at three responsive breakpoints (360 / 768 / 1440 px).
4. Compares each screenshot against the corresponding mock in `./docs/mocks/`.
5. Records deviations under `./docs/evaluations/TP-UI.md`.
6. Applies fix-on-find — every deviation becomes a frontend slice fix.
7. Re-evaluates until clean.

The `ui-audit` skill (already available in this environment) is the engine for steps 3–5. The TP-UI task is a thin wrapper that supplies the route list and the mock directory.

## Tradeoff

- Wall time — UI audit + fix-on-find can iterate several times. Real verification is slower than nominal verification.
- Infrastructure — the task needs a way to boot both the backend and frontend reliably in headless mode. Today the runbooks (`./docs/runbooks/backend.md`, `./docs/runbooks/frontend.md`) document the run commands; TP-UI would consume those.
- False positives — pixel comparison against hand-drawn mocks is noisy. The audit should be tolerant of intentional rendering differences (font hinting, anti-aliasing) and strict on layout / color / spacing.
- The mocks themselves become load-bearing. If the mocks are wrong, the audit dutifully drives the UI to match them. This raises the importance of MD1 / MD2 (mock approval).

## Open questions

- Does TP-UI run once at the end, or every N implementation slices? End-of-pipeline is cheaper but fixes ship later; per-slice is more responsive but multiplies the cost.
- Authentication — many screens require a signed-in session. The audit needs a known test user and a way to sign in before walking authenticated routes.
- Does the audit also exercise interactive states (hover, focus, loading, error, empty)? Static-screen comparison is the v1; interactive states are a v2 expansion.
