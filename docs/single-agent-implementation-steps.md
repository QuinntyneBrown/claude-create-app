# Single-Agent Implementation Steps

Input idea: `./docs/idea.md`

Goal: hand one agent an idea and a linear sequence of work that takes an authenticated .NET + Angular full-stack app from idea to deployed, tested software.

## Source Task Mapping

1. `D1 - Create HTML mocks` maps to linear steps 3 through 12.
2. `D2 - Evaluate HTML mocks` maps to linear steps 13 through 15.
3. `R1 - Create requirements (L1 + L2)` maps to linear steps 16 through 25.
4. `R2 - Evaluate requirements` maps to linear steps 26 through 28.
5. `MB1 - Create backend MVP` maps to linear steps 29 through 45.
6. `MB2 - Evaluate backend MVP` maps to linear steps 46 through 48.
7. `MF1 - Create frontend MVP` maps to linear steps 49 through 63.
8. `MF2 - Evaluate frontend MVP` maps to linear steps 64 through 66.
9. `BP1 - Plan backend implementation` maps to linear steps 67 through 69.
10. `BP2 - Evaluate backend plan` maps to linear steps 70 through 71.
11. `BT1 - Create vertically sliced backend tasks` maps to linear steps 72 through 74.
12. `BT2 - Evaluate backend tasks` maps to linear steps 75 through 76.
13. `BI1 - Implement each backend task with ATDD` maps to linear steps 77 through 85.
14. `FP1 - Plan frontend implementation` maps to linear steps 86 through 88.
15. `FP2 - Evaluate frontend plan` maps to linear steps 89 through 90.
16. `FT1 - Create vertically sliced frontend tasks` maps to linear steps 91 through 93.
17. `FT2 - Evaluate frontend tasks` maps to linear steps 94 through 95.
18. `FI1 - Implement each frontend task with ATDD (Playwright POM)` maps to linear steps 96 through 105.
19. `DP1 - Create Azure deployment workflow (cheapest plan)` maps to linear steps 106 through 110.
20. `TP1 - Create test plan` maps to linear steps 111 through 112.
21. `TP2 - Ensure app runs locally` maps to linear steps 113 through 115.
22. `TP3 - Run test plan 5 times, log bugs, fix bugs` maps to linear steps 116 through 120.
23. Final completion checks from the source workflow map to linear steps 121 through 128.

## Operating Rules

1. Work from the repo root.
2. Treat `./docs/idea.md` as the product brief. If the product idea is not already there, write it there before starting.
3. Use this document as a sequential checklist. Do not skip an evaluation gate.
4. Keep generated documentation under `./docs/`.
5. Keep backend source under `./backend/`.
6. Keep frontend source under `./frontend/`.
7. Put executable scripts under `./scripts/`, not under `./docs/`.
8. Put CI/CD workflows under `./.github/workflows/`.
9. After each creation step, run its paired evaluation step and fix every finding before proceeding.
10. For every evaluation step, append notes to `./docs/evaluations/<step-id>.md` with a `## Pass N - findings` section.
11. Use the fix-on-find rule: if an evaluation finds a problem, fix it, rerun the evaluation, and stop only after a full clean pass.
12. Use ATDD for implementation: write the acceptance test first, confirm it fails for the expected reason, implement, then make it pass.
13. Each acceptance test must include a trace comment naming the L2 requirement IDs it covers.
14. Do not leave stubs, TODOs, `NotImplementedException`, empty methods, hard-coded fake responses, debug `console.log` calls, or commented-out logic.
15. Optional integrations that the idea or requirements explicitly defer may use clearly named logging no-op services, and each one must be documented in the relevant evaluation notes.
16. Before moving to the next major phase, ensure the working tree is clean or intentionally contains only the current phase artifacts.

## Required Documentation Layout

