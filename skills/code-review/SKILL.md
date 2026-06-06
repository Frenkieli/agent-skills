---
name: code-review
description: >-
  Review concrete code plan drafts, specs, diffs, and implementation shapes. Use for code-review requests, serious code-plan design critique, and judging whether a proposed direction is sound. Prioritize solution direction, premise validity, logic chain, constraints, alternatives, design shape, contracts, tests, local fit, and actionable findings. Near miss: use code-plan to create or revise plans; use code-scope-gate for pre-spec scope shaping.
---

# Code Review

Perform an adversarial review of the change. Look for opportunities to **reduce layers** (pass-through wrappers, single-use abstractions), **remove complexity** (code-judo: restructurings that delete branches/modes/helpers, not rearrange them), and **increase reliability** (correctness, contract, state, concurrency, migration risk). For plan and spec reviews, judge the chosen direction before acceptance or test details: whether it solves the stated problem, rests on valid premises, assigns ownership to the right layer, fits observed constraints, and beats credible lower-scope or root-cause alternatives. Honor repo-wide policies (`AGENTS.md`, `CONTRIBUTING.md`, ADRs) as hard constraints. Verify what you can. Keep the original goal fixed; challenge the proposed route when evidence shows a better route to that same goal.

## Attribution

Distinguish **NEW** (introduced or made materially worse by this change) from **PRE-EXISTING** (already true on the base branch). Compare against base — read the unchanged file or check blame, not just the diff hunks. When the change replaces or deletes code, enumerate the guards and preconditions the old surface enforced — not just which capabilities it kept — and confirm each survives; an action that stays present but loses the predicate that gated it is invisible in a structural diff. Report PRE-EXISTING only when the change touches the same surface and it blocks the intended outcome, the change makes it worse, or the user asked for a broader audit. Tag every finding `[NEW]` or `[PRE-EXISTING]`; if uncertain, say so rather than defaulting to NEW.

## Plan-Direction Gate

Activate this gate when reviewing a plan, spec, proposed implementation approach, migration design, or any request asking whether an approach is right, wrong, best, optimal, or worth doing.

Required evidence before judging plan quality:

- Target outcome and explicit constraints from the prompt, repo evidence, linked issue, or local project rules.
- The plan's chosen strategy and ownership point: data source, state write, shared contract, call boundary, persistence/schema boundary, or presentation layer.
- A checked logic chain from problem → decision point/root cause → proposed change → intended effect.
- Credible alternatives within the same authorized goal, especially earlier fix points, smaller-scope routes, deletion of unnecessary layers, and existing local owners.
- A bounded optimality judgment: best-supported under observed constraints, adequate but suboptimal, wrong/unsupported, or unjudged without named evidence or a user decision.

Prohibited substitutes:

- Acceptance criteria, test coverage, rollout steps, milestone structure, extra documentation, or clearer wording do not satisfy this gate unless they also support the direction judgment.
- A plan that proves the proposed path can work does not satisfy this gate unless the review also checks whether it is the right path for the stated problem and constraints.
- "Could be cleaner" does not satisfy this gate without naming the violated invariant, extra complexity accepted, or better owner/fix point.

Incomplete behavior:

- If missing evidence materially affects the direction judgment, state the missing evidence as an open question and mark the direction unjudged rather than validating it indirectly.
- If the proposed plan is acceptable only as a compromise, state the invariant it sacrifices, the risk it accepts, the constraints it depends on, and the stop condition.

## Bar

Volume is failure. Each finding names a concrete surface, the impact, and the smallest correction. Drop anything that wouldn't change the merge decision or follow-up plan. Cap nits (style/naming only) at three — if you have more, the bar is too low.

Filter on severity and merge-relevance, not on your own confidence. Investigate fully, then decide what to report: surface a plausible correctness/contract/state issue even when you are unsure of it — flag the uncertainty and name what would confirm it, rather than dropping it silently.

Severity: **blocker** (correctness/contract/state), **raise** (real issue, follow-up acceptable), **nit** (small polish). Order by severity. Split atomic findings when surface or correction differs; merge only exact duplicates. If nothing meets the bar, say so and name residual risk.

