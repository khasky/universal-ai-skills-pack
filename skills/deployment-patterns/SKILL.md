---
name: deployment-patterns
description: Deployment strategies, release safety, health checks, and rollback. Use when designing or documenting deploy/rollback, CI/CD, or production readiness. Applies to any stack or platform.
---

# Deployment Patterns

Getting changes to production safely matters regardless of stack or platform. This skill focuses on *patterns* and *principles* so you can adapt them to your runtime, host, and team size.

**Why this matters:** Bad deploys waste time, burn trust, and sometimes take the product down. A little structure—clear strategy, health checks, and a rollback plan—turns "hope it works" into "we know how to fix it."

## When to Activate

- Designing or documenting how you deploy (CI/CD, manual, or hybrid)
- Adding or changing health checks, rollback, or release process
- Preparing for production (first deploy or a new service)
- User asks about "deployment", "rollback", "production readiness", or "CI/CD"
- Writing or updating a runbook for deploy/rollback

## Core Principle

**Every deploy should have a rollback path.** If you can't undo it, don't ship it until you've defined how you would undo it. In practice that means: know what "previous good" is, and have a tested way to get back there.

## Generalization Note

Your project might use Node, Python, Go, Rust, or something else. You might deploy to Vercel, Railway, Fly.io, Kubernetes, EC2, or on-prem. The *ideas* below (strategies, health checks, rollback, checklist) apply everywhere. Use the commands and config your stack and platform expect; the principles stay the same.

---

## Deployment Strategies (Conceptual)

How you roll out a new version determines risk and rollback speed. Pick one that fits your risk tolerance and infrastructure.

### Rolling (default for many platforms)

Replace instances or processes gradually. Old and new versions run at the same time during the rollout.

- **Good for:** Most apps; backward-compatible changes. No extra infra.
- **Tradeoff:** Two versions live at once—APIs and data must stay compatible.
- **Rollback:** Roll forward to previous version (same mechanism as deploy).

### Blue–green

Two identical environments; traffic switches from one to the other in one step.

- **Good for:** Critical services where you want instant rollback (switch traffic back).
- **Tradeoff:** You need 2× capacity (or at least a standby). More cost or complexity.
- **Rollback:** Switch traffic back to the old environment.

### Canary (or staged rollout)

Send a small share of traffic to the new version first, then increase if all looks good.

- **Good for:** High-traffic or risky changes; you see real-world impact before full cutover.
- **Tradeoff:** Needs traffic-splitting (load balancer, feature flags, or platform feature). More moving parts.
- **Rollback:** Route traffic away from the new version; fix or revert.

**When in doubt:** Start with whatever your platform does by default (often rolling). Document it. Add blue–green or canary when the team or product needs tighter control or safer rollouts.

---

## Health Checks

Something must answer "is this instance ready?" and "is it still alive?" so the platform or load balancer can route traffic and restart bad instances.

**Principle:** Expose a minimal endpoint (e.g. `/health` or `/ready`) that returns success only when the app can serve work. No secrets, no heavy logic.

- **Liveness:** "Process is up." Often: HTTP 200 from a simple route or a quick DB ping. Fail this and the platform may restart the process.
- **Readiness:** "Ready to take traffic." Might depend on DB, cache, or config. Fail this and the platform stops sending traffic until it passes again.

Use your platform’s conventions (Kubernetes liveness/readiness, or a single health URL). The important part is: *something* checks that the app is really okay, and the deploy/rollback process uses it.

**Why it matters:** Without health checks, bad deploys can receive traffic and cause cascading failures. A simple check catches many problems before users do.

---

## Rollback

Before you deploy, know how you’ll go back.

- **What is "previous good"?** — Previous image tag, previous commit, or previous release artifact. Record it (e.g. in CI or runbook) so you’re not guessing at 3 a.m.
- **How do you roll back?** — Redeploy previous artifact, switch traffic back (blue–green/canary), or revert a migration (see database-migrations skill). Document the exact steps and who can run them.
- **Data and config:** If the deploy includes DB migrations or config changes, rollback may require a backward-compatible migration or config revert. Plan that when you design the change.

If you can’t roll back safely, treat the deploy as higher risk and add checks (staging, canary, or manual approval) until you’ve defined and tested rollback.

