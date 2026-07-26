# task baseline

Canonical per-tool baseline for [Task](https://taskfile.dev), established in
issue #2. This directory is **copy-template class**: consuming repos copy the
content in, and propagation of later changes is a deliberate per-repo
drift-audit — never automatic. Policy changes happen HERE first.

## Files

- `Taskfile.yml` — the canonical skeleton: header, `ci` umbrella, the
  universal lint layer (fingerprinted), per-stack placeholders. Copied in and
  adapted; the header, naming, and umbrella shape are non-negotiable, the
  per-stack tasks are the repo's own.

## Decision record: remote includes — rejected

Task's remote Taskfiles (experiment #1317) would give live propagation like
the Renovate preset. Rejected for now:

1. Gated behind `TASK_X_REMOTE_TASKFILES=1`; experiments are "subject to
   breaking changes and/or removal at any time" and production use is
   explicitly discouraged. A Task minor release could break all three repos
   at once — acceptable for `default.json` only because Renovate presets are
   a stable, first-class mechanism.
2. The trust/checksum prompts fight CI: non-interactive runs need `--yes`
   (unpinned = CI behavior changes with no commit in the consuming repo,
   breaking reproducibility and bisection) or pinned checksums (= a per-repo
   commit per change anyway — copy-template with extra indirection).
3. The shareable core is small: monumental-archive's Taskfile is almost
   entirely repo-specific; only the two Rust repos share a sizeable block.

Revisit if/when the experiment graduates to stable. (Lefthook — issue #3 —
has stable remote configs; that's where live propagation genuinely fits.)

## Rules

1. **Header.** `---` document start, `version: "3"`, and bash strict mode
   `set: [errexit, nounset, pipefail]` in every Taskfile. Section order per
   the upstream style guide: version → includes → settings → vars →
   env/dotenv → tasks.
2. **Naming.** `namespace:action` with colons (`lint:yaml`, `migrate:apply`),
   kebab-case task names, UPPERCASE vars, `desc:` on every task,
   `{{.VAR}}` without inner whitespace.
3. **`ci` is the umbrella** — the full gate, identical locally and on
   GitHub; the workflow runs `task ci` and nothing else decides what CI
   means. Independent gates run as parallel `deps:`; ordered stages (image
   builds, smoke, scans of built artifacts) follow as serial `cmds:`.
4. **One `lint:<tool>` task per tool**, with `lint` as a `deps:` umbrella:
   parallel for free, individually addressable, and the unit fingerprinting
   attaches to. Same for `fmt` (apply mode) mirroring the check-mode tasks.
5. **Fingerprint every gate** (`method: checksum` + `sources:`), per
   monumental-archive ADR 0014 / issue #40:
   - sources = the gate's file class **+ the tool's own config + the pinned
     toolchain** (`mise.toml`, `mise.lock`);
   - err on over-triggering — a missed source is a false skip and treated
     as a bug;
   - **never fingerprint security/secret scanners** — trivy fs,
     `scan:bases`-class policy gates, AND vulnerability scans of built
     artifacts: they must look at whatever changed, whatever it is, and CVE
     scans have an input (the vulnerability database) no source checksum
     can see. Repos with an always-on scan family add a weekly `schedule:`
     trigger to the ci workflow so new CVEs surface during quiet weeks;
   - Task writes a checksum only after the task succeeds, and CI caches
     save only on green jobs — a failing gate can never be cached green;
   - `task --force` and an uncached CI run are the escape hatches;
   - tasks that produce artifacts a fingerprint can't see (e.g. Docker
     images that survive in `.task` but not in a pruned daemon) pair
     `sources:` with a `status:` existence check — both must pass to skip.
6. **CI fingerprint cache.** Each consuming repo's ci workflow caches
   `.task` keyed on `task-fingerprints-${{ github.sha }}` with a
   `task-fingerprints-` restore-key: always saved on green, and a partial
   hit is safe because Task re-verifies every checksum against the actual
   sources.
7. **`run: once`** on build-ish tasks that multiple gates dep on (image
   builds, tool bootstraps): at most one execution per invocation.
   **`output: prefixed`** at the root: parallel deps interleave otherwise.
8. **Safety idioms.** `prompt:` on destructive tasks, `preconditions:` with
   `msg:` for environment assumptions, `requires: vars:` for parameterized
   tasks, `interactive: true` for TTY tasks.
9. **Prefer external scripts** (`scripts/*.sh`, themselves shellcheck'd)
   over inline blocks once a command exceeds ~5 lines, per the upstream
   style guide.

## De-drift checklist (run per repo after a baseline change)

1. Header block matches `Taskfile.yml` here verbatim.
2. `ci` umbrella exists and the workflow runs `task ci`; independent gates
   are parallel deps.
3. Every universal mise tool has its `lint:<tool>` task; every gate is
   fingerprinted per rule 5; no fingerprint on security scanners.
4. The `.task` cache step exists in the ci workflow (rule 6).
5. `task --list` output reads as `namespace:action` with a desc per task.
