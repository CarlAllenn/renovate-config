# gitleaks baseline

Universal secret scanning, established in issue #7. Before this, secret
detection was the one enforcement class with zero universal coverage
(monumental-archive got it incidentally via trivy; edtf and iiif-server had
nothing between a pasted token and `git push`). Copy-template class for the
CI job; the pre-commit job propagates live via `lefthook/universal.yml`.

## Two layers, both deterministic

1. **Pre-commit** (`lefthook/universal.yml`): upstream's own hook
   invocation — `gitleaks git --pre-commit --redact --staged --verbose`.
   No glob: any file type can carry a secret.
2. **CI** (`ci.yml` job in every repo): full-history scan — `gitleaks git`
   over the whole clone (`fetch-depth: 0`). History scans are cheap at this
   scale (the largest repo scans in under a second) and re-apply each
   pinned ruleset to every commit, so a rule added in an upgrade still
   catches an old leak.

Both layers are gate-class, not scheduled-class: the ruleset ships inside
the mise-pinned binary, so identical inputs give identical verdicts.

## Configuration: none, deliberately

The bundled default ruleset is the baseline; there is no canonical
`.gitleaks.toml`, and repos must not add one to weaken rules. False
positives use, in order of preference:

- inline `#gitleaks:allow` on the flagged line — the reason lives at the
  site (add a comment saying why it is not a secret);
- a `.gitleaksignore` fingerprint entry (for findings in history, which no
  inline comment can reach) — every entry gets a why-comment above it.

A real leak is never ignored; it is rotated, then the rotated finding's
fingerprint is recorded as triaged.

## Decision record: trivy overlap in monumental-archive — double-cover

trivy's secret detection stays on. The surfaces differ: gitleaks reads git
history; trivy reads the working tree and built images (a secret baked into
an image layer never appears in a diff). Revisit trigger: sustained
duplicate-finding noise, which at that point is deduped by scoping trivy's
secret scanner to images only — never by dropping either tool.

## Decision record: gitleaks over Betterleaks

gitleaks is feature-complete upstream (security patches only); the same
authors now build [Betterleaks](https://github.com/betterleaks/betterleaks)
as its successor. Betterleaks was rejected for now: no stable release, not
in the aqua registry, different CLI surface. Revisit trigger: a Betterleaks
stable release appearing in the aqua registry.

## Baseline scan

Full-history scans of all four repos (renovate-config, edtf, iiif-server,
monumental-archive) ran clean on 2026-07-27 with gitleaks 8.30.1 — the
baseline starts from zero findings, so any future finding is new, not
backlog.
