---
name: code-goal-writer
description: >-
  Write complete, measurable Codex Goal contracts for coding work from user intent. Use when defining, clarifying, refining, or validating goals, /goal prompts, durable coding objectives, acceptance criteria, measurable outcomes, stop conditions, validation loops, Figma or design parity, behavior-preserving migrations, refactors, debugging, implementation tasks, or long-running code agent work. Produces intent, scope, non-goals, acceptance results, verification, checkpoints, pause conditions, and stop conditions before planning or execution.
---

# Code Goal Writer

## Goal

Produce a self-contained coding Goal contract that another agent can follow across a long-running engineering task without losing the user's intent, acceptance bar, boundaries, or stopping condition.

The contract emphasizes final acceptance results over implementation approach. It names what done means in observable, measurable terms, and it keeps design, planning, and execution as separate downstream work.

## Success Criteria

A strong Goal contract:

- States one durable objective rather than a backlog or broad project wish.
- Preserves the user's intent in the user's terms.
- Separates scope, non-goals, constraints, and allowed differences.
- Converts completion into measurable acceptance results with metric, threshold, data source, and scope.
- Names the context the agent must inspect before acting, such as files, docs, issues, Figma links, screenshots, logs, commands, or plans.
- Defines a validation loop, checkpoint evidence, stop condition, pause conditions, and compact progress-report format.
- Makes subjective quality bars evaluable through examples, rubrics, score thresholds, or human review gates.

## Evidence Budget

Start from the user's request and any artifacts they provided. Inspect local files, screenshots, Figma references, docs, issues, tests, logs, or existing plans only when they affect the goal's acceptance bar, validation loop, boundary, or stop condition.

Ask one narrow question when missing information would materially change acceptance. Otherwise proceed with a conservative assumption and label it in the contract.

High-impact missing facts include:

- the target artifact, route, component, module, repository, or file set
- the source of truth for expected behavior or visuals
- allowed differences from current behavior or the reference design
- required verification commands, environments, browsers, viewports, fixtures, or scores
- credentials, external side effects, destructive operations, deployment, or persistence changes

## Output Contract

When the user asks for a goal, produce this shape. Keep the `Ready Goal` self-contained enough to paste into `/goal` or a long-running task prompt.

```markdown
Ready Goal:
[One complete goal statement. Include the objective, boundaries, acceptance bar, validation loop, and stop condition.]

Intent:
- [Why the user wants this outcome, in the user's terms.]

Objective:
- [One durable objective.]

Scope:
- [Included surfaces, artifacts, states, routes, modules, or workflows.]

Non-goals:
- [Explicitly excluded work, behaviors, files, systems, or decisions.]

Required context:
- [Files, docs, links, screenshots, Figma frames, logs, tests, commands, plans, or issues to read first.]

Acceptance results:
- Result: [Observable outcome.]
  Metric: [What is measured.]
  Threshold: [Pass/fail boundary.]
  Data source / verification: [Command, screenshot diff, test, API response, rendered UI, log, artifact, or human review gate.]
  Scope: [Browsers, viewports, fixtures, states, locales, modules, or inputs.]

Allowed differences:
- [Changes explicitly permitted by the user or existing project contract.]

Validation loop:
- [Exact checks to run, when to run them, and how to react to failure.]

Checkpoints:
- [Milestones and proof required at each checkpoint.]

Stop condition:
- [Concrete state where the agent should stop because the goal is met.]

Pause conditions:
- [Conditions requiring user input, such as ambiguous product judgment, missing access, failed validation needing scope choice, destructive action, deployment, or boundary conflict.]

Progress report format:
- Current checkpoint:
- Changes made:
- Verification run:
- Result:
- Remaining work:
- Blockers:
```

For small goals, keep the same fields but collapse empty or obvious sections. Keep `Acceptance results`, `Validation loop`, and `Stop condition` explicit.

## Quantification Rules

Each acceptance result should be as measurable as the domain permits.

Prefer:

- `All existing tests pass with [command]` over `works correctly`.
- `No public API signatures, exported names, event payloads, or persisted formats change` over `compatible`.
- `Screenshot diff has 0 unapproved changed pixels after masking declared dynamic content` over `looks the same`.
- `Score reaches at least 0.90 on [eval command]` over `improves quality`.
- `No new console errors, type errors, lint errors, accessibility violations, or failed network requests in [scope]` over `polished`.

