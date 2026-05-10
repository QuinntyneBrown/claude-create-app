# idea.md

This document defines a spec for an **HTML workboard** that can be used with a looped Claude interaction such as:

`/loop 1m [path-to-workboard.html] [path-to-idea.md]`

The goal is to let one or more agents repeatedly open the same HTML file, claim the next available task, complete it, and update the board until an app moves from idea to shipping.

## 1. What the HTML document must do

The HTML document should act as a machine-readable and human-readable queue of work.

It should:

- expose the current product idea and constraints
- break work into small tasks with unique ids
- show dependencies so agents know what is blocked
- show which tasks can be done in parallel
- identify the next highest-priority unblocked task
- tell the agent which skill or workflow to use
- require evaluation passes after major deliverables
- keep enough status history that multiple agents can safely coordinate

## 2. Recommended HTML structure

Use simple semantic HTML so Claude can read and update it reliably.

Recommended top-level sections:

- `header`: app name, mission, status summary
- `section#context`: product idea, users, goals, constraints
- `section#operating-rules`: how agents should claim and close tasks
- `section#phases`: ordered phases from idea to shipping
- `section#task-board`: all task cards
- `section#decision-log`: major decisions and open questions

Each task should be a standalone element, for example an `article`, with attributes and fields such as:

- `data-task-id`
- `data-phase`
- `data-status` (`todo`, `ready`, `in_progress`, `blocked`, `done`)
- `data-owner`
- `data-parallel-group`
- `data-depends-on`
- `data-skill`
- `data-evaluation-count`

Each task body should include:

- title
- goal
- inputs
- deliverable
- parallelization notes
- completion criteria
- evaluation instructions

## 3. Agent operating rules

Agents should use the workboard with the following rules:

1. Read the full board before editing.
2. Prefer the highest-priority task with `data-status="ready"` and no unmet dependencies.
3. If multiple tasks are ready, choose one whose `data-owner` is empty.
4. Claim the task by setting `data-status="in_progress"` and `data-owner` to the current agent.
5. Do not take a task whose dependencies are incomplete.
6. Use `data-parallel-group` to see what can run concurrently with other tasks.
7. When finished, update artifacts, mark the task `done`, and promote newly unblocked tasks to `ready`.
8. If the task requires evaluation, create or update the paired evaluation task and ensure it completes the required number of passes.

## 4. Parallel work model

To support multiple agents, tasks should be:

- small enough to finish in a single loop when possible
- explicit about dependencies
- grouped by parallel lanes
- followed by evaluation gates before downstream work begins

Use these conventions:

- `parallel-group="design"` for UI and mock work
- `parallel-group="requirements"` for requirements and architecture work
- `parallel-group="implementation"` for coding tasks that do not overlap
- `parallel-group="quality"` for docs, CI/CD, tests, linting, and release readiness

Tasks in the same phase may run in parallel when:

- they touch different deliverables
- they do not depend on each other
- their completion criteria do not conflict

## 5. Suggested end-to-end phase flow

The board should usually include phases like these:

1. **Design seed**
   - Build initial HTML mocks using the frontend designer skill.
2. **Design evaluation**
   - Review and improve the HTML mocks for at least 5 passes.
3. **Requirements**
   - Produce product requirements using design artifacts, the idea doc, and technical guidance docs.
4. **Requirements evaluation**
   - Review and improve requirements for at least 5 passes.
5. **Architecture and delivery planning**
   - Define technical plan, repo structure, milestones, and risks.
6. **Implementation**
   - Deliver features in small dependency-aware slices.
7. **Quality**
   - README, CI/CD workflow, linting, test plan, test execution, bug fixing.
8. **Release readiness**
   - Final verification, packaging, and shipping checklist.

Major deliverables should be followed by an evaluation task with a minimum of 5 review/improvement passes.

## 6. Authoring guidance

When authoring the HTML workboard:

- make tasks concrete and action-oriented
- prefer one deliverable per task
- write completion criteria that can be checked from files or outputs
- separate creation tasks from evaluation tasks
- mark evaluation tasks with a required pass count
- make parallel-safe work explicit instead of implicit
- keep the board current so the next agent can start immediately

Good task title examples:

- `Create initial landing-page HTML mocks`
- `Evaluate mock set pass 3 of 5`
- `Draft product requirements from idea and approved mocks`
- `Create CI workflow for lint, test, and build`

## 7. How agents should interpret `/loop`

Given:

`/loop 1m [path-to-workboard.html] [path-to-idea.md]`

the agent should:

1. open the HTML workboard first
2. use this document as policy and authoring guidance
3. claim the next ready task
4. perform only that task and any directly required board updates
5. stop with the board in a state that lets the next loop continue cleanly

## 8. Sample HTML workboard

