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
Actions, pinned devDependencies, weekly lock file maintenance, npm 3-day
release age, abandonment detection, config migration PRs), plus:

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

- `minimumReleaseAge`: 3 days base, **14 days for runtime `dependencies`**
  (the docs-recommended malware-scanner/unpublish window). Branches are not
  even created until the age gate passes (`internalChecksFilter: strict` is
  Renovate's default), so nothing burns CI while pending.
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

## Future

This repo will also grow a `templates/` directory of canonical per-tool
baselines (mise settings block, biome, trivy, lefthook, editorconfig) —
copy-template class, synced by drift audits rather than live propagation.

Decision record: `docs/decisions/0018-renovate-shared-preset.md` in
monumental-archive.
