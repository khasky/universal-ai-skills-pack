---
name: runbook-incident
description: Creates and updates runbooks and incident response steps. Use when documenting ops procedures, incident response, or when the user asks for a runbook.
---

# Runbook & Incident Response

Document operational procedures and incident response so on-call and ops can act quickly and consistently.

## When to Activate

- User asks for "runbook", "incident response", "how do we deploy/rollback", or "ops procedures"
- Adding a new service or deploy process that needs documentation
- Post-incident: document what was done and what to do next time
- Setting up or improving on-call and escalation

## Work Process

1. **Identify procedure type** — Deploy, rollback, failover, scaling, alert response, or incident flow. Gather from the user or from existing scripts and docs: what is done today, in what order, and with what tools.
2. **Gather steps and commands** — List actual commands, URLs, and tool paths. Test commands in a safe environment if possible (staging, dry-run). Note where credentials or secrets come from (vault, env); never put real secrets in the runbook.
3. **Add structure** — Purpose, prerequisites, numbered steps, expected outcome per step, failure handling ("if X fails, do Y or escalate to Z"), verification, rollback, and contacts.
4. **Write for the on-call** — Assume 3 a.m. and high stress. One action per step; exact commands; clear "success looks like this" and "if this fails, do that." Use code blocks for commands; specify shell and environment (e.g. production, staging).
5. **Review and place** — Put in repo (e.g. docs/runbooks/) or link from README/ops doc. Align with existing runbook style and tooling (PagerDuty, status page) if present. Keep contacts and escalation paths updated.

## Runbook Contents

### Required sections

- **Purpose** — When to use this runbook (e.g. "Deploy application to production", "Rollback API to previous version", "Respond to alert: HighErrorRate"). One sentence.
- **Prerequisites** — Access (e.g. AWS role, VPN), tools (kubectl, CLI), and where to get credentials (e.g. "Secrets in Vault path X" or "Use SSO role Y"). No actual secrets in the doc.
- **Steps** — Numbered, in order. Each step: (1) action in plain language, (2) command or link, (3) expected outcome, (4) if it fails: retry, rollback step, or escalate (with contact or link).
- **Verification** — How to confirm success: health check URL, dashboard, smoke test command, or "check logs for X." Include a concrete check.
- **Rollback** — How to undo or roll back; when to use it (e.g. "If deploy causes errors, run Rollback API runbook"). Short steps or link to rollback runbook.
- **Contacts** — Who to escalate to (team, Slack channel, PagerDuty). Keep names/channels current.

### Optional

- **Frequency** — How often this is run (e.g. every release, only on incident).
- **Estimated duration** — Rough time (e.g. "~5 minutes") so on-call can plan.
- **Related** — Links to other runbooks (e.g. "See also: Database failover", "See: Incident response flow").

## Format and style

- **Markdown** in the repo (e.g. `docs/runbooks/deploy-production.md`, `docs/runbooks/rollback-api.md`). One file per procedure when possible.
- **Commands in code blocks** with language (bash, powershell). Specify environment: "Run in production." or "Set ENV=production."
- **Idempotency** — Note "run once" vs "repeat if needed" for steps that are not idempotent.
- **No secrets** — Reference "use credential X from vault" or "env var Y"; never paste keys or passwords.

## Incident response (high level)

When documenting incident response (not a single procedure):

- **Detection** — How the issue is detected: alert name, dashboard, user report. Link to alert or dashboard.
- **Triage** — Severity (P1/P2) and impact (who/what affected). Initial hypothesis or "run runbook X first."
- **Mitigation** — Steps to restore service or limit impact: rollback, disable feature, scale, restart. Link to specific runbooks.
- **Communication** — When and how to update status page, users, or internal channels. Who is responsible.
- **Post-incident** — Blameless postmortem: what happened, root cause, action items (fix, runbook update, alert tuning). Optional: link to template or doc.

## Example runbook outline

```markdown
# Deploy to production

## Purpose
Deploy the application to production. Use for scheduled releases.

## Prerequisites
- Access to CI/CD (GitHub Actions or deploy tool).
- Credentials: use SSO role "deploy-prod". No manual secrets.

## Steps
1. **Trigger deploy** — In GitHub Actions, run workflow "Deploy Production" on branch `main`. Expected: workflow starts.
   - If fails: Check Actions tab for error; see [Troubleshooting](link). Escalate to #eng-ops.
2. **Wait for green** — Wait until workflow completes (~5 min). Expected: "Deploy Production" green.
   - If fails: See [Rollback](link). Do not retry without checking logs.
3. **Verify** — Open https://app.example.com/health. Expected: `{"status":"ok"}`.
   - If not ok: Run [Rollback API](rollback-api.md). Notify #eng-ops.

## Verification
- Health: https://app.example.com/health
- Dashboard: [Link to Grafana or monitoring]

## Rollback
See [Rollback API](rollback-api.md).

## Contacts
- Escalate: #eng-ops (Slack). PagerDuty: rotation "backend".
```

## Rules

- **No real secrets or credentials** in the runbook. Reference vault, env, or "obtain from X."
- **Align with existing style** — If the project has runbooks, match structure and tool names. If they use PagerDuty or a status page, reference them.
- **Keep steps actionable** — "Run X" with the exact command or link. Avoid "ensure the config is correct" without saying how to check or fix.
- **Update after incidents** — If something was missing or wrong during an incident, update the runbook and add a postmortem action item.

## Checklist

- [ ] Purpose and prerequisites clear; no secrets in doc
- [ ] Steps numbered; each has action, command/link, expected outcome, failure handling
- [ ] Verification step is concrete (URL, command, or dashboard)
- [ ] Rollback documented or linked
- [ ] Contacts/escalation listed and current
- [ ] Commands and env (prod/staging) specified

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| Vague "fix the issue" | Specific steps: "Run rollback runbook", "Restart service X", "Scale to N replicas" |
| Secrets in runbook | "Get token from Vault path X" or "Use env var Y (set in deploy)" |
| No failure handling | For each step: "If this fails, do Y or escalate to Z" |
| No verification | Add "Confirm success by: [URL or command]" |
| Outdated contacts | Review and update escalation paths periodically |
