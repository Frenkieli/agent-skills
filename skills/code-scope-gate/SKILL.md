---
name: code-scope-gate
description: >-
  Pre-work scope triage for coding, debugging, refactoring, automation, planning, and review. Separate the actual request, candidate implementations, ambiguous intended scope, and boundary changes; delete unrequested surface and choose the next narrow action. Use silently for low-risk work; show a visible gate when the user asks for minimal scope or scope risk affects the next action. Use code-plan for full executable plans, code-review for diff/spec critique.
---

# Code Scope Gate

## Stance

Hunt unrequested implementation surface. Read the request adversarially, then default to deleting, deferring, reusing, or doing nothing before adding files, abstractions, or process. When an ambiguous item plausibly carries the user's intended outcome, preserve it as a clarification question or pause condition instead of treating ambiguity as proof it should be cut.

Ceiling: question → classify → clarify or delete → simplify. Stop there. Hand off rather than expand.

## The Three Moves

1. **Question** — name the actual outcome and its evidence; mark which items are requirements, candidate implementations, ambiguous intended scope, or boundary changes; name what must stay unchanged. Untethered candidate implementations get downgraded or dropped. Possibly intended requirements become clarification questions or pause conditions. Ask one narrow question only if a missing fact would change the path.
2. **Delete** — cut features not requested, hypothetical-future options, single-use abstractions, new deps/config/persistence/CI, adjacent refactors, and any public-behavior / API / schema / security / deployment / cross-module change without authorization. Do not cut an ambiguous intended capability only because it is under-specified; name the boundary or evidence needed to keep it. Keep an item only if removing it harms correctness, maintainability, verification, or the requested outcome.
3. **Simplify** — for what survived: smallest coherent file set, existing patterns and boundaries, boring direct code. Add abstraction only when it removes real duplication *now*.

## Boundary Rule

Public behavior, shared contracts, APIs, schemas, persistence, security, deployment, cross-module ownership = **Boundary Change**. Unauthorized → offer the best local alternative; if none exists, stop and ask.

## When To Show The Gate

Visible when: user asks for minimal scope; request mixes outcome with implementation; work could add surface (features, deps, persistence, schemas, workflows, APIs); a boundary change looks necessary or tempting.

Silent for: low-risk obvious work, typo fixes, single requested commands, authorized large rewrites, emergencies.

When `code-plan` owns the artifact, write the gate result into its `Scope` / `Non-goals` / `Proposed approach` / `Pause conditions` instead of emitting a separate gate.

## Output

```markdown
## Scope Gate

Actual request: [the real outcome]
Ambiguous intended scope: [plausible requirement needing clarification, or none]
Deletes: [what's cut, deferred, reused, or left out]
Smallest sufficient path: [smallest next action that satisfies the request]
Boundary: [none / authorized / unauthorized — local alternative or pause]
Verification: [concrete checks needed to trust the result]
Next: [proceed / hand off to code-plan|code-review|code-test-strategy / ask one question / stop]
```

After the gate, act only within scope. Don't slide into acceleration, automation, platform work, or adjacent refactors without explicit authorization.

Self-check before acting: does the final change introduce any file, dependency, abstraction, or boundary change not named in this gate? If so, it is out of scope until added to the gate or authorized.

## Common Traps

- "Useful later" → `Future`, not now.
- "Unclear, but possibly intended" → `Clarify`, not automatic delete.
- "Cleaner if generic" → only when the abstraction pays rent immediately.
- "While I'm here" → adjacent cleanup is a separate change.
- "Senior asked for it" → seniority isn't evidence; still needs owner and rationale.
- "Too small for a gate" → then the gate is tiny, not skipped.
