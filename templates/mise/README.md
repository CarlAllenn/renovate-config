# mise baseline

Canonical per-tool baseline for [mise](https://mise.jdx.dev), established in
issue #1. This directory is **copy-template class**: consuming repos copy the
content in, and propagation of later changes is a deliberate per-repo
drift-audit — never automatic. Policy changes happen HERE first.

## Files

- `settings.toml` — the canonical `min_version` + `[settings]` block. Copied
  verbatim to the top of every repo's `mise.toml`.

## Rules

1. **No ad-hoc tool invocations.** Anything executed gets pinned in
   `mise.toml` first. `locked = true` enforces this mechanically: installs
   resolve only from `mise.lock`, so an unpinned `mise x tool@ver` or a CI
   `cargo install`/`npx`/`uvx` has no sanctioned path. Sanctioned exceptions:
   - **MSRV check jobs** (`rustup toolchain install $MSRV`): deliberately a
     second toolchain outside mise; the version is tracked in a workflow env
     var or Cargo.toml `rust-version`, bumped manually (MSRV bumps are policy
     decisions, per the Renovate preset).
   - **Nightly toolchains for fuzzing**: unpinnable by nature; confined to
     the fuzz workflow.
   - **SHA-pinned, digest-verified fixture installs** (e.g. iiif-server's
     validator: git SHA pin + sha256-verified assets via pinned `uv`).
2. **Backend policy** (in order of preference):
   - registry short names resolving to **aqua** or **core** — checksums +
     provenance (cosign/SLSA/GitHub attestations) verified natively;
   - **`github:`** for GitHub-release binaries not in the aqua registry;
   - **`http:`** only for vendors outside GitHub, and always with
     `checksum_url` (or explicit per-platform `checksum`) so `mise lock`
     records checksums for **all** `lockfile_platforms`, not just the machine
     that ran it;
   - **`pipx:`/`npm:`** for ecosystem CLIs. Lock entries are version-only
     (no checksums) — compensating controls: 72h `minimum_release_age` is
     forwarded to transitive deps (uv `--exclude-newer`, aube), and aube
     denies dependency lifecycle scripts by default;
   - **`cargo:`** for Rust CLIs with no prebuilt release (source build with
     `--locked`);
   - **`conda:`** fixtures-only carve-out (see iiif-server
     `tools/fixtures/mise.toml`): the conda backend cannot write cross-platform
     lock entries, so that config root sets `lockfile = false` + `locked = false`
     with exact pins and the 72h cooldown as compensating controls.
   - **never `asdf:` or `ubi:`** — disabled via `disable_backends`.
3. **CI prerequisites.**
   - `jdx/mise-action` pinned by digest **and** mise itself pinned via the
     `version:` input with a `# renovate:` comment (tracked by this repo's
     custom manager). Every workflow that installs tools uses mise-action —
     including scheduled/auxiliary workflows.
   - Locked mode means `mise.lock` must contain URL entries for every CI
     platform; `lockfile_platforms` guarantees `mise lock` produces them.
   - Consuming repos' CI triggers on push to `renovate/**` branches (branch
     automerge; see repo README).
4. **Every pinned tool is wired.** A tool in `mise.toml` that no task, hook,
   or workflow invokes is drift in the other direction — remove it or wire it.

## Tool split

Universal enforcement layer (every repo, regardless of stack — "every text
format has a linter, warnings are errors"):

| Tool | Backend (via registry) | Purpose |
| --- | --- | --- |
| `task` | aqua | task runner (Taskfile) |
| `lefthook` | aqua | git hooks |
| `actionlint` | aqua | workflow lint |
| `zizmor` | aqua | workflow security audit |
| `shellcheck` | aqua | shell lint |
| `shfmt` | aqua | shell format |
| `yamllint` | pipx | YAML lint |
| `taplo` | aqua | TOML fmt + lint |
| `editorconfig-checker` | aqua | editorconfig conformance |
| `pipx:codespell` | pipx | spelling |
| `node` + `npm:markdownlint-cli2` | core + npm | Markdown lint |
| `uv` | aqua | python tool installer (backs `pipx:` tools) |

Per-stack additions (repo-local, in that repo's `mise.toml`):

- **Rust**: `rust` (with `components = "rustfmt,clippy"`, plus `targets` for
  cross targets like wasm), `aqua:EmbarkStudios/cargo-deny`, and `cargo:`
  build tools the CI needs (e.g. `cargo-pgrx`, `cargo-fuzz`, `wasm-pack`).
- **Containers / deploy** (monumental-archive): `hadolint`, `trivy`,
  `cosign`, `ruby` (kamal via Gemfile), `go` (demesne build), `http:atlas`.
- **Python**: `ruff`, `pipx:sqlfluff`.
- **TypeScript**: `pnpm`, `biome` (formatter only).

## De-drift checklist (run per repo after a baseline change)

1. Copy `settings.toml` content over the repo's `min_version` + `[settings]`.
2. `mise lock` — regenerate for all `lockfile_platforms`; commit `mise.lock`.
3. Confirm every universal tool is present *and wired* (Taskfile lint target
   or lefthook command).
4. Grep CI/Taskfile/lefthook/scripts for ad-hoc installs
   (`cargo install|npx|uvx|pipx run|go install|curl .*sh`) — anything found
   is either pinned into mise.toml or documented as a sanctioned exception.
5. `mise install` from a clean cache must succeed in locked mode.
