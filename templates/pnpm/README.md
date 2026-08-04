# pnpm supply-chain baseline

Canonical `pnpm-workspace.yaml` supply-chain block for pnpm repos (currently
only `monumental-archive`). Copy-template class: repos copy the block from
[pnpm-workspace.yaml](pnpm-workspace.yaml) below their `packages:` list;
propagation is a deliberate drift-audit, never automatic. Established in
issue #5 (2026-07-26), from an end-to-end read of the pnpm v11 settings
reference.

## The three-layer 14-day policy

One number — **7 days** — enforced at three independent points:

| Layer | Setting | Where |
| --- | --- | --- |
| Renovate (bump time) | `minimumReleaseAge: "7 days"` | this repo's `default.json` |
| pnpm (install time) | `minimumReleaseAge: 10080` | consuming repo `pnpm-workspace.yaml` |
| mise (install time) | `minimum_release_age = "168h"` | consuming repo `mise.toml` |

The numbers must stay equal. pnpm re-verifies **lockfile entries** on every
install, so a pnpm gate stricter than the Renovate gate makes fresh Renovate
branches fail resolution until the stricter clock expires; a looser one is a
bypass. Security releases route around the gate deliberately: Renovate OSV
alerts ignore `minimumReleaseAge`, and the local twin is a commented
`minimumReleaseAgeExclude` entry.

## Maximum-enforcement posture

Every setting in the block is at its strictest value. pnpm 11 ships several
of them as defaults (`strictDepBuilds`, `blockExoticSubdeps`,
`verifyStoreIntegrity`, `strictStorePkgContentCheck`, `trustLockfile: false`);
they are pinned explicitly so a future default change or downgrade cannot
silently weaken the posture.

Deliberate deviations from the theoretical maximum, with reasons:

- **`trustPolicyIgnoreAfter: 525600` (1 year)** — softens `trustPolicy:
  no-downgrade` for versions published over a year ago. The no-downgrade
  check targets account-takeover releases, which are discovered within days;
  a year-old release flagged only because it predates provenance is noise.
  This replaces per-package `trustPolicyExclude` entries (it retired
  monumental-archive's `semver@6.3.1` adjudication) and is strictly narrower
  than the alternative, which is an unbounded exclude list.
- **No `minimumReleaseAgeExclude` / `allowBuilds` entries by default** — both
  stay commented-out templates. Any entry added in a repo needs an inline
  adjudication comment (what, why, date).

## Not in the block

`packages:`, catalogs, hoisting, and linker settings are repo-local layout
concerns, not policy — they stay in each repo's file, outside the canonical
block.
