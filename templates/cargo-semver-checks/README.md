# cargo-semver-checks baseline

Semver-honesty gate for published crates — currently the edtf workspace
only — established in issue #7. Catches accidental API breaks at the PR
that introduces them, not at release time.

## Invocation and class

`cargo semver-checks --workspace` in edtf's `ci.yml` push gate, as its own
job (no `.task` fingerprint cache: the baseline is registry state, not
repo content, so a content-hash skip could hide a stale verdict).

Gate-class despite the network fetch: the baseline is the **latest
published version on crates.io**, which changes only when Carl publishes —
identical verdicts between releases. `index.crates.io`/`static.crates.io`
join that job's harden-runner allowlist (already present wherever cargo
builds). Exit 100 = semver violation; exit 101 = check could not complete —
both red.

## Scope

- **Checked**: every workspace member with a lib target — edtf-core,
  edtf-calendars, edtf-normalize, edtf-wasm (bin-only edtf-cli has no API
  surface; the tool skips it itself).
- **edtf-postgres: exempt, recorded.** Its contract is SQL, not a Rust
  API; it lives outside the workspace and rustdoc'ing it needs an
  initialized pgrx toolchain. Nobody consumes it via cargo.

## Features

Default heuristic (all features except `unstable`/`nightly`-style names) —
no flags. For edtf this correctly includes the `serde` features. Revisit
only if a feature is added that the heuristic misclassifies.

## Version-bump semantics

A major bump (including 0.x → 0.(x+1)) licenses breaks — the issue #32
unification to 1.0.0 passes the gate by construction. The gate's job is
the *unintended* break: an API change without the matching bump.
Per-lint overrides (`[package.metadata.cargo-semver-checks.lints]`) are
allowed only with a recorded reason, same rule as every disabled lint.

## Fallback baseline

If registry lookups misbehave, `--baseline-rev <last-release-tag>` is the
documented fallback (tags match publishes from v0.2.0 onward). Not the
default: the registry is the contract consumers actually see.

## Baseline run

2026-07-27, v0.49.0: four crates checked against the 2026-07-26 publishes,
196 checks each, all pass, ~40s total workspace wall-clock.
