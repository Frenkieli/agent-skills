---
name: agent-team
description: >-
  Decompose a problem into multiple parallel sub-agents and cross-verify their findings, to cover or confirm something a single context cannot. Use whenever fanning out two or more non-conflicting sub-agents: parallel exploration of separate modules or sources, multi-lens review, exhaustive bug or issue hunts, or independent research that supports a plan or decision. Delegation is assumed authorized. The number of agents is not chosen up front — it falls out of the decomposition, and even a single question splits into multiple angles. Near miss: use agent-handoff for one indivisible delegation, and for the per-agent contract this skill reuses for every member. Do not use for a single atomic task, or to spawn agents whose work would overlap or edit shared state without disjoint ownership.
---

# Agent Team

Run many sub-agents against ONE decomposed problem: the main agent decomposes the problem, dispatches the slices in parallel, cross-verifies what comes back, and synthesizes a single result. A team exists to cover more ground than one context window can hold, or to confirm a result through independent perspectives — not to maximize agent count.

The main agent stays the **orchestrator and integrator**. It owns the goal, the decomposition, conflict detection across agents, and the final synthesis and verification. Sub-agents own bounded slices and return distilled evidence.

Delegation is assumed authorized; spawn freely. Each member's contract — packet, return format, stop condition, report-as-evidence — is owned by `agent-handoff`. This skill governs how the team is **shaped and verified**, and reuses that contract for every member.

## When To Fan Out, And How Wide

Fan out when a problem is worth decomposing into parallel coverage or independent verification: several modules, sources, or hypotheses to investigate; a multi-file or high-risk surface to review; an exhaustive hunt where one pass will miss the tail; or a decision that needs unbiased confirmation.

Scale the structure to the stakes — under-building a high-stakes question is the more common failure, and the one a conservative executor falls into, because a thin fan-out still *looks* like a team:

- "find anything off in here" → a few finders, single-pass verification.
- "thoroughly audit this / I need to be confident" → a larger finder pool, multi-vote adversarial verification, and a synthesis pass.
- An irreversible or high-stakes decision — architecture, a rewrite, a trade-off you will not revisit cheaply → a *multi-round* team, not one round of parallel angles. Field several **mutually-exclusive whole solutions** (the options you must choose between, not facets of one analysis), give **each its own adversaries** — a distinct set of lenses paid to kill that option specifically — then synthesize one ruling against real evidence. It is a matrix, each candidate × its own lenses, not a list, so its size is roughly positions × lenses and a serious decision fans out wide. Two under-builds pass as compliant here: splitting the question into research angles instead of competing candidates, and reviewing everything in one shared pass instead of gauntleting each candidate (see Cross-Verify).

Keep the work to one agent (`agent-handoff`) or local when it is a single indivisible unit, when the next step is a blocking critical-path decision the main agent must make now, or when the slices cannot be made disjoint and would edit shared state.

## Decompose First — Cardinality Comes From The Decomposition

The first act is always to decompose, and the agent count falls out of it. Do not pick a number up front and then invent work for it; derive the number from the structure of the problem.

A single question is still a team. Research it from multiple angles — source-of-truth behavior, regression surface, prior art, failure modes — and each angle is an agent. Splitting one question into angles *is* the work, which is why a lone question does not collapse into a single handoff.

When the size of the work is unknown, **scout first, then fan out** over what you find. One agent enumerates the work-list — the call sites, the affected files, the candidate sources — and the team's cardinality is that list, derived from data rather than guessed.

Partition by the context each agent needs, not by vibes:

- Good: separate modules or directories, separate sources, separate hypotheses, separate review lenses, separate angles on one question.
- Bad: several agents solving the same whole task, agents with overlapping edit rights, "research everything" with no boundary.

A cross-cutting concern is not a partition. When one invariant's definition and its enforcement land in different slices — a guard, a permission check, a validation or error contract, an auth boundary — splitting by module threads it across agents and leaves no owner for the whole of it. Give such a concern a single owner that follows it across slices, or keep an orchestrator checklist of what each agent reports it did NOT inspect. Otherwise it falls through the seam.

## Dispatch Topology