1. Keep the product brief at `./docs/idea.md`.
2. Keep static mock HTML at `./docs/mocks/*.html`.
3. Keep mock screenshots at `./docs/mocks/screenshots/*.png`.
4. Keep the mock render script at `./docs/mocks/render.mjs`.
5. Keep requirements at `./docs/specs/L1.md` and `./docs/specs/L2.md`.
6. Keep backend and frontend plans at `./docs/plans/backend.md` and `./docs/plans/frontend.md`.
7. Keep vertical task lists at `./docs/plans/backend-tasks.md` and `./docs/plans/frontend-tasks.md`.
8. Keep evaluation reports under `./docs/evaluations/`.
9. Keep runbooks under `./docs/runbooks/`.
10. Keep the QA test plan at `./docs/qa/test-plan.md`.
11. Keep test execution bug logs under `./docs/bugs/`.

## Global Implementation Rules

1. Build a mobile-first web app that also works well on large screens.
2. Follow S.O.L.I.D. principles.
3. Prefer the smallest design that fully implements the requirements.
4. Implement every approved requirement completely before marking its task done.
5. Backend must use Clean Architecture with Api, Application, Domain, and Infrastructure separation.
6. Backend commands and queries must use MediatR.
7. Place backend feature code in vertical feature folders under `backend/src/<App>.Application/Features/<FeatureName>/`.
8. Use ASP.NET Core controllers for HTTP endpoints.
9. Use Entity Framework Core through an `IAppDbContext` abstraction.
10. Do not add repository or unit-of-work abstractions over EF Core.
11. Use Microsoft SQL Server.
12. Use SQL Server Express for local development with a default connection string shaped like `Server=.\SQLEXPRESS;Database=<App>;Trusted_Connection=True;TrustServerCertificate=True;`.
13. Commit the local development connection string in `appsettings.Development.json`.
14. Use FluentValidation for command and request validation.
15. Do not use DataAnnotations on commands, queries, or request DTOs.
16. Add a MediatR validation pipeline that maps validation failures to HTTP 400 `ValidationProblemDetails`.
17. Store local users in the application database.
18. Implement username and password sign-in that issues a signed JWT access token.
19. Store passwords only as salted hashes from Argon2id, PBKDF2, or bcrypt with adequate settings.
20. Validate JWT issuer, audience, signature, and expiration on every protected request.
21. Rate-limit, lock out, or throttle repeated failed sign-ins and log each security event.
22. Implement registration, sign-in, sign-out, password reset, profile management, account deletion, and RBAC.
23. Frontend must use Angular with Angular Material and Material 3 visual language.
24. Frontend must define design tokens for color and spacing.
25. Use BEM class naming in frontend HTML and SCSS.
26. Split every Angular component into `.ts`, `.html`, and `.scss` files.
27. Split the Angular workspace into an app plus `api`, `components`, and `domain` libraries.
28. Put backend-facing models and services in the `api` library.
29. Give every frontend service a matching `*.service.contract.ts` containing the TypeScript interface and injection token.
30. Put reusable presentation components in the `components` library.
31. Put components that consume the API layer in the `domain` library.
32. Enforce import direction: `components` imports nothing from `api` or `domain`; `domain` may import `api` and `components`; the app may import all three.
33. Use Playwright Page Object Model tests for important frontend and end-to-end behavior.

## Linear Steps

