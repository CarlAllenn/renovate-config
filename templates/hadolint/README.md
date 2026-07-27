# hadolint baseline

Canonical `.hadolint.yaml` — copy-template class, established in issue #8
(2026-07-27). The posture originated in monumental-archive (issue #4 era);
iiif-server's bench-harness Dockerfile made it a second-adopter tool, so it
canonizes here per the rule.

## Decisions

- **`failure-threshold: style`** — everything hadolint can find fails.
- **`disable-ignore-pragma: true`** — inline `# hadolint ignore=` pragmas
  are dead; every exemption lives in the config file with its reason, so
  the exemption inventory is one place per repo.
- **DL3005 ignored (canonical)** — `apt-get upgrade` keeps base-OS security
  patches current between base-image rebuilds; the stylistic objection
  loses to the patch posture.
- Per-repo ignores append after the canonical ones, each with a reason
  (e.g. iiif-server carries DL3008 for its Cantaloupe bench harness:
  Ubuntu's archive rotates point releases out, so pinned apt versions break
  rebuilds of a non-shipping image).

Consumers: monumental-archive (`.hadolint.yaml`, audited in
monumental-archive#68), iiif-server.