For draft-plan reviews, corrections are plan changes (revised approach, added context, stronger non-goal, checkpoint, user decision).

## Lenses

Most reviews are inline — apply whichever lenses fit the change. When surfaces fan out or risk concentrates (multi-file, public API/CLI/schema/migration/persistence), dispatch focused sub-agents using the lens descriptions below as their prompts (consult `subagent-delegation` first). Sub-agents stay read-only, apply the attribution rule, and return concrete candidates with source pointers — silence is a valid output.

**Design shape.** Shallow interfaces, information leakage, tactical patches adding branches/modes/fallbacks without reducing underlying complexity, weak ownership boundaries, error handling that should be removed by design, layers that only forward calls. Use APOSD vocabulary ([references/aposd-complexity-review.md](references/aposd-complexity-review.md)) — every design finding names reader task + symptom (change amplification / cognitive load / unknown-unknown) + cause (dependency / obscurity). Reject "cleaner" / "more DRY" / "add comments" without that mapping.

**Contract surface.** Public/shared contracts: API, CLI, schemas, persisted state, generated artifacts, wrappers, migrations. New capability promises, silent rewrites/repairs/normalizations on parse, wrappers that reinterpret upstream contracts, missing collision checks on derived identities (paths/routes/cache keys/external refs), undocumented changes to behavior/imports/CLI output/event payloads/errors/timing, migrations that drop source baseline or fixture matrix, rewrites that keep an action but silently drop the precondition that gated it (permission / verification / risk-control / validation / enable-disable predicate). Don't invent compatibility requirements beyond observed contracts.

**Test validation.** Regression evidence. Behavior that could regress without coverage, tests that assert private state / call order / component names instead of public behavior, mocks of things that should be real, production-only seams added for testability, plans that prove the new path while leaving existing behavior unprotected. Don't report coverage gaps in untouched code.

**Implementation fit.** Fit with the local codebase. New helpers/wrappers/adapters that duplicate existing owners, single-use abstractions, wrong-layer logic, near-copy variants, broad find-and-replace that scatters one behavior, options/branches/files for future flexibility, calls to nonexistent or bypassed helpers. Don't propose adjacent refactors unless they are the smallest correction.

**Synthesis critic.** Dispatch when risk is high or evidence conflicts. Different role: challenges the draft findings, doesn't produce new candidates. Probes: evidence real, attribution correct against base, PRE-EXISTING justified, set clears the bar (drop low-value / excess nits), no bad duplicates/splits/severity, APOSD findings name reader-task/symptom/cause not slogans, severe sub-agent candidates not silently dropped, unreviewed surfaces acknowledged. Returns per challenge: target finding (or missing area), issue, evidence, action (keep / drop / retag / reword / reorder / verify / surface in judgment).

## Self-Review

Before final output, check:

- Could this review sound useful while only strengthening acceptance, tests, rollout, or wording without judging the plan direction? If yes, the review is incomplete for a plan/spec request.
- Does every direction-level judgment cite observed evidence or explicitly mark the missing evidence?
- Are direction, premise, ownership, or logic-chain findings ordered before acceptance/test findings of equal or lower severity?
- Does the review distinguish "best-supported under these constraints" from a theoretical global optimum?
- For changes that replace or delete code, did the review enumerate the guards/preconditions the old surface enforced and confirm each survives — not just that capabilities were kept?

## Output

User's primary language for prose; keep code symbols, paths, errors original. Return: findings (each prefixed `[NEW]`/`[PRE-EXISTING]` and severity-tagged) → open questions that materially change the decision → short overall judgment (note blocked/unreviewed surfaces). For plan/spec reviews, the findings section starts with direction, premise, ownership, and logic-chain issues when present; the overall judgment names the direction verdict: best-supported, adequate but suboptimal, wrong/unsupported, or unjudged. No edits, commits, or pushes without separate authorization.

## Stop Rules

For plan/spec reviews, finish only after the Plan-Direction Gate has a stated verdict, material open questions are named, and acceptance/test observations have not displaced a more important direction judgment. For diff/code reviews, finish only after attribution, severity, and residual risk are clear. If verification is unavailable or out of scope, state the next-best evidence used.
