---
name: multi-perspective-review
description: Multi-perspective PR review with caller-chosen lenses. Gathers context via gh, submits labelled inline findings. Advisory only, no gate verdicts.
---

# Multi-Perspective Review

Deep, lens-structured PR review for any change type. Produces inline, actionable, labelled GitHub review comments — not a merge verdict.

## Routing: this skill vs factory-review (read first)

- **Use this skill** when you want deep findings from multiple independent lenses before or alongside a gate review.
- **Use `factory-review`** when you need the merge decision. `factory-review` is the SOLE authority for APPROVE / APPROVE WITH NITS / REQUEST CHANGES and for risk-manager APPROVE-FOR-LIVE / VETO.
- This skill MUST NOT issue a gate verdict, MUST NOT claim merge-readiness, and MUST NOT duplicate the risk-manager veto. If asked for a verdict, route to `factory-review`.
- Typical flow: this skill (findings) → `factory-review` (verdict). Reference this skill's comment IDs/threads in the gate review; do not re-litigate them there.

## Step 0: Ask for perspectives (mandatory, runtime)

Do NOT assume perspectives. ASK the caller which lenses to apply, and RECOMMEND defaults by change type:

| Change type | Recommended perspectives (default max 3) |
|---|---|
| Trading (strategy, backtest, config, risk, execution) | Sceptical SSE (correctness/edge cases) / Architect (structure/drift/coupling) / Quant (backtest fidelity/fees/slippage/risk-guard) |
| Infra / DevOps (IaC, pipelines, Lambda, networking) | Sceptical SSE (correctness/failure modes) / Architect (blast radius/coupling/drift) / Security (secrets/IAM/least-privilege) |
| Generic (features, refactors, docs-adjacent code) | Correctness (does it do what it claims + edge cases) / Maintainability (readability/duplication/abstraction fit) / Scope (single purpose, no stowaways) |

Rules:

- Present the recommendation, then require explicit caller confirmation (or caller-supplied alternatives) before reviewing.
- Cap at 3 perspectives unless the caller justifies more. Each finding is labelled with the perspective that raised it.
- Caller-defined custom lenses are allowed; restate the agreed lens list before proceeding.

## Perspective checklist (instantiate per agreed lens)

Apply each agreed perspective sequentially to every file in the diff. Illustrative checklists for the built-in lenses:

- **Sceptical SSE:** off-by-one, sign errors, type confusion, null/NaN/empty/boundary inputs, error-path silence, float precision, dead code, config fallback semantics, test-claim mismatch (tests that don't prove their name).
- **Architect:** duplicated logic across modules (and drift risk), magic constants, mixed concerns / unclear boundaries, hidden coupling, over/under-engineering, scope discipline.
- **Quant (trading only):** backtest-vs-production gaps (data source, fees, slippage, fill model), risk-guard weakening, capital-efficiency cost, parameter sensitivity, concurrency / double-execution.
- **Security (infra only):** secret handling, IAM scope, public exposure, unpinned actions/deps, missing audit logging.
- **Correctness / Maintainability / Scope (generic):** claim-vs-code match, edge cases, readability, duplication, abstraction fit, single-purpose discipline.

## Workflow

### Step 1: Gather context

Before reading code, collect:

- PR title, description, and linked issues
- The full diff (`gh pr diff <number>`)
- Existing review comments (`gh pr view <number> --json comments`)
- The base and head branches
- Key files touched (per change type: strategy/config/backtest/tests, or IaC/pipeline/workflow/manifest files)

### Step 2: Read through each agreed perspective

For each file in the diff, apply the agreed perspectives sequentially (one pass per lens). Do not blend lenses into generic observations — keep each finding attributable.

### Step 3: Cross-validate existing comments

Read all existing comments on the PR. Do not repeat them. If you agree, leave a reaction. If you disagree or have a different angle, say so explicitly with reasoning.

### Step 4: Draft comments

Each comment MUST be:

- **Inline** — attached to the specific line in the diff
- **Contextual** — reference the exact code, not general principles
- **Actionable** — suggest a concrete change, or explicitly mark FYI
- **Labelled** — prefix with the perspective, e.g. `**Perspective: [SSE | Architect | Quant | Security | Correctness | Maintainability | Scope]**`

Do NOT create:

- Summary blurb comments or "overall looks good" meta-comments
- Comments that repeat existing review feedback
- Any APPROVE / REQUEST CHANGES verdict language

### Step 5: Submit comments via GitHub API

Use a single API call to create a review with all inline comments:

```text
POST /repos/{owner}/{repo}/pulls/{pull_number}/reviews
```

```json
{
  "commit_id": "<latest commit SHA>",
  "event": "COMMENT",
  "comments": [
    {
      "path": "relative/path/to/file",
      "body": "**Perspective: [<agreed lens>]**\n\nDetailed, specific observation.\n\nSuggestion: concrete change or question.",
      "line": "<line-number-in-new-file>"
    }
  ]
}
```

Requirements:

- `line` must reference a line number present in the diff hunk (right-side / new file).
- `commit_id` must be the latest commit on the PR branch.
- `event` MUST be `COMMENT` — never `APPROVE` or `REQUEST_CHANGES` (gate authority belongs to `factory-review`).

### Step 6: Verify submission

Confirm the comments were created:

```text
gh api repos/{owner}/{repo}/pulls/{pull_number}/comments --paginate --jq '.[].body[0:80]'
```

Check that all comments appear and none were silently dropped. Report comment count + permalinks back to the caller with the agreed lens list.

## Suggested prompt template

When delegating this review to another agent:

```text
Review PR #<number> in <owner>/<repo> using the multi-perspective-review skill.

Branch: <branch>
Description: <PR title/body>
Change type: <trading | infra | generic>

Recommended perspectives: <list per table above> — confirm or override before starting.
Apply ONLY the agreed perspectives and label every finding.
Do not repeat existing comments. Do not create a summary comment.
Do not issue a gate verdict — findings only; verdict belongs to factory-review.

Key files to pay attention to: <list>
```

## See also

- `factory-review` — the merge gate (sole verdict + risk-veto authority)
- `code-review-and-quality` — general multi-axis review approach
- `security-and-hardening` — security-specific concerns
- `doubt-driven-development` — adversarial cross-examination of non-trivial decisions
