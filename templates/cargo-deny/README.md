# cargo-deny baseline

Canonical `deny.toml` for the Rust repos (`edtf`, `iiif-server`).
Copy-template class: repos copy [deny.toml](deny.toml) and fill the PER-REPO
sections; propagation is a deliberate drift-audit, never automatic.
Established in issue #5 (2026-07-26), from an end-to-end read of the
cargo-deny book (all four check chapters, graph config, CLI reference).

## Maximum-enforcement posture

Every lint that can be `deny` is `deny`, every allowlist is exact-tree-only
and fail-closed, and every unused-entry lint (`unused-allowed-license`,
`unused-allowed-source`, `unused-ignored-advisory`, …) is `deny` so config
can never describe a tree that no longer exists. The one lint that cannot be
expressed in `deny.toml` — `unmatched-skip`, a warning by default — is
promoted in CI: the check must run as

```bash
cargo deny check -D unmatched-skip
```

Advisories run under v2 semantics: all vulnerability/unsound/notice
advisories are unconditional errors; the only opt-out is an `ignore` entry
with an inline reason, and stale ignores themselves fail.

## Deliberate deviations from the theoretical maximum

- **`external-default-features` left at default (`allow`)** — deny would
  require per-crate feature allowances for essentially the entire tree;
  it is feature hygiene, not a supply-chain control, and the noise would
  drown real findings.
- **`[licenses] include-dev` left at default (`false`)** — license risk is a
  distribution concern and dev-dependencies don't ship in artifacts. (Bans,
  advisories, and sources DO cover dev-deps: `multiple-versions-include-dev`
  is on, and the advisory/source checks are graph-wide.)
- **`allow-wildcard-paths = true`** — workspace-internal path deps are how
  Cargo spells intra-repo structure; crates.io mechanically forbids
  publishing wildcard path/git deps, so the relaxation cannot leak.

## PER-REPO sections

- `licenses.allow` — exactly the SPDX identifiers in the tree today, each
  addition commented with the crate(s) needing it (`unused-allowed-license =
  "deny"` makes shrinkage mechanical).
- `bans.build.allow-build-scripts` — enumerate every crate with a build
  script from the lockfile, each with a review comment. The Rust twin of
  pnpm's `allowBuilds`.
- `bans.skip` / `advisories.ignore` / `sources.allow-git` — temporary by
  construction; every entry carries a reason and, for forks, the tracking
  issue that removes it.