Dispatch independent packets in the same turn so they run concurrently; spawning them one at a time serializes work the partition already made parallel.

Default to a **pipeline**: each slice flows through its stages — find, then verify — independently, with no barrier between stages, so slice A can be in verification while slice B is still being found. Wall-clock is the slowest single chain, not the slowest stage summed across all slices.

Use a **barrier** — wait for an entire stage to finish before the next begins — only when the next stage genuinely needs the whole previous set: to dedup or merge across all findings before expensive verification, to early-exit when the total is zero, or when a stage compares findings against each other. "I need to flatten or filter the results first" is not a barrier reason — do that inside the next stage.

Each member's packet, return format, and stop condition follow `agent-handoff`.

## Cross-Verify — The Half That Buys Trust

Parallel coverage finds candidates; verification is what makes them trustworthy, and it is the line between a team and a parallel dump. A finding that no independent agent checked is a hypothesis, not a result.

Verify **adversarially**: for each material finding, spawn one or more independent agents prompted to *refute* it — default to "not real" unless the evidence forces otherwise — and keep the finding only if it survives. For a high-stakes claim, use several skeptics and require a majority to let it through.

When a finding can fail in more than one way, give each verifier a **distinct lens** — correctness, security, does-it-actually-reproduce — rather than several identical skeptics. Diverse lenses catch failure modes that redundant ones are all blind to together.

This discipline applies equally when the team's job is to **decide**, not to find. A candidate answer — an architecture, a recommendation, a chosen trade-off — is a hypothesis too. Field several mutually-exclusive positions and run *each* through its own distinct critique lenses — does the data model actually hold, does it survive scale, does it fit the product — before synthesizing one ruling. Lens each candidate separately, not the whole set in one shared review: an option only dies when an adversary is paid to kill it in particular. Covering a question from several angles is not the same as a choice that survived attack: angles inform, lenses refute.

## Coverage Patterns

Compose these as the task calls for; they are tools, not a fixed sequence:

- **Loop-until-dry**: for unknown-size discovery, keep spawning finders until K consecutive rounds surface nothing new — a fixed count misses the tail. Dedup each round against everything seen, not just what was already confirmed, or rejected findings keep reappearing.
- **Multi-modal sweep**: several agents each search a different way — by container, by content, by entity, by timeline — blind to each other, because one search angle never finds everything.
- **Completeness critic**: a final agent that asks "what is missing — a modality not run, a claim unverified, a slice not inspected?" Its answer is the next round of work.
- **Judge panel**: generate several independent attempts from different angles, score them with parallel judges, then synthesize from the winner while grafting the best of the runners-up.

Never cap coverage silently. If you stop at top-N, skip a retry, or sample, say what was dropped — a silent cap reads as "covered everything" when it did not.

## Integrate, Stall, Stop

Synthesize into ONE result — resolve conflicts between reports, dedup across them, and decide. A bundle of pasted sub-agent notes is not a synthesis.

While agents run, do not duplicate their work locally; do non-overlapping critical-path work or wait. Write a coordination record only when three or more agents run, the work spans rounds, or reports may conflict and need reconciliation — and keep it to assignments, status, and distilled evidence, never a second transcript.

Set limits before the loops start: max follow-up rounds, max verify/repair loops, max stall count. Stall signals — the same blocker twice, an agent expanding scope instead of converging, verifiers repeating generic feedback, reports contradicting with no new evidence being added. When stalled, narrow the task, ask one focused follow-up, or take it local; do not keep spawning equivalent agents. Stop when the requested work is covered and confirmed — not because capacity remains.

## Red Flags

- An agent count picked before the problem was decomposed.
- Slices with overlapping edit rights, or a cross-cutting invariant that no agent owns.
- Findings accepted without an independent refutation pass.
- A high-stakes decision split into research angles instead of competing candidates, or reviewed in one shared pass instead of each candidate gauntleted by its own adversaries — breadth standing in for a choice that survived attack.
- A barrier where a pipeline would do, leaving fast agents idle behind the slowest.
- Discovery stopped after a single round on an unknown-size problem.
- Reports pasted into the main context instead of synthesized into one result.
- The coordination record growing into a second transcript.
- Agents still spawning after the question is already answered.
