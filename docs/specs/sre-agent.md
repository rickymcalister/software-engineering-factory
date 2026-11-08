# Spec: SRE agent (steady-state reliability)

## Problem

The factory has no owner for steady-state reliability. CloudWatch log investigation, metric and alarm review, SLO and threshold design, alert lifecycle management, and runbooks currently fall between senior-engineer (feature code), quality-engineer (merge gate), and the forthcoming change-time operations role (devops engineer, PR #2, not yet on main). Without a defined owner, alarm thresholds drift, noisy alerts go untuned, and incident diagnosis has no guardrailed home. This item adds a dedicated SRE agent scoped to CloudWatch logs, metrics, alarms, and dashboards for v1.

## Non-goals

- No X-Ray tracing, no CloudTrail audit wiring, and no SNS paging trees in v1 (CloudWatch logs, metrics, alarms, and dashboards only).
- No change-time operations: deploys, release rollbacks, pipeline edits, SAM and stack mutations, and wiring alarms at deploy time belong to the change-time operations role (devops engineer, PR #2, not yet on main).
- No application feature work (senior-engineer), no architecture decisions or ADRs (solutions-architect, cto), no merge-gate verdicts (quality-engineer through factory-review), no live-capital risk calls (risk-manager).
- No new CI jobs and no changes to existing agent or skill authority in this item.

## Users

- delivery-manager, which routes steady-state reliability work (diagnose, propose thresholds, lifecycle alerts, write runbooks) to the new agent.
- On-call humans receiving diagnoses, alarm proposals with evidence, and runbooks for known failure modes.
- Service owners wanting SLOs, thresholds, and dashboards that change only under plan, approval, and verification guardrails.

## Acceptance criteria

- Given a diagnosis request, When the agent investigates, Then it works read-only (CloudWatch logs, metrics, alarm state), reports zero mutations, and cites the evidence used.
- Given an alarm change the user has NOT approved, When the agent is asked to apply it, Then nothing is applied and the refusal plus the exact approval required are recorded in the report.
- Given explicit approval for an alarm change, When the agent applies it, Then it records the approver and the approved revision, verifies alarm and metric state afterwards, attaches the evidence, and notes the rollback path.
- Given a request mixing steady-state and change-time work, When the seam is resolved, Then steady-state (SLOs, thresholds, alert lifecycle, log-driven diagnosis, runbooks) stays with the SRE agent, change-time (deploy, verify, rollback, pipeline, SAM and stack mutation, wiring alarms at deploy time) goes to the change-time operations role (devops engineer, PR #2), and disputed production thresholds escalate to a named human with nothing applied until resolved.
- Given any investigation touching secrets, When the agent reports, Then secret values are never printed and only names and locations appear, redacted by default.
- Given the four repo CI jobs, When validation runs, Then the new agent file and this spec pass frontmatter, markdown hygiene, spec-template compliance, and routing integrity with no unresolvable references.

## Risks / open questions

- Risk: readers confuse the SRE agent with the change-time operations role (devops engineer, PR #2) since both touch CloudWatch alarms. Mitigated by the seam rule in the agent file (steady-state here, change-time there) and forward-compatible prose that does not require the other role file to exist.
- Risk: alarm proposals get applied on implied consent. Mitigated by mandatory guardrails (plan before execute, named approver for production, refusal recorded without approval, verify after with rollback note).
- Risk: routing-integrity CI flags role mentions as unresolvable refs. Mitigated by hyphenating only agent names that exist on this branch and writing the not-yet-merged role space-separated (devops engineer, PR #2).
- Open: none blocking. User decisions on scope, rights, production rule, and split structure are binding and recorded above.

## Rollout / rollback

- Rollout: spec plus agent file plus one operating-model routing row on branch `feature/sre-agent`; DRAFT PR only. No merge and no push from this item; the branch is left ready for review.
- Rollback: delete the branch. No existing agent, skill, or CI job is modified, so the factory is unchanged.
