# Spec: Technical writer agent (docs ownership)

## Problem

Factory docs (READMEs, AGENTS.md files, specs, ADRs, runbooks, API docs) drift without an owner: hygiene slips, structure diverges, templates go stale, and gaps surface only at review time. Docs work currently falls between product-manager (spec authority), cto (ADR approval), solutions-architect (seams and design), senior-engineer (code), the SRE function (runbook content), and quality-engineer (merge gate), with no defined advisory owner for cross-cutting hygiene, information architecture, and lifecycle.

## Non-goals

- No spec authority: what and why stays with product-manager.
- No ADR approval: decisions stay with cto; the writer drafts, never approves.
- No seam or design ownership: module boundaries and contracts stay with solutions-architect.
- No code changes: implementation stays with senior-engineer.
- No runbook content ownership: runbook content stays with the SRE function; the writer formats and lints only.
- No gate verdicts: the merge gate stays with quality-engineer and live veto stays with risk-manager.

## Users

- delivery-manager, which routes docs hygiene, structure, and lifecycle work to this agent.
- Factory engineers and reviewers needing readable, consistent, findable docs.

## Acceptance criteria

- Given drifting docs, When the agent runs a gap pass, Then it produces an advisory report listing each gap with file and line evidence, severity, and a suggested fix, without blocking the line.
- Given the advisory-only rule, When outputs are checked, Then they contain findings and recommendations only and never emit gate verdicts, so quality-engineer keeps the merge gate and risk-manager keeps the live veto.
- Given adjacent ownership, When seams are checked, Then spec authority stays with product-manager, ADR approval stays with cto, seams and design stay with solutions-architect, code stays with senior-engineer, and runbook content stays with the SRE function while the writer formats and lints.
- Given template drift, When lifecycle is checked, Then the agent owns docs templates, hygiene rules, information architecture, and lifecycle guidance, including forward-compatible ADR guidance for `docs/adr/` when present, without requiring that directory to exist.
- Given repo pre-commit checks, When CI validates frontmatter, Then the agent file at `.opencode/agents/technical-writer.md` carries `description` plus `mode: subagent` and this spec exists at `docs/specs/docs-writer-agent.md`.
- Given the change lands, When reviewed, Then no files outside the agent plus spec plus operating-model routing touch occur and no merge to main occurs, with the branch left ready for review.

## Risks / open questions

- Risk: writer used as gatekeeper, slowing the line and blurring the quality-engineer gate, mitigated by the explicit advisory-only guardrail and no-verdict rule.
- Risk: writer rewrites runbook content and breaks operational accuracy, mitigated by the format and lint only boundary with content staying with the SRE function.
- Risk: scope creep into broad rewrites, mitigated by small doc slices and delivery-manager scoping.
- Open: none blocking, scope is broad docs ownership with advisory enforcement as confirmed by the user.

## Rollout / rollback

- Rollout: new agent file plus this spec plus a minimal operating-model routing touch on branch `feature/technical-writer-agent`, left unmerged and unpushed for review.
- Rollback: delete the branch, existing agents and docs unaffected.
