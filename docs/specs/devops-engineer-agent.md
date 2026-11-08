# Spec: DevOps Engineer agent

## Problem

Factory routing has no owner for AWS serverless operations and CI/CD pipeline work (Lambda/SAM, API Gateway, EventBridge, CloudWatch, GitHub Actions). Such work currently falls between senior-engineer (features), solutions-architect (design), and cto (decisions), with no defined guardrails for live infrastructure execution.

## Non-goals

- No application feature implementation (stays with senior-engineer).
- No cross-service architecture design or ADR ownership (stays with solutions-architect / cto).
- No merge-gate verdicts (stays with quality-engineer) and no risk-manager veto override.
- No autonomous production deploys without explicit human approval.

## Users

- delivery-manager, which routes infra/pipeline work to this agent.
- Factory engineers needing advisory + guarded execution for serverless and CI/CD tasks.

## Acceptance criteria

- Given AWS serverless + CI/CD scope, When the agent file is read, Then it defines responsibilities covering Lambda/SAM, API Gateway, EventBridge, CloudWatch, and GitHub Actions pipelines in advisory + executable capacity.
- Given adjacent roles exist, When boundaries are checked, Then the file states non-goals and boundaries vs senior-engineer (implements features), solutions-architect (designs), cto (decides), and quality-engineer (gates), plus when delivery-manager routes to devops-engineer.
- Given live infrastructure risk, When guardrails are checked, Then the file requires plan-before-execute, verify-after, no secret exfiltration, prod deploys only with explicit approval, rollback note, and CloudWatch evidence.
- Given repo pre-commit checks, When CI validates frontmatter, Then the agent file has `description` + `mode: subagent` frontmatter and the spec exists at `docs/specs/devops-engineer-agent.md`.
- Given the change lands, When reviewed, Then no files outside the agent + spec are touched and no merge to main occurs (branch left ready for PR).

## Risks / open questions

- Risk: agent used as general implementer, blurring senior-engineer boundary → mitigated by explicit non-goals + routing table.
- Risk: accidental live-infra mutation → mitigated by plan/approval/verify guardrails and rollback note requirement.
- Open: none blocking — scope (serverless + CI/CD, advisory + guarded execution) confirmed by user.

## Rollout / rollback

- Rollout: new agent file + spec on branch `feature/devops-engineer-agent`; DRAFT PR, no main merge.
- Rollback: delete branch; existing agents unaffected.

## Feasibility (cto sign-off, implicit)

- Decision: BUILD. Markdown-only addition, no stack/dependency/ADR implications. No ADR needed.
