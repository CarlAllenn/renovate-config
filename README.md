# renovate-config

Shared Renovate policy for all active CarlAllenn repos, consumed via:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>CarlAllenn/renovate-config"]
}
```

Because this repo is named `renovate-config` with a `default.json`, Renovate
auto-suggests it as the sole config when onboarding any new repo under this
account — new repos are born compliant.

## Policy (what `default.json` does)

Built on `config:best-practices` (digest pinning for Docker images and GitHub
Actions, pinned devDependencies, weekly lock file maintenance, abandonment
detection, config migration PRs; its 3-day npm release age is raised to 14
days below), plus:

| Tier | Behavior |
| --- | --- |
| Non-major, semver-stable (>=1.0) | Automerge as silent branch push; PR only if CI red |
| 0.x patch | Automerge (branch push) |
| 0.x minor | Manual PR — SemVer allows 0.x minors to break |
| Digest / pin / pinDigest | Automerge (branch push) |
| Major | Manual PR, one per major hop (`:separateMultipleMajorReleases`) |
| Lock file maintenance | Weekly, silent branch automerge |
| Security fixes | Bypass all queues, schedules, and age gates |

Guard rails that substitute for human review on the automerged tiers:

- `minimumReleaseAge`: **7 days for everything** — one full ecosystem cycle
  including a weekend (when takeovers preferentially ship and detection is
  slowest), comfortably past npm's 72h unpublish window. Was 14 days
  (2026-08-01 → 2026-08-04); halved because the pnpm install-time gate is
  security-blind, so every security fix younger than the gate costs a
  manual adjudication entry, and 14 doubled that chore for no detection
  benefit anyone has measured past one week. Branches are not even created until the
  age gate passes (`internalChecksFilter: strict` is Renovate's default), so
  nothing burns CI while pending. This is one leg of the **three-layer
  age-gate policy**: the same 7 days is enforced at install time by pnpm
  (`minimumReleaseAge: 10080`) and mise (`minimum_release_age = "168h"`) in
  the consuming repos. The numbers must stay equal — pnpm re-verifies
  lockfile entries on install, so a gap in either direction breaks Renovate
  branches or opens a bypass. Rationale: `templates/pnpm/README.md`.
- `osvVulnerabilityAlerts`: offline OSV checks on direct deps, including
  malicious-package protection (known-bad versions never get a PR).
- Rust toolchain (mise `rust` pin) is never automerged — MSRV bumps are
  deliberate decisions.

Rules are matcher-based, so the policy is stack-agnostic: cargo rules are
inert in npm repos and vice versa. Repo-specific rules (custom managers,
approval gates, groups) stay in each repo's own `renovate.json`, which
overrides anything here when both match.

## CI prerequisite

Branch automerge needs the `ci` check on the branch itself (no PR exists
unless the branch goes red), so each consuming repo's CI workflow must
trigger on:

```yaml
push:
  branches: [main, "renovate/**"]
pull_request:
```

## Parking a PR

Apply the `stop-updating` label to any Renovate PR to freeze it (no rebases,
no CI reruns) while keeping it open and visible. Remove the label to resume.

## Beyond Renovate presets

- `templates/` — canonical per-tool baselines (mise, task, lefthook, and the
  nine universal linters so far; supply-chain and CI to follow) —
  copy-template class, synced by deliberate drift audits rather than live
  propagation. The linter decision record lives in
  `templates/linters/README.md`; this repo consumes the universal lint layer
  itself (dogfood) via the same `remotes:` mechanism as every active repo.
- `lefthook/universal.yml` — the shared git-hook job set, consumed **live**
  by every active repo via lefthook `remotes:` (the one tool besides
  Renovate itself with a stable first-class remote-config mechanism).
  Decision record and consumer skeleton: `templates/lefthook/README.md`.

Decision record: `docs/decisions/0018-renovate-shared-preset.md` in
monumental-archive.