1. Read `./docs/idea.md` and extract the product name, primary users, core goals, primary entities, authenticated surfaces, unauthenticated surfaces, constraints, and any explicitly deferred integrations.
2. Create the documentation directories needed for the workflow: `docs/mocks`, `docs/mocks/screenshots`, `docs/specs`, `docs/plans`, `docs/evaluations`, `docs/runbooks`, `docs/qa`, and `docs/bugs`.
3. Create static HTML mocks under `./docs/mocks/`, one file per screen, using Material 3 visual conventions.
4. Include at least these mock screens: sign-in, sign-up, password reset, primary authenticated landing page, one list/detail or CRUD screen per primary entity, profile/settings, an empty state, and an error state.
5. Keep mock data hard-coded directly in the HTML. Do not use API calls, JSON loading, `fetch`, `XMLHttpRequest`, `localStorage`, or `sessionStorage`.
6. Use mobile-first responsive CSS for 360px, 768px, and 1440px widths.
7. Use BEM class names in every mock.
8. Add `./docs/mocks/index.html` linking to all mock screens.
9. Add `./docs/mocks/render.mjs` to render every mock page to PNG screenshots at 360px, 768px, and 1440px widths with Playwright.
10. Install Chromium for Playwright if needed with `npx --yes playwright install chromium`.
11. Run `node ./docs/mocks/render.mjs`.
12. Confirm each mock has `./docs/mocks/screenshots/<screen>.360.png`, `<screen>.768.png`, and `<screen>.1440.png`.
13. Evaluate the mocks and record findings in `./docs/evaluations/D2.md`.
14. During mock evaluation, verify current screenshots, Material 3 adherence, radical simplicity, primary-action clarity, responsive behavior, hard-coded data, BEM naming, idea coverage, mobile-first layout, and no-build browser opening.
15. Fix every mock evaluation finding, rerender screenshots, and repeat the evaluation until `D2.md` records a full clean pass.
16. Draft high-level requirements in `./docs/specs/L1.md`.
17. Number L1 requirements sequentially as `L1-001`, `L1-002`, and so on.
18. Include L1 requirements for product capabilities plus cross-cutting security, performance, accessibility, reliability, maintainability, observability, and responsive behavior.
19. Draft detailed requirements in `./docs/specs/L2.md`.
20. Number L2 requirements sequentially as `L2-001`, `L2-002`, and so on.
21. For every L2, include exactly one `Traces to: L1-xxx` field.
22. For every L2, include Given/When/Then acceptance criteria specific enough to write tests from.
23. Add L2 requirements for all authentication, authorization, validation, password hashing, JWT validation, failed sign-in protection, RBAC, profile, account deletion, security logging, and OWASP-relevant behavior.
24. Add L2 requirements for responsive behavior across XS, SM, MD, LG, and XL breakpoints where UI is involved.
25. Add a traceability summary at the bottom of `L2.md` mapping every L2 to its L1.
26. Evaluate requirements and record findings in `./docs/evaluations/R2.md`.
27. During requirements evaluation, verify L1/L2 traceability, idea coverage, mock coverage, implementation-guidance coverage, testability, responsive coverage, security explicitness, unambiguous language, and sequential unique IDs.
28. Fix every requirements evaluation finding and repeat until `R2.md` records a full clean pass.
29. Create a backend MVP under `./backend/`.
30. Use an old-style `.sln` file with backend projects under `backend/src` and backend tests under `backend/tests`.
31. Create Api, Application, Domain, and Infrastructure projects.
32. Wire MediatR in the Application layer.
33. Add one real sample command, handler, and FluentValidation validator in a per-feature folder under `Application/Features/<FeatureName>/`.
34. Add one real sample query and handler in a per-feature folder.
35. Add `IAppDbContext` and an EF Core `AppDbContext`.
36. Wire the local development database to SQLEXPRESS in `appsettings.Development.json`.
37. Add a real sample entity and migration.
38. Add one ASP.NET Core controller that calls MediatR.
39. Implement local username/password JWT authentication at MVP level.
40. Hash passwords with Argon2id, PBKDF2, or bcrypt.
41. Protect at least one sample endpoint with JWT validation.
42. Ensure one type per C# file across the backend.
43. Write `./docs/runbooks/backend.md` with the backend run command, SQLEXPRESS prerequisite, `dotnet ef database update` step, and architecture pattern notes.
44. Run `dotnet build` for the backend and fix errors or new warnings.
45. Verify the backend sample slice round-trips through HTTP, MediatR, EF Core, and local SQLEXPRESS.
46. Evaluate the backend MVP and record findings in `./docs/evaluations/MB2.md`.
47. During backend MVP evaluation, verify Clean Architecture, MediatR CQS, feature folders, no repository or unit-of-work, `IAppDbContext` usage, FluentValidation, SQLEXPRESS, JWT validation, password hashing, one-type-per-file, no stubs, and a complete sample slice.
48. Fix every backend MVP evaluation finding and repeat until `MB2.md` records a full clean pass.
49. Create a frontend MVP under `./frontend/`.
50. Create an Angular workspace with a main app plus `api`, `components`, and `domain` libraries.
51. Wire Angular Material with Material 3 visual styling and design tokens.
52. Create one backend-facing sample service and model in the `api` library.
53. Create the service contract and injection token in `*.service.contract.ts`.
54. Create one reusable presentation component in the `components` library.
55. Create one domain component in the `domain` library that consumes the API service through its contract.
56. Scaffold sign-in routing and one authenticated route.
57. Make the sample frontend flow call the backend MVP and render real backend data.
58. Use BEM class names and split every component into `.ts`, `.html`, and `.scss`.
59. Add one Playwright POM acceptance test for the MVP flow.
60. Write `./docs/runbooks/frontend.md` with the frontend run command and library structure.
61. Run `ng build` or the workspace equivalent and fix errors or new warnings.
62. Run the Playwright MVP test and make it pass.
63. Verify the frontend MVP visually matches accepted mocks at 360px, 768px, and 1440px.
64. Evaluate the frontend MVP and record findings in `./docs/evaluations/MF2.md`.
65. During frontend MVP evaluation, verify library boundaries, service contracts, Angular Material 3, design tokens, BEM, file splitting, Playwright POM test evidence, mock fidelity, no stubs, and radical simplicity.
66. Fix every frontend MVP evaluation finding and repeat until `MF2.md` records a full clean pass.
67. Plan the full backend implementation in `./docs/plans/backend.md`.
68. In the backend plan, cover every approved requirement, module, project, entity, DbContext change, command, query, validator, controller, auth flow, RBAC role, migration, and deferred integration.
69. Ensure every backend plan item maps to Clean Architecture, MediatR CQS, FluentValidation, local username/password JWT auth, SQL Server, and one-type-per-file guidance.
70. Evaluate the backend plan and record findings in `./docs/evaluations/BP2.md`.
71. Fix every backend plan evaluation finding and repeat until `BP2.md` records a full clean pass.
72. Break the approved backend plan into vertical backend tasks in `./docs/plans/backend-tasks.md`.
73. For each backend task, include an ID, title, requirements implemented, controller endpoint, command or query, validator, entity changes, migration, acceptance test, and applicable guidance rules.
74. Ensure each backend task delivers end-to-end value and is small enough to implement and evaluate in a few focused iterations.
75. Evaluate the backend task list and record findings in `./docs/evaluations/BT2.md`.
76. Fix every backend task-list evaluation finding and repeat until `BT2.md` records a full clean pass.
77. Implement each backend task one at a time.
78. For each backend slice, write the acceptance test first with L2 trace comments.
79. Run the backend acceptance test and confirm it fails for the expected missing behavior.
80. Implement the backend slice through controller, command or query, validator, handler, `IAppDbContext`, EF migration if needed, and database behavior.
81. Run the backend acceptance test, backend unit or integration tests, and `dotnet build`.
82. Evaluate the backend slice against the implementation rubric and append notes to `./docs/evaluations/BI1-<slice-id>.md`.
83. Fix every backend slice finding and repeat until the slice has a clean evaluation pass.
84. Mark the backend slice done in `./docs/plans/backend-tasks.md`.
85. Repeat steps 77 through 84 until every backend task is implemented and accepted.
86. Plan the full frontend implementation in `./docs/plans/frontend.md`.
87. In the frontend plan, cover every approved requirement, accepted mock screen, route, component, service, library boundary, design token, auth integration, Playwright POM test, and deferred integration.
88. Ensure every frontend plan artifact names its target library: `api`, `components`, `domain`, or app.
89. Evaluate the frontend plan and record findings in `./docs/evaluations/FP2.md`.
90. Fix every frontend plan evaluation finding and repeat until `FP2.md` records a full clean pass.
91. Break the approved frontend plan into vertical frontend tasks in `./docs/plans/frontend-tasks.md`.
92. For each frontend task, include an ID, title, requirements and mocks implemented, route, components and their libraries, service and contract, backend integration, Playwright POM test, and applicable guidance rules.
93. Ensure each frontend task names the correct library for every artifact it creates.
94. Evaluate the frontend task list and record findings in `./docs/evaluations/FT2.md`.
95. Fix every frontend task-list evaluation finding and repeat until `FT2.md` records a full clean pass.
96. Implement each frontend task one at a time.
97. For each frontend slice, write the Playwright POM acceptance test first with L2 trace comments.
98. Run the frontend acceptance test and confirm it fails for the expected missing behavior.
99. Implement the frontend slice with routes, components in the correct libraries, API services in `api`, service contracts, backend integration, Material 3 components, BEM classes, design tokens, and responsive layout.
100. Run the frontend acceptance test and `ng build` or the workspace equivalent.
101. Verify the slice visually against its accepted mock at 360px, 768px, and 1440px.
102. Evaluate the frontend slice against the implementation rubric and append notes to `./docs/evaluations/FI1-<slice-id>.md`.
103. Fix every frontend slice finding and repeat until the slice has a clean evaluation pass.
104. Mark the frontend slice done in `./docs/plans/frontend-tasks.md`.
105. Repeat steps 96 through 104 until every frontend task is implemented and accepted.
106. Add Azure deployment automation on the cheapest viable plan.
107. Add the CI/CD workflow under `./.github/workflows/`.
108. Add the Azure CLI provisioning script under `./scripts/`.
109. Write `./docs/runbooks/deploy.md` covering required Azure resources, provisioning command, environment variables, secrets, cheapest-plan SKU choices, deployment steps, and rollback procedure.
110. Verify a push to the deploy branch provisions and deploys successfully.
111. Create `./docs/qa/test-plan.md`.
112. In the test plan, add at least one scenario for every L2 requirement plus regression and edge-case scenarios.
113. Create `./docs/runbooks/local.md`.
114. In the local runbook, document the single command and prerequisites required to start the whole app on a fresh checkout.
115. Verify the backend and frontend start cleanly from the documented local command.
116. Run the full test plan pass 1, log bugs in `./docs/bugs/pass-1.md`, fix them, and rerun pass 1 until clean.
117. Run the full test plan pass 2, log bugs in `./docs/bugs/pass-2.md`, fix them, and rerun pass 2 until clean.
118. Run the full test plan pass 3, log bugs in `./docs/bugs/pass-3.md`, fix them, and rerun pass 3 until clean.
119. Run the full test plan pass 4, log bugs in `./docs/bugs/pass-4.md`, fix them, and rerun pass 4 until clean.
120. Run the full test plan pass 5, log bugs in `./docs/bugs/pass-5.md`, fix them, and rerun pass 5 until clean.
121. Run final backend verification: `dotnet build`, backend tests, migrations against SQLEXPRESS, and backend run command.
122. Run final frontend verification: `ng build`, Playwright tests, and frontend run command.
123. Run final local verification from `./docs/runbooks/local.md` on a fresh checkout or clean local state.
124. Confirm deployment instructions in `./docs/runbooks/deploy.md` can recreate the environment from scratch.
125. Confirm no implementation files contain TODOs, stubs, debug logs, `NotImplementedException`, empty work methods, hard-coded fake responses, or library-boundary violations.
126. Confirm every evaluation report ends with a clean pass.
127. Confirm every L2 requirement is covered by at least one acceptance test and at least one QA test-plan scenario.
128. Confirm the app is deployed, locally runnable, documented, tested, and aligned with the original idea.
