---
name: context-engineering
description: >-
  Route durable rules and context to the right layer — task, project, skill, tooling, hooks, MCP, or global — and produce the smallest directly applicable edit. Use for global rules files (~/.claude/CLAUDE.md, global AGENTS.md), repo-local AGENTS.md/CLAUDE.md, task context packs, hook placement (Codex/Claude Code settings.json), collaboration friction diagnosis, and rule-placement decisions.
---

# Context Engineering

Place every durable rule or context fragment at the lowest layer that holds it — task < project < skill / tooling / hooks / MCP < global — and produce the smallest directly applicable edit. Preserve the user's requested artifact and boundary; do not edit adjacent layers without authorization.

For "do/check/inject X on event Y" behaviors (pre/post tool, prompt-submit, stop, session-start), route to hooks in `settings.json` — the harness executes hooks; text-layer rules only describe behavior and the model may skip them.

Reject rules that are task-local, already enforceable by lint/test/CI/hooks, duplicate model defaults, or describe a specialized workflow that belongs in a skill. Use absolute words (`must`, `never`, `always`, `only`) only for safety, irreversible actions, exact output contracts, or tool syntax.

## Evidence

Read the target artifact first — the rules file, candidate text, or friction evidence. When asserting a project convention, cite one existing file or command. Fetch external prompt guidance only when the user asks to align with it. Stop reading once classification and edit text are supported.

## Output

1. Classification per candidate, with chosen layer and reason.
2. Concrete edit text or context block for the chosen layer.
3. Items routed elsewhere, named by correct layer.
4. Evidence read or evidence gap.

Omit empty sections. Stop once classification, edit text, and routing are clear. Ask one narrow question only when missing information would change the layer, authorization boundary, or artifact.