Below is a sample HTML document showing the intended pattern.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Complex App Delivery Board</title>
  </head>
  <body>
    <header>
      <h1>Complex App Delivery Board</h1>
      <p><strong>Mission:</strong> Build a complex app from idea to shipping using looping Claude agents.</p>
      <p><strong>Loop command:</strong> /loop 1m ./docs/workboard.html ./docs/idea.md</p>
    </header>

    <section id="context">
      <h2>Context</h2>
      <p><strong>Product idea:</strong> A collaborative planning app for small software teams.</p>
      <p><strong>Primary users:</strong> founders, designers, engineers, QA.</p>
      <p><strong>Constraints:</strong> clear documentation, incremental delivery, release readiness.</p>
    </section>

    <section id="operating-rules">
      <h2>Operating Rules</h2>
      <ol>
        <li>Always claim a ready unowned task before making changes.</li>
        <li>Do not start blocked work.</li>
        <li>After completing a task, update statuses for newly unblocked tasks.</li>
        <li>Evaluation tasks must run 5 passes minimum.</li>
      </ol>
    </section>

    <section id="task-board">
      <h2>Task Board</h2>

      <article
        data-task-id="D1"
        data-phase="design-seed"
        data-status="ready"
        data-owner=""
        data-parallel-group="design"
        data-depends-on=""
        data-skill="frontend designer"
        data-evaluation-count="0"
      >
        <h3>D1 - Build initial HTML mocks</h3>
        <p><strong>Goal:</strong> Create the first set of HTML mocks for the main product flows.</p>
        <p><strong>Inputs:</strong> idea.md</p>
        <p><strong>Deliverable:</strong> HTML mock files checked into the repo.</p>
        <p><strong>Parallel:</strong> Can run alone. Unlocks design evaluation.</p>
        <p><strong>Done when:</strong> core screens exist and are reviewable.</p>
      </article>

      <article
        data-task-id="D2"
        data-phase="design-evaluation"
        data-status="blocked"
        data-owner=""
        data-parallel-group="design"
        data-depends-on="D1"
        data-skill="evaluation"
        data-evaluation-count="5"
      >
        <h3>D2 - Evaluate HTML mocks for 5 passes</h3>
        <p><strong>Goal:</strong> Review and improve the initial mock set over five iterations.</p>
        <p><strong>Inputs:</strong> D1 outputs, idea.md</p>
        <p><strong>Deliverable:</strong> revised HTML mocks plus recorded pass count.</p>
        <p><strong>Parallel:</strong> No downstream task starts until complete.</p>
        <p><strong>Done when:</strong> passes 1 through 5 are complete and final mocks are accepted.</p>
      </article>

      <article
        data-task-id="R1"
        data-phase="requirements"
        data-status="blocked"
        data-owner=""
        data-parallel-group="requirements"
        data-depends-on="D2"
        data-skill="requirements"
        data-evaluation-count="0"
      >
        <h3>R1 - Create requirements from idea and approved design</h3>
        <p><strong>Goal:</strong> Produce product requirements using the idea doc, design outputs, and technical guidance docs.</p>
        <p><strong>Inputs:</strong> idea.md, approved mocks, technical guidance docs</p>
        <p><strong>Deliverable:</strong> requirements document in the repo.</p>
        <p><strong>Parallel:</strong> Can unlock architecture and test-planning tasks later.</p>
        <p><strong>Done when:</strong> scope, features, constraints, and acceptance criteria are documented.</p>
      </article>

      <article
        data-task-id="R2"
        data-phase="requirements-evaluation"
        data-status="blocked"
        data-owner=""
        data-parallel-group="requirements"
        data-depends-on="R1"
        data-skill="evaluation"
        data-evaluation-count="5"
      >
        <h3>R2 - Evaluate requirements for 5 passes</h3>
        <p><strong>Goal:</strong> Review and improve requirements over five iterations.</p>
        <p><strong>Inputs:</strong> R1 outputs, idea.md, mocks</p>
        <p><strong>Deliverable:</strong> approved requirements document.</p>
        <p><strong>Parallel:</strong> Must finish before implementation planning.</p>
        <p><strong>Done when:</strong> all five passes are completed and open issues are resolved.</p>
      </article>

      <article
        data-task-id="Q1"
        data-phase="quality"
        data-status="blocked"
        data-owner=""
        data-parallel-group="quality"
        data-depends-on="R2"
        data-skill="delivery"
        data-evaluation-count="0"
      >
        <h3>Q1 - Prepare repo for shipping</h3>
        <p><strong>Goal:</strong> Create README, CI/CD workflow, lint setup, test plan, test execution, and release checklist.</p>
        <p><strong>Inputs:</strong> approved requirements and implemented features</p>
        <p><strong>Deliverable:</strong> repo is documented, validated, and ready to ship.</p>
        <p><strong>Parallel:</strong> Split into sub-tasks where no file ownership conflicts exist.</p>
        <p><strong>Done when:</strong> docs, CI/CD, linting, tests, and release checks are complete.</p>
      </article>
    </section>

    <section id="decision-log">
      <h2>Decision Log</h2>
      <ul>
        <li>Use evaluation gates after major deliverables.</li>
        <li>Prefer small tasks that a single loop can finish.</li>
        <li>Keep dependency data current so multiple agents can coordinate safely.</li>
      </ul>
    </section>
  </body>
</html>
```

## 9. Minimum checklist for future workboards

Before using a workboard in a loop, confirm it has:

- a clear app idea and constraints
- task ids and statuses
- dependency tracking
- explicit parallelization notes
- required skill per task where relevant
- evaluation tasks with at least 5 passes after major milestones
- endgame tasks for README, CI/CD, linting, test plan, testing, and shipping readiness
