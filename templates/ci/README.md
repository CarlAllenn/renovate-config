# CI baseline

Canonical GitHub Actions patterns, established in issue #6. Copy-template
class: consuming repos own their workflow files; propagation is a
deliberate drift-audit against this directory, never automatic.

## Decision record: reusable workflows / composite actions — rejected

The shared surface across the active repos is a **step-level prologue**
(triggers, `concurrency`, `permissions`, harden-runner, checkout,
mise-action, caches) followed by genuinely divergent jobs (MSRV, pgrx
matrix, arm64 matrix, IIIF validators, kamal). That shape defeats both
reuse mechanisms:

1. Reusable workflows compose at **job** granularity (`workflow_call` is a
   job-level `uses:`), so factoring the prologue would force every repo's
   jobs through one mega-workflow parameterized via string/number/boolean
   inputs — matrix definitions and env maps threaded stringly-typed.
2. A composite action could wrap checkout + mise + cache, but saves ~8
   lines per job while adding a live cross-repo dependency in the
   security-critical path — itself digest-pinned and Renovate-tracked —
   and hides the in-file why-comments that are house style.
3. The copy-template + drift-audit class already exists for exactly this
   trade (mise, task, linters); lefthook remains the sole live-propagation
   exception, argued in `templates/lefthook/README.md`.

## Decision record: merge queue — rejected

GitHub positions merge queues for branches "with a relatively high number
of pull requests merging each day from many different users". Solo with
Renovate branch-automerge, merges are serial in practice and Renovate
rebases stale branches before automerging, which already covers the
green-alone-red-together race the queue exists to catch. The costs are
concrete: a mandatory `merge_group` trigger in every workflow, no wildcard
branch-protection patterns, added latency on every merge. Revisit trigger:
a second regular contributor.

## Decision record: harden-runner — adopted, block mode

`step-security/harden-runner` is the **first step of every job** in every
workflow (before checkout — it cannot police what preceded it). It fills
the gap zizmor's static audit cannot see: runtime egress. A compromised
action or dependency exfiltrating tokens mid-build hits a closed socket,
not just an audit log.

- `egress-policy: block` with explicit per-job `allowed-endpoints`
  (`domain:443` lines; wildcards only where a service rotates subdomains).
  Audit mode is monitoring theatre — the max-enforcement bar is block.
  Allowlists are derived from one audit-mode run per job, then flipped to
  block in the same session; a blocked legitimate endpoint fails loud and
  is added deliberately.
- `disable-sudo: true` wherever the job never needs root; jobs that
  legitimately apt-get (edtf's pgrx matrix) keep sudo, recorded inline.
- Recorded trade-off: harden-runner ships runtime telemetry to
  StepSecurity's API (its own endpoints are auto-allowed). That is the
  price of runtime egress visibility; the action is digest-pinned and
  Renovate-tracked like everything else.
- Community-tier limits, verified empirically (2026-07-26): block-mode
  **enforcement is local and works on private repos** (monumental-archive
  runs enforced), but the insights dashboard needs the Enterprise app
  there — allowlist changes are derived from logs/failures instead. ARM
  runners (iiif-server's `ubuntu-24.04-arm` leg) are **not supported**:
  the step no-ops with a notice, so that leg is unenforced. Recorded
  fallback: block on x86, inert on arm — never drop the arm leg for it.
- Linux-only enforcement; all jobs run on ubuntu runners.

## Decision record: SBOM — adopted for shipped images; edtf exempt

`anchore/sbom-action` (syft) generates an SPDX-JSON SBOM for every
artifact a repo actually ships, at the job that builds it:

- **monumental-archive**: the two deployable images (db, cantaloupe),
  in a dedicated `sbom` job on pushes to main, with
  `dependency-snapshot: true` so GitHub's dependency graph — and therefore
  Dependabot alerts — covers what is inside the built images, not just the
  lockfiles. The snapshot submission needs `contents: write`, which is why
  it is a separate job with per-job elevation, never a widening of the ci
  job.
- **iiif-server**: no release pipeline exists yet; the SBOM step lands
  with it. Deferred, not exempt.
- **edtf**: exempt. It ships a crate; `Cargo.lock` plus crates.io metadata
  already give consumers the exact component inventory an SBOM would.

Format: SPDX-JSON (sbom-action default) — broadest ecosystem tooling;
CycloneDX adds nothing at this scale.

## Cache rules

- Exact key per input-hash (or per-sha for `.task` fingerprints) with
  prefix `restore-keys` fallback. Safe because caches are branch-scoped
  with default-branch fallback, and only trusted triggers (`push`,
  `schedule`, …) may write default-branch caches.
- **Never write secrets, tokens, or credentials to a cached path** — the
  poisoning-resistance above assumes cache contents are non-sensitive.
- `.task` fingerprint pattern is sound end-to-end: go-task writes
  checksums only after a task succeeds, `actions/cache` saves only on
  green jobs, and on restore task re-verifies every checksum — a failing
  gate can never be cached green; a stale entry simply re-runs.

## Canonical prologue

`prologue.yml` in this directory is the copy-source for the workflow-level
skeleton: triggers (`main` + `renovate/**` push — branch automerge needs
the check on the branch itself — `pull_request`, weekly cron where a gate
has a database input that changes without commits), `concurrency`
(cancel-in-progress for push gates; fuzz-style workflows doing useful work
queue instead), top-level `permissions: contents: read` with per-job
elevation only, and the harden-runner → checkout → mise-action step order.

## Sanctioned action set

All digest-pinned, digests updated by Renovate:

| action | why |
| --- | --- |
| `step-security/harden-runner` | runtime egress enforcement, first step of every job |
| `actions/checkout` | always `persist-credentials: false` |
| `jdx/mise-action` | toolchain from `mise.toml`, version Renovate-commented |
| `actions/cache` | per the cache rules above |
| `actions/upload-artifact` | reports and crash artifacts |
| `anchore/sbom-action` | SBOMs for shipped images |
| `docker/setup-buildx-action`, `docker/build-push-action` | monumental image builds, gha layer cache |

Any action outside this set is a per-repo decision recorded in the
workflow file. New actions join the table here first.
