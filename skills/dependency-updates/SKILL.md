---
name: dependency-updates
description: Upgrades dependencies safely and reviews breaking changes. Use when updating packages, resolving vulnerabilities, or when the user asks to bump or upgrade deps.
---

# Dependency Updates

Upgrade dependencies in a controlled way, account for breaking changes, and keep the build and tests green.

## When to Activate

- User asks to "update dependencies", "upgrade X", "bump version", or "fix npm audit"
- Addressing security advisories or CVEs
- Preparing a maintenance release or keeping deps current
- After adding a new dependency (ensure version and peer deps are correct)

## Work Process

1. **Identify what to update** — All deps, major-only, or specific packages. Check manifest (package.json, requirements.txt, go.mod, etc.) and lockfile (package-lock.json, yarn.lock, Pipfile.lock). For security, prefer minimal version bump that fixes the CVE.
2. **Check changelogs and release notes** — For each major (or significant minor) upgrade, read CHANGELOG or GitHub releases for breaking changes and migration steps. Note required code or config changes.
3. **Plan order** — Prefer: dev/test deps first, then runtime; one or a small group at a time so failures are easy to attribute. If multiple majors, consider doing them in separate PRs.
4. **Apply changes** — Update version in manifest; run install/update command (e.g. `npm install`, `pip install -r requirements.txt`, `go get`). Resolve any immediate resolution or compile errors (peer dep, removed API).
5. **Run tests and key flows** — Full test suite; fix failures caused by API or behavior changes. Run app locally if applicable (smoke test).
6. **Commit lockfile** — Always commit the updated lockfile so CI and deploys are reproducible.
7. **Document** — In PR or release notes: list upgraded packages, any breaking change or config change, and manual steps (e.g. env var, new config option).

## Safety Rules

- **Major versions** — Assume possible breaking changes. Read migration guide; run full test suite; fix or adapt call sites.
- **Lockfile** — Commit updated lockfile. Do not leave it out or "regenerate in CI" unless that is the project's explicit policy.
- **Security fixes** — Prefer smallest version bump that addresses the CVE. Verify no new breakage; if the fix is only in a major, assess risk and document.
- **Monorepos / shared deps** — Consider impact on other packages or services that depend on the same library. Test dependents after upgrade.
- **Rollback** — If the change is risky, ensure previous lockfile or version is available for quick rollback.

## Tool-Specific Notes

### npm / Node

```bash
# Audit
npm audit
npm audit fix   # Apply semver-compatible fixes; review before commit

# Update
npm update [package]   # Within semver range
npm install package@latest   # Latest; may be major
npm install package@x.y.z    # Pinned

# After change
npm ci          # Clean install from lockfile
npm test
```

- Check `peerDependencies` and `overrides`/`resolutions` when upgrading. Fix peer warnings if they cause runtime issues.
- For workspaces/monorepos, run from root and in affected packages.

### Python (pip)

```bash
pip install --upgrade package
pip freeze > requirements.txt   # Or use pip-tools: pip-compile
pip check   # Verify deps consistent
```

- Prefer `requirements.txt` with pinned versions (or pip-tools) for reproducibility. Document why a version was upgraded (e.g. CVE).

### Go

```bash
go get module@version
go mod tidy
go build ./...
go test ./...
```

- Major version in path (e.g. `module/v2`). Update import paths if module changed major version.

## Output

- **Updated manifest and lockfile** — Or exact commands run and resulting versions.
- **Summary** — Packages changed (name and old → new version). Breaking changes addressed (list or "none"). Tests run (pass/fail). Any manual steps (env, config, migration).
- **If something cannot be updated** — Explain (e.g. incompatible peer, no fix in supported version). Suggest: patch fork, wait for upstream, or accept risk with justification.

## Checklist

- [ ] Changelog/release notes checked for breaking changes
- [ ] Version updated in manifest; install/update run
- [ ] Lockfile updated and committed
- [ ] Test suite passes
- [ ] No new peer dep or resolution errors in production path
- [ ] Breaking changes and config changes documented

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| `npm update` or bulk upgrade without reading changelogs | Check breaking changes for majors and significant minors |
| Committing manifest without lockfile | Always commit lockfile for reproducibility |
| Ignoring peer dependency warnings | Resolve or document; can cause runtime failures |
| Upgrading many majors in one PR | Prefer one major (or small group) per PR for easier review and rollback |
| "Tests pass, ship it" without running the app | Smoke test critical flows when runtime behavior might change |

## Red flags

- **Security fix only in major** — Evaluate risk; document decision (upgrade vs accept vs mitigate).
- **Deprecated API with no replacement** — Plan migration or find alternative package.
- **Incompatible peer** — Do not force-resolve without testing; consider upgrading the peer or finding another package.
