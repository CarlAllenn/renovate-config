# Rust lint baseline (rustc + clippy)

Canonical `[workspace.lints]` block in `Cargo-lints.toml` — paste into each
Rust repo's workspace `Cargo.toml`, with `[lints] workspace = true` in every
member crate. Copy-template class; established in issue #8 (2026-07-27).

## Decisions

- **Wholesale `all`/`pedantic`/`nursery`/`cargo` at warn, CI escalates with
  `-D warnings`.** Recorded decision against the clippy book's advice to
  cherry-pick pedantic/nursery: iiif-server ran full pedantic with zero
  allows before this baseline existed, so wholesale is proven at our scale.
  Warn-in-manifest keeps local `cargo build` output informative without
  blocking iteration; the hook and CI make it a gate.
- **`restriction` cherry-picked** — the group contains mutually
  contradictory lints and clippy itself warns against enabling it whole.
  Each opt-in is commented in the template. Additions/removals happen here
  first, then propagate.
- **`clippy.toml` per repo** carries `msrv` (derived from the workspace
  `rust-version` — single source of truth, same derivation as the MSRV CI
  job) and `avoid-breaking-exported-api = false` while a repo is pre-1.0
  (flip to true at v1; the semver gate is cargo-semver-checks' job, not
  clippy's).
- **Per-crate re-allows** must use
  `#[allow(lint, reason = "...")]` / `[lints]` overrides with a comment —
  `allow_attributes_without_reason` fails anything else.
- **Known-noisy lints stay in until they misfire.** `single_use_lifetimes`
  and `explicit_outlives_requirements` have false-positive history upstream;
  the max-enforcement order of operations is in-first, individually reasoned
  out only on an actual false positive, recorded in the template.
