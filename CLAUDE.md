# renovate-config

Shared config baseline for all of Carl's active repos: `monumental-archive`,
`edtf`, `iiif-server` (the only repos in Mend Renovate — deliberately; never
onboard others without being asked). `sovereign-archive-db` is deprecated and
off-limits.

## What lives here

- `default.json` — the shared Renovate policy. Every active repo extends it
  via `github>CarlAllenn/renovate-config`. Policy changes happen HERE, never
  in per-repo `renovate.json` files (those hold only repo-local rules).
  Decision record: ADR 0018 in monumental-archive (`docs/decisions/`).
- `templates/` — canonical per-tool baselines (mise, task, lefthook,
  linters, supply-chain, CI, scanners done — issues #1–#7; #8 covers
  non-universal linters per-repo). Copy-template class:
  propagation to repos is a deliberate drift-audit, not automatic. The
  linter bar is maximum enforcement — every rule on unless
  `templates/linters/README.md` records why not.
- `lefthook/universal.yml` — the shared git-hook job set, consumed LIVE by
  the active repos via lefthook `remotes:` (no ref, 24h refetch). This path
  is load-bearing: renaming it breaks hook updates in every consuming repo.
  Decision record: `templates/lefthook/README.md`. Changes must pass
  `lefthook validate` (CI enforces; pre-commit hook runs it locally).

## Working rules

- Any change to `*.json` presets must pass
  `renovate-config-validator --strict` — CI (`validate.yml`) enforces this,
  but run it locally before pushing; a broken preset breaks Renovate in
  every consuming repo at once.
- The repo name (`renovate-config`) and `default.json` filename are
  load-bearing: they trigger auto-suggestion when onboarding new repos.
  Don't rename either.
- The roadmap for building out `templates/` is issue #8 (non-universal
  linters, per-repo); #1–#7 are done. Method per tool:
  read the docs end to end → decide with Carl → canonize here → de-drift
  the three repos fully in the same session (implementation included, not
  just the pattern).
- Consuming repos' CI must trigger on push to `renovate/**` branches
  (branch automerge needs the check on the branch). Keep the README's
  CI-prerequisite section accurate if policy changes.
