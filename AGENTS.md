# Agent Instructions

This repo is the software engineering factory: agents + skills that run product work. Product repos (e.g. cointrader, mean-reversion-analyser) do the building; this repo defines how.

## How to work here

Talk to the **delivery-manager** agent for any multi-step task. It owns the queue and delegates to specialists. Do not bypass it by doing multi-stage work ad hoc.

## Skills

Before starting any non-trivial task, load the appropriate skill(s) using the skill tool.

Factory lifecycle sequence:

```text
factory-intake → factory-delivery → factory-review → git-workflow-and-versioning
```

Refer to the `using-agent-skills` skill for the full lifecycle and skill selection decision tree. Trading work additionally uses the strategy-designer → quant-trader → algo-trader → risk-manager chain; risk-manager has veto power over live capital.

## Branching

When beginning a new set of changes, check out a new branch before making any modifications. If it is unclear whether changes warrant a new branch, ask before proceeding.

**Branch naming convention:** `feature/`, `fix/`, `docs/`, `refactor/` + kebab-case descriptor, e.g. `feature/add-risk-veto`.

**Stacked branches:** incremental changes use stacked branches — each branch based on the previous feature branch, not on `main`.

## Pre-Commit Checks

This repo is markdown + config. Before committing, run the same four jobs CI runs (`.github/workflows/ci.yml`):

1. **Frontmatter (strict):** every `.opencode/agents/*.md` has `description` (10–200 chars) + `mode` from allowlist `[subagent]`; every `.opencode/skills/*/SKILL.md` has `name` (must match parent dir, unique) + `description` (10–200 chars).
2. **Markdown hygiene:** no trailing whitespace, no heading-level jumps (e.g. `#` → `###`), every fenced code block opens with a language tag (e.g. ` ```text `), no multiple consecutive blank lines.
3. **Spec/template compliance:** every `docs/specs/*.md` and `docs/templates/spec-template.md` must contain sections Problem, Non-goals, Users, Acceptance criteria (with Given/When/Then), Risks, Rollout. (`plan-template.md` is exempt — slice-plan shape, not a spec.)
4. **Routing integrity:** every `factory-*` / `multi-perspective-review` / agent-slug reference in markdown must resolve to an existing skill or agent file (lines naming external repos like `cointrader` are out of scope); no skill except `factory-review` may claim gate-verdict authority (`APPROVE` / `REQUEST CHANGES` / `VETO` / `APPROVE-FOR-LIVE`) — route verdicts to `factory-review`.

## Pull Requests

After committing and pushing changes, create pull requests as **DRAFT**.

Use `gh api -X PATCH repos/<owner>/<repo>/pulls/<number> --input -` (REST API) rather than `gh pr edit` for updating PR metadata, as the GraphQL endpoint used by `gh pr edit` is deprecated and silently ignores the edit.

## Autonomy and Failure Handling

When a GitHub API call returns HTTP 429 (rate limited) or a 5xx server error: pause 10 seconds, then retry that call once. If the error persists, report the failure and stop — do not retry indefinitely. Do not retry non-transient errors (400, 404, 403).
