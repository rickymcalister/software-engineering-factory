# Spec: Port PR review skill (multi-perspective-review)

## Problem

Cointrader's `pr-code-review-trader` skill provides a valuable three-perspective PR review workflow (Sceptical SSE, Architect, Quant Trader QE, with `gh` diff + inline GitHub API comments), but it is hard-coded to trading concerns and lives outside the factory. The factory has no reusable, change-type-agnostic multi-perspective review skill; `factory-review` remains the only review asset and conflates deep-review technique with merge-gate authority.

## Non-goals

- No changes to `.opencode/skills/factory-review/SKILL.md` gate authority in this item (it stays authoritative for APPROVE / REQUEST CHANGES + risk-manager veto).
- No auto-merging, auto-fixing, or submitting approving reviews on behalf of the gate.
- No port of cointrader-specific trading heuristics beyond what is needed as one recommended perspective set.

## Users

- Factory agents (delivery-manager, quality-engineer) invoking deep PR review on any change type.
- Human reviewers wanting structured, inline, multi-lens PR feedback.

## Acceptance criteria

- Given the cointrader source skill has been read fully, When the new skill is written, Then its workflow preserves gather-context-via-`gh`, cross-validate-existing-comments, draft-inline-actionable-labelled-comments, submit-via-single-review-API-call, and verify steps.
- Given a caller invokes the new skill, When perspectives are selected, Then the skill ASKS FOR perspectives at runtime and RECOMMENDS defaults per change type (trading → SSE/Architect/Quant; infra → SSE/Architect/Security; generic → correctness/maintainability/scope), instead of hard-coding SSE/Architect/Quant.
- Given both review assets exist, When a reader compares them, Then the new skill defines routing (when to use factory-review vs multi-perspective-review) and states factory-review is the sole gate authority (no duplicated APPROVE / REQUEST CHANGES authority, no risk-manager veto duplication).
- Given repo pre-commit checks, When CI validates frontmatter, Then the new skill file has `name` + `description` frontmatter and the spec exists at `docs/specs/port-pr-review-skill.md`.
- Given the change lands, When reviewed, Then no files outside the skill + spec are touched and no merge to main occurs (branch left ready for PR).

## Risks / open questions

- Risk: callers confuse deep-review output with gate verdict → mitigated by explicit routing section denying gate authority to the new skill.
- Risk: perspective proliferation (too many lenses, noisy comments) → mitigated by recommending max 3 perspectives and requiring caller confirmation.
- Open: none blocking — user already answered perspective-genericity requirement.

## Rollout / rollback

- Rollout: new skill file + spec on branch `feature/multi-perspective-review-skill`; DRAFT PR, no main merge.
- Rollback: delete branch; factory-review remains unaffected.

## Feasibility (cto sign-off, implicit)

- Decision: BUILD. Markdown-only addition, no stack/dependency/ADR implications. No ADR needed.