When exact automation is unavailable, define the manual acceptance evidence: who reviews, what artifact they inspect, which rubric they apply, and what result counts as pass.

## Figma And Visual Parity

When the user provides a Figma design, screenshot, mock, or visual reference, treat the expected result as near-100% visual parity unless the user gives a lower bar.

The Goal contract should capture:

- source of truth: Figma selection URL, frame name, screenshot path, or design artifact
- target surface: route, page, component, state, or viewport
- viewport matrix: design viewport plus required mobile, tablet, and desktop sizes
- state matrix: default, hover, focus, active, disabled, loading, empty, error, and responsive states that apply
- property parity: layout, spacing, typography, colors, borders, radius, backgrounds, shadows, icons, assets, cropping, and z-order
- pixel comparison: screenshot or visual-regression command, diff threshold, masks for dynamic content, and review rule for any remaining diff
- behavior text: interactions, validation, keyboard behavior, focus order, animations, and accessibility expectations that are not obvious from the static design

Default acceptance wording:

```markdown
Acceptance results:
- Result: The implemented UI matches the referenced Figma frame across the required viewport and state matrix.
  Metric: Pixel diff, property parity, and interaction parity.
  Threshold: 0 unapproved visual diffs; any nonzero diff must be either fixed or listed as an allowed difference with reason. Layout, colors, borders, backgrounds, typography, and assets match the reference source after declared dynamic content is masked.
  Data source / verification: [Playwright, browser screenshot comparison, Chromatic, Figma inspect, or project visual test command].
  Scope: [routes/components], [viewports], [states].
```

## Behavior-Preserving Migrations

When the user asks to migrate a component, module, framework, stack, or implementation from another project into the current project, treat behavior parity as the acceptance default. Adapting to current project style covers code organization, naming, imports, formatting, linting, framework conventions, and local helper usage; it does not expand permission to change observable behavior.

The Goal contract should capture:

- source implementation and target location
- public contracts: exports, props, inputs, outputs, errors, events, callbacks, accessibility roles, keyboard flows, API shapes, persisted data, side effects, and timing behavior
- fixture matrix: representative inputs, states, permissions, errors, loading cases, empty cases, and edge cases
- parity checks: existing tests, characterization tests, side-by-side output comparison, DOM state, screenshots, API requests, logs, or contract tests
- allowed differences: explicitly authorized behavior, style, dependency, or framework adaptation differences
- rollback or fallback expectation until parity checks pass

Default acceptance wording:

```markdown
Acceptance results:
- Result: The migrated component or module preserves all observable source behavior while conforming to current project conventions.
  Metric: Public contract parity, fixture parity, test parity, and allowed-difference review.
  Threshold: No user-visible behavior, public API, event payload, persisted format, accessibility behavior, keyboard flow, or error/loading behavior changes except items listed under Allowed differences.
  Data source / verification: [existing tests], [new characterization or parity tests], [manual smoke flow], [screenshot or DOM comparison], [build/lint/typecheck].
  Scope: [source module/component], [target module/component], [fixture matrix].
```

## Clarification Rules

Ask at most one question at a time. Ask only when the answer changes the goal's objective, acceptance threshold, validation method, authorization boundary, or stop condition.

Useful narrow questions:

- `Which Figma frame or selection URL is the source of truth?`
- `What viewport and state matrix must count for acceptance?`
- `What visual diff threshold should be treated as pass, or should every diff require review?`
- `Which source project path is the behavior baseline for this migration?`
- `Are any behavior differences allowed, or should parity be exact?`
- `Which command is the authoritative validation check?`

If the user says to proceed without the answer, write the conservative assumption into `Required context`, `Acceptance results`, or `Allowed differences`.

## Self-Review

Before returning the Goal contract, check:

- The contract has exactly one objective.
- Acceptance results are observable and include metric, threshold, data source, and scope where possible.
- Non-goals and allowed differences protect the user's boundary.
- Figma or screenshot goals include pixel, property, viewport, and state parity.
- Migration goals include public contract, fixture, test, and allowed-difference parity.
- The stop condition is concrete enough for an agent to stop without asking.
- Pause conditions cover ambiguity, missing access, external side effects, destructive changes, and failed validation requiring product judgment.

## Stop Rules

The skill is complete when the user has either an approvable Goal contract or a short list of blockers that materially prevent writing one.

The next phase is a separate action: setting `/goal`, planning, coding, writing documentation, running tests, or changing files requires the user's current request or an already active execution task.
