# Release engineering baseline (release-plz)

Canonical release pipeline for publishable Rust repos — copy-template class,
established in issue #8 (2026-07-27). First consumer: edtf.

> **Status note (2026-08-04).** iiif-server's deferral ended: v0.1.0
> shipped — but NOT on release-plz, which cannot package interdependent
> workspace crates absent from a registry (release-plz#2595); it releases
> via git-cliff + gh instead, recorded in its docs/release-engineering.md.
> This template remains the canon for registry-publishing Rust repos
> (edtf); workspace-inherited, unpublished-crate repos follow iiif's
> pattern. The CI SBOM job the parenthetical below described was removed
> the same day (SBOM posture: templates/publish/README.md — attach to what
> you publish, scan nothing). Cross-repo publish invariants live in
> templates/publish/.

## Decisions

- **Tool: release-plz** over cargo-release (manual-first, laptop-shaped —
  the thing being eliminated) and cargo-dist (binary/installer distribution,
  a complement not a driver; alive upstream, v0.32.0 May 2026).
- **The Release PR is the commitment point** (doctrine change from edtf#32's
  tag-as-commitment, agreed 2026-07-27). release-plz maintains a PR on main
  previewing version bumps, changelog, and cargo-semver-checks results;
  merging it publishes everything, tags, and cuts the GitHub release.
  `release_always = false` so nothing publishes outside that path.
- **Trusted publishing everywhere, no long-lived tokens** (posture from
  #5/#7): crates.io via `rust-lang/crates-io-auth-action` (`id-token:
  write`, no CARGO_REGISTRY_TOKEN secret); npm via trusted publishers
  (CLI ≥ 11.5.1, provenance automatic on public packages). Account-side
  trusted-publisher registration is per-crate/package, owner-performed.
- **Artifact attestations** (`attestations: write` + `id-token: write`) on
  everything published from the release job.
- **npm publishes chain off release-plz outputs** (`releases_created`, the
  `releases` array) in the same workflow — documented release-plz pattern,
  no parallel release path.
- **Nested workspaces** (edtf-postgres, fuzz/) are outside release-plz's
  workspace: publish as a chained step ordered last (it resolves the core
  crates from the registry), or a second `--manifest-path` invocation —
  whichever the first execution proves cleaner; record the outcome here.
- **Unified version** via `version_group` on every publishable crate; the
  changelog is root-level keep-a-changelog.
- **Tag immutability**: once publishing is automated, a repository ruleset
  forbids tag deletion and moves. The attestation story is only as strong
  as the tag it points at.

## Workflow shape

Two jobs per the release-plz two-job pattern, both opening with the #6
harden-runner block canon:

1. `release-plz-pr` — `release-plz release-pr`; `contents: write` +
   `pull-requests: write`; concurrency-guarded per branch. Needs a PAT (or
   GitHub App token) so the Release PR's own CI runs — `GITHUB_TOKEN`-created
   PRs don't trigger workflows (same class of constraint as the
   renovate-branch CI prerequisite in the root README).
2. `release-plz-release` — `release-plz release`; `contents: write`,
   `id-token: write`, `attestations: write`; deliberately NOT
   concurrency-guarded (a queued release must not be skipped). Chained
   steps: crates publish (trusted publishing) → nested-workspace publish →
   wasm-pack build + npm publish (gated on `releases_created`) →
   attestations over published artifacts.

Consumers copy `release-plz.toml` (filling PER-REPO sections) and implement
the workflow against their own audit-derived harden-runner allowlists.
