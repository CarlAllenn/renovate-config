# cargo-machete baseline

Unused-dependency detection for the Rust repos (edtf, iiif-server),
established in issue #7. Every dependency is attack surface; an unused one
is attack surface bought for nothing. Copy-template class — there is no
config file to copy; the canon is the invocation and the escape mechanism.

## Invocation: `cargo machete --with-metadata`, everywhere

`--with-metadata` resolves real dependency names and per-build-type usage
via `cargo metadata --all-features` instead of the default regex heuristic.
The upstream warning that it "may modify Cargo.lock" was tested empirically
(2026-07-27, v0.9.2, both repos): with a committed, in-sync lockfile it
touches nothing — and it caught a genuinely dead dependency the heuristic
mode missed. Deterministic given the lock, so it is gate-class.

Wiring per Rust repo:

- **CI**: step in the gate (`task` target), run from the repo root so
  workspace-excluded trees (`fuzz/`, out-of-workspace crates) are scanned
  too — machete walks the directory, not the workspace graph.
- **lefthook** (repo-local `lefthook.yml`, not universal — stack-specific):
  pre-commit job globbed on `**/Cargo.toml`; dependency drift only enters
  through a manifest edit.

## False positives

`[package.metadata.cargo-machete] ignored = [...]` in the crate's own
Cargo.toml, always with a why-comment. The canonical legitimate case,
found on day one: a direct dependency declared only to force features onto
a transitive dependency (iiif-server pins `reqwest`'s TLS/http2 features
for `object_store`). An ignore without a reason is drift.
