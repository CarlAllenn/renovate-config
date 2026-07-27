# lychee baseline

Link checking for every repo, established in issue #7 — every text format
has a linter except links, which rot silently. Copy-template class:
`lychee.toml` is copied to each repo root, drift-audited deliberately.

## Class: scheduled, never a gate

Link state is the never-fingerprint class — the input is the public
internet, which changes without commits. lychee runs only in the weekly
`scheduled.yml` (see `templates/ci/README.md`), never pre-commit and never
in the push gate. This also quarantines its egress: a link checker's job
is arbitrary outbound traffic, so its CI job runs harden-runner in audit
mode — acceptable only because it is isolated in a workflow where no
sensitive job shares the runner.

## Config decisions (see comments in lychee.toml)

- `include_fragments = "full"` — broken anchors are rot too; max tier.
- `require_https = true` — validated clean across all four repos'
  link corpus on 2026-07-27; an http:// link where https:// works is a
  finding.
- private/loopback/link-local excluded — not checkable, not probeable
  from CI.
- `cache = false` — the weekly cadence is the cache; a cached OK defeats
  a rot check.
- `GITHUB_TOKEN` from the workflow env (default token) so github.com
  links check authenticated instead of rate-limited.

## Escapes and surfacing

Per-link escapes go in the repo's `.lycheeignore`, one why-comment per
entry. Failure surfacing is the red scheduled run plus GitHub's
workflow-failure notification — deliberately no issue-filing action, which
would add an action to the sanctioned set for notification plumbing a solo
maintainer gets anyway.