---

## Environment and Secrets

**Principle:** Config and secrets live outside the code and the image. Use environment variables or a secrets manager; inject at runtime. Never bake secrets into the image or commit them.

- Document required env vars (see **env-and-config** skill). Validate at startup when possible so misconfig fails fast.
- Different environments (dev, staging, prod) should get config from the same mechanism (env, vault, etc.) with different values—not different code paths.

**Generalization:** Whether you use Vercel env, Kubernetes secrets, AWS Secrets Manager, or a `.env` file (dev only), the idea is the same: no secrets in repo or image; inject at run time.

---

## Production Readiness (Checklist)

Before calling a deploy "production ready," the team (or you) can run through something like this. Adapt to your stack and risk level; small teams might trim, regulated teams might add.

**Application**

- [ ] Tests pass (unit and, if you have them, integration); run the project’s test command.
- [ ] No hardcoded secrets; config/secrets from env or vault.
- [ ] Errors handled and logged in a way you can debug; no sensitive data in logs (see **logging-standards**).
- [ ] Health endpoint exists and reflects real readiness (and liveness if your platform uses it).

**Deploy and rollback**

- [ ] Rollback path is defined and, if possible, tested (e.g. in staging).
- [ ] "Previous good" artifact or tag is known and reachable.
- [ ] If you use DB migrations, they are backward-compatible or rollback is documented (see **database-migrations**).

**Operational**

- [ ] Someone knows how to run rollback and when to do it (runbook or doc).
- [ ] Logs and metrics go somewhere you can use (platform default or your own). No need for perfection—just enough to debug and know when things break.

**Security (basics)**

- [ ] Dependencies scanned or reviewed for known critical issues (see **dependency-updates**, **security-audit**).
- [ ] HTTPS and auth where the product expects them; no default admin credentials.

You don’t have to tick every box on day one. Use this as a target and improve over time. When in doubt, at least have: tests passing, no secrets in code, health check, and a rollback plan.

---

## CI/CD (Principles Only)

Automation should run tests and (optionally) deploy in a repeatable way. Exact tools don’t matter; the flow does.

- **On every change (or every PR):** Lint, test, and build. Fail the pipeline if any of these fail. Use the project’s commands (e.g. `npm test`, `pytest`, `go test ./...`).
- **On merge to main (or your release branch):** Build an artifact (image or bundle), run tests again if needed, then deploy to staging (if you have it) and/or production. Who triggers prod (automatically vs. manual) depends on your risk and compliance.
- **Secrets:** Never log or expose secrets in CI. Use the platform’s secret store (e.g. GitHub Actions secrets, GitLab CI variables) and reference them by name.

Whether you use GitHub Actions, GitLab CI, Jenkins, or something else, the same ideas apply: test before deploy, build once and promote the same artifact, and keep secrets out of logs and code.

---

## Human Touch and Exceptions

- **Small or solo projects:** You might deploy manually and skip staging. Still define "previous good" and one way to roll back. It pays off the first time a deploy goes wrong.
- **Regulated or large teams:** You may need stricter approvals, audit logs, and change windows. The patterns (health check, rollback, no secrets in code) still hold; add process around them.
- **When something doesn’t fit:** Document the exception and why (e.g. "We deploy from main without staging because …"). The next person will thank you.

---

## Integration

- **runbook-incident** — Turn this into concrete runbooks: "Deploy to production", "Rollback API", with exact commands for your platform.
- **database-migrations** — Deploy and DB changes must be coordinated; migrations should be backward-compatible or rollback planned.
- **env-and-config** — Document env vars and validate at startup so misconfig is caught early.
- **verification-before-completion** — Before marking "deploy done", verify health (and tests if you run them post-deploy).

---

## Checklist (Before a Production Deploy)

- [ ] Tests and build pass in CI (or locally if no CI).
- [ ] Rollback path is known and documented.
- [ ] Health check exists and is used by the platform or load balancer.
- [ ] Config and secrets come from env/vault; not from code or image.
- [ ] At least one person (or runbook) can perform rollback.

Apply these patterns in the way your stack and platform support. The goal is safe, repeatable deploys and a clear way back when things go wrong.
