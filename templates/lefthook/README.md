# lefthook baseline

Canonical baseline for [lefthook](https://lefthook.dev) git-hook
orchestration, established in issue #3. Unlike the other tool baselines,
this one is **live-propagation class**, not copy-template: the shared job
set lives at `lefthook/universal.yml` in this repo's root and is consumed
by every active repo via lefthook's first-class `remotes:` mechanism — a
change there reaches every dev machine within 24 hours, no per-repo commit.

## Decision record: remotes — adopted

Task's remote includes were rejected in `templates/task/README.md`; lefthook
`remotes:` is the opposite case on every axis that mattered there:

1. Stable, first-class feature (in lefthook since 1.x, refetch semantics
   hardened and documented through 2.0.16) — not an experiment behind an
   env flag.
2. No CI trust surface: git hooks run on dev machines only; CI enforcement
   in the consuming repos is `task ci`, which never invokes lefthook. An
   unpinned remote can therefore never change CI behavior or break
   reproducibility/bisection.
3. Graceful offline behavior: a failed fetch is a warning and the cached
   copy is used; only a first-ever install with no network ignores the
   remote (and `task ci` still gates the result).

Configuration: no `ref` (track the default branch — pinning a tag would
mean a per-repo bump commit per change, i.e. copy-template with extra
indirection) and `refetch_frequency: 24h`. The residual risk — a bad push
to main propagating everywhere — is bounded by the `lefthook validate` job
in this repo's CI and pre-commit hooks.

## Consumer skeleton

A consuming repo's `lefthook.yml` contains only the remote reference and
its own per-stack jobs:

```yaml
---
remotes:
  - git_url: https://github.com/CarlAllenn/renovate-config
    refetch_frequency: 24h
    configs:
      - lefthook/universal.yml

pre-commit:
  parallel: true
  jobs:
    # per-stack jobs only (rust: fmt/clippy/deny; python/ts: ruff/biome/...)

pre-push:
  jobs:
    # per-stack gates (rust: cargo test; containers: trivy/base-images)
```

## Rules

1. **Merge order is load-bearing.** `lefthook.yml` → `extends` → remotes →
   `lefthook-local.yml`. A repo *cannot* override a universal job from its
   own `lefthook.yml`; only a personal `lefthook-local.yml` (gitignored
   everywhere, never committed) can — and the policy is: comply with the
   tools; fixing output is never optional.
2. **No silent skips.** `assert_lefthook_installed: true` (a missing binary
   fails the hook instead of running nothing) and a `[hooks]`
   `postinstall = "lefthook install"` in every repo's `mise.toml` (a fresh
   clone gets its git hooks installed by the standard `mise install`
   bootstrap; lefthook auto-resyncs on every run thereafter).
3. **Formatters fix, never just complain:** write mode + `stage_fixed: true`
   (safe as of lefthook 2.1.7, hence `min_version`). Pure linters run on
   `{staged_files}` wherever the tool accepts file lists.
4. **Universal set** (pre-commit, parallel): shfmt, shellcheck, yamllint,
   taplo, markdownlint, codespell, editorconfig (ec), actionlint, zizmor —
   plus conventional-commit enforcement on commit-msg (inlined in
   universal.yml; remote `scripts:` would require a root `.lefthook/` dir
   here). Tool versions are pinned per-repo via mise; tool configs are the
   conventionally shared dotfiles (canonization of those: issues #4–#5).
5. **Escape hatch is explicit and single-use:** `LEFTHOOK=0 git commit`
   exists for emergencies; CI (`task ci`) remains the backstop that catches
   anything bypassed locally.

## Ops notes

- `lefthook dump` prints the merged effective config (main + remote +
  local); `lefthook validate` checks it against the JSON schema.
- Remote cache lives under `.git/info/lefthook-remotes/`; `lefthook install`
  fetches it initially.
- Renovate tracks the lefthook version itself via each repo's mise pin.
