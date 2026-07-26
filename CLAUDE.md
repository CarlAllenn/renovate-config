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
- `templates/` — canonical per-tool baselines (mise done — issue #1; next:
  task, lefthook, linters, supply-chain, CI — issues #2–#6). Copy-template
  class: propagation to repos is a deliberate drift-audit, not automatic.

## Working rules

- Any change to `*.json` presets must pass
  `renovate-config-validator --strict` — CI (`validate.yml`) enforces this,
  but run it locally before pushing; a broken preset breaks Renovate in
  every consuming repo at once.
- The repo name (`renovate-config`) and `default.json` filename are
  load-bearing: they trigger auto-suggestion when onboarding new repos.
  Don't rename either.
- The roadmap for building out `templates/` is issues #2–#7, in order
  (#1, mise, is done — the session that set the method).
  Method per tool: read the docs end to end → decide with Carl → canonize
  here → de-drift the three repos.
- Consuming repos' CI must trigger on push to `renovate/**` branches
  (branch automerge needs the check on the branch). Keep the README's
  CI-prerequisite section accurate if policy changes.
