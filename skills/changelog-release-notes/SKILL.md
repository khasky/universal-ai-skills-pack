---
name: changelog-release-notes
description: Generates changelogs and release notes from commits and tags. Use when releasing a version, writing release notes, or maintaining a CHANGELOG.
---

# Changelog & Release Notes

Produce human-readable changelogs and release notes from git history so users and stakeholders see what changed.

## When to Activate

- User asks for "changelog", "release notes", or "what changed in v1.2"
- Preparing a release (before tagging or publishing)
- Maintaining CHANGELOG.md or similar
- After a release cycle to document shipped work

## Work Process

1. **Get the range** — Two refs (e.g. `v1.1.0..HEAD`, `main..release/1.2`) or list of commits. Confirm with user if ambiguous.
2. **Fetch commits** — `git log --oneline BASE..HEAD` (or equivalent). Optionally filter by path or author per project.
3. **Parse and group** — If using conventional commits, group by type (feat → Added, fix → Fixed, etc.). Deduplicate and merge similar entries (e.g. "fix login" and "fix auth" → one line or two distinct bullets).
4. **Draft entries** — One line per change; add issue/ticket ref if project uses them. Do not invent; use commit message or diff to describe. If messages are vague ("fix bug"), infer from diff when possible or use a generic line and note "verify description."
5. **Apply project format** — Match existing CHANGELOG style (e.g. Keep a Changelog). Omit empty sections. Add version heading and date.
6. **Release notes** — From the same data: short summary, bullet list of user/developer impact, breaking changes and migration steps, link to full changelog.

## Inputs

- **Required:** Git range (e.g. `v1.1.0..HEAD`) or explicit list of commits.
- **Optional:** Conventional commit usage (feat/fix/docs/etc.), issue tracker prefix (e.g. `#123`, `PROJ-456`), existing CHANGELOG path and style.

## Changelog Format (Keep a Changelog style)

```markdown
## [1.2.0] - 2024-03-15

### Added
- CSV export for reports (PROJ-101).
- Retry with backoff for payment API calls.

### Changed
- Pagination now uses cursor by default; offset still supported.

### Fixed
- Login redirect when session expired (PROJ-99).
- Date formatting in report header.

### Security
- Bumped axios to 1.6.2 (CVE-2023-XXXX).

### Deprecated
- Query param `api_key`; use `X-API-Key` header (removal in v2).

### Removed
- (none this release)
```

- **Version and date** — `## [X.Y.Z] - YYYY-MM-DD`. Use date of release or tag.
- **Sections** — Added, Changed, Deprecated, Removed, Fixed, Security. Omit empty sections.
- **Entries** — One line per change; optional ticket ref. No redundant "Fixed a bug" without context.

## Mapping conventional commits to sections

| Commit type | Changelog section |
|-------------|-------------------|
| feat | Added |
| fix | Fixed |
| docs | Changed (or omit if internal-only) |
| perf | Changed or Added |
| refactor | Changed (if user-visible) or omit |
| chore, ci, build | Omit unless notable (e.g. security) |
| BREAKING CHANGE | Added under **Breaking** or **Changed**; describe migration |

## Release Notes (user-facing)

- **Summary** — 1–2 sentences: what this release is and why it matters.
- **Highlights** — Bullet list of user- or developer-visible changes; emphasize impact and benefits.
- **Breaking changes** — Clearly called out with upgrade or migration steps (and version when the old behavior is removed).
- **Links** — Full changelog, diff, or upgrade guide if applicable.

Example:

```markdown
# Release 1.2.0 — March 15, 2024

This release adds CSV export for reports and improves reliability of payment processing.

## Highlights
- **Export reports as CSV** — Download any report from the Reports page.
- **More reliable payments** — Automatic retry with backoff when the payment provider is slow.
- **Security** — Dependency updates for known vulnerabilities.

## Breaking changes
None.

## Upgrade
No action required. Deploy as usual. See [CHANGELOG](link) for full details.
```

## Rules

- **Do not invent changes** — Only include what is in the given commits or diff. If a commit is vague, infer from the diff or use a neutral line and suggest verification.
- **Match project style** — If the repo has CHANGELOG.md, use its section names and order (e.g. Added/Changed/Fixed vs New/Changed/Fixed). If it uses bottom-up order (oldest first), follow that.
- **One entry per logical change** — Merge duplicate or near-duplicate commits into one line when they describe the same change.
- **Prefer appending** — New version at top for "newest first" style; update existing file rather than creating a duplicate.

## Checklist

- [ ] Git range or commit list confirmed
- [ ] Entries derived from actual commits/diff (no invented items)
- [ ] Sections and formatting match existing CHANGELOG (if any)
- [ ] Version and date set correctly
- [ ] Breaking changes and migration called out in release notes
- [ ] Empty sections omitted

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| Copying commit messages verbatim when vague | Infer from diff or write one clear line; note "verify" if unsure |
| Listing every commit including "chore" | Include user/developer-visible changes; omit internal chore unless security/release-critical |
| Claiming "no breaking changes" without checking | Scan for BREAKING CHANGE footer and major dependency/API changes |
| New CHANGELOG in different format than existing | Match existing style; if none, use Keep a Changelog |
