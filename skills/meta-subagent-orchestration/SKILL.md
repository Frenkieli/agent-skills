---
name: meta-subagent-orchestration
description: >-
  Orchestrate complex agent work with focused sub-agents while preserving the main agent's context. Use when a task may benefit from parallel sub-agents, delegated research, codebase exploration, independent implementation slices, background verification, or worker/explorer agents. Also use when the user asks to act as an orchestrator, use parallel agents, delegate to sub-agents, keep the main context clean, avoid duplicated agent work, or coordinate multiple agent sessions. Do not use for simple single-threaded tasks where delegation would add coordination overhead.
---

# Meta Subagent Orchestration

Use this skill to coordinate sub-agents without turning the main agent into a duplicate worker.

The goal is not to maximize the number of sub-agents. The goal is to protect the main agent's context and attention while still completing the user's requested work. Delegate bounded work that can run independently, then integrate the returned evidence.

This skill is tool-neutral. "Sub-agent" may mean a worker, explorer, background agent, forked agent, delegated session, agent thread, or any equivalent capability provided by the current coding or agent tool.

## Core Principle

The main agent is the orchestrator and integrator.

It owns:

- The user's goal and constraints.
- Task decomposition and sequencing.
- The critical path.
- Cross-agent conflict detection.
- Final decisions, edits, synthesis, and verification.

Sub-agents own:

- Focused investigation or execution slices.
- Local evidence gathering.
- Bounded implementation within explicit ownership.
- Reports that compress their work into actionable findings.

Do not delegate the same task and then immediately redo it locally. That defeats the purpose of context isolation and parallelism.

## When To Use

Use this skill when at least one of these is true:

- The task naturally decomposes into independent research, analysis, review, or implementation slices.
- A subtask may produce large logs, search results, file reads, or exploratory context that should not pollute the main conversation.
- Multiple files, modules, systems, sources, or hypotheses can be investigated in parallel.
- The user explicitly asks for sub-agents, parallel agents, orchestration, delegation, background workers, or context isolation.
- The task benefits from a separate verifier, reviewer, test runner, or source checker.

## When Not To Use

Do not use sub-agents when:

- The task is small enough that delegation adds more coordination cost than value.
- The next step is a blocking critical-path decision that the main agent must make now.
- The subtask is vague, open-ended, or lacks a clear stop condition.
- Sub-agents would need to edit overlapping files or shared state without explicit ownership.
- The work requires continuous shared context between agents rather than final reports.

If the work needs shared evolving knowledge, prefer a shared document, task board, or state file with explicit write rules before adding more agents.

## Workflow

### 1. Define The Critical Path

Before spawning any sub-agent, identify:

- What must be decided or done by the main agent next.
- Which work can proceed independently.
- Which work would block the next main-agent action.
- What evidence is needed before integration.

Keep the immediate blocker local. Delegate sidecar work only when the main agent can make progress without needing the result immediately.

### 2. Partition Work By Context, Not By Vibes

Split work according to the context each agent needs.

Good partitions:

- Separate modules or directories.
- Separate data sources or websites.
- Separate hypotheses.
- Separate review lenses, such as security, tests, API contract, performance, or documentation.
- Separate implementation areas with disjoint file ownership.

Bad partitions:

- Multiple agents solving the same whole task.
- Agents with overlapping edit rights.
- Agents asked to "research everything" or "find issues" with no boundary.
- Agents whose findings must constantly inform each other before either can proceed.

### 3. Write A Delegation Packet

Each sub-agent gets a concrete task packet. Include only the context it needs.

```markdown
Objective:
[One concrete outcome.]

Context:
[Minimal background, user constraints, relevant decisions.]

Scope:
- Owns: [files/modules/sources/questions]
- May inspect: [paths/sources]
- Must not edit: [paths/sources or "anything outside ownership"]

Execution rules:
- You are not alone in this workspace.
- Do not revert or overwrite work by others.
- Keep changes minimal and consistent with existing patterns.
- If blocked, report the blocker instead of expanding scope.

Verification:
[Commands, checks, source requirements, or "read-only investigation".]

Return format:
- Summary
- Evidence observed
- Files changed, if any
- Commands or sources checked
- Risks, conflicts, or unknowns
- Recommended next step

Stop condition:
[What counts as done, plus any time/depth limit.]
```

For code edits, assign disjoint ownership. If disjoint ownership is impossible, use read-only agents first and keep edits local to the main agent.

### 4. Enforce The Non-Overlap Rule

After delegating, the main agent must not start doing the same subtask locally while the sub-agent is still running.

Allowed while waiting:

- Work on a different part of the task.
- Prepare integration scaffolding that does not depend on the delegated result.
- Read small files needed for the critical path.
- Draft a verification plan.
- Handle unrelated blockers.
- Wait if the delegated result is the next blocker.

Not allowed while waiting:

- Re-run the delegated investigation from scratch.
- Read the same large file set or source set assigned to the sub-agent.
- Implement the same file/module assigned to a worker.
- Preemptively fact-check every delegated claim before the report exists.
- Spawn another agent with the same task because the first result has not arrived yet.

Verification is still required, but it happens after the sub-agent returns or at an explicit checkpoint. The orchestrator should verify the returned report, not consume context recreating the entire delegated process in parallel.

### 5. Receive Reports As Evidence

Treat sub-agent output as evidence, not truth.

Check:

- Did the sub-agent stay within scope?
- Did it provide concrete evidence, file paths, commands, or sources?
- Did it run the requested verification?
- Are there conflicts with other reports or known constraints?
- Did it make assumptions that affect the final decision?
- Did it introduce scope creep or boundary changes?

If a report is weak, ask for a focused follow-up. Do not silently redo the whole task unless the sub-agent failed and the work is necessary for completion.

### 6. Integrate And Stop

Integrate only what satisfies the user's requested work.

Before final response:

- Resolve conflicts between reports.
- Apply or refine worker changes if needed.
- Run the smallest meaningful final verification.
- State what changed, what was checked, and what remains unverified.
- Stop when the requested work is complete. Do not keep spawning agents just because capacity remains.

## Common Patterns

### Parallel Exploration

Use for codebase orientation, research, or diagnosis.

- Agent A investigates module or source A.
- Agent B investigates module or source B.
- Main agent continues on the critical path.
- Main agent synthesizes reports and decides the implementation or answer.

### Worker Slices

Use for implementation only when ownership is clean.

- Worker A owns one module or file set.
- Worker B owns a different module or file set.
- Main agent avoids editing those owned areas while workers run.
- Main agent reviews diffs, resolves integration, and verifies.

### Reviewer Or Verifier

Use when quality risk is high.

- Main or worker produces a result.
- Verifier checks against explicit criteria.
- Main agent uses verifier feedback to revise or accept.

The verifier must have a concrete rubric. "Check if this is good" is not enough.

## Red Flags

Watch for these failure modes:

- The main agent delegates, then immediately repeats the same exploration locally.
- Sub-agents receive broad tasks with no stop condition.
- Multiple workers edit the same file or shared contract.
- The orchestrator waits idly even though unrelated critical-path work is available.
- The orchestrator trusts a report with no evidence.
- The orchestrator asks for exhaustive verification when spot-checking the report would be enough.
- Reports are copied wholesale into the main context instead of summarized into decisions.

## Minimal Orchestration Note

When this skill triggers, keep the visible orchestration note short unless the user asks for a full plan:

```markdown
Orchestration:
- Main thread: [critical-path work kept local]
- Delegated: [sub-agent tasks and ownership]
- Non-overlap: [what the main agent will not duplicate while agents run]
- Integration: [how reports will be checked and merged]
```

