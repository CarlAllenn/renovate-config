# trivy baseline

Canonical trivy configuration and invocations — copy-template class,
established in issue #15. Before this, three configurations existed across
two repos, and the weakest (iiif-server's CRITICAL,HIGH floor) was weakest
for the worst reason: written without looking at what monumental-archive
already did. One max-enforcement shape, consumed identically, so no repo
can quietly miss a scanner. Posture originated in monumental-archive.

## The two invocations

**Filesystem scan** — the repository tree, with `trivy.yaml` from this
directory at the repo root:

```text
trivy fs --config trivy.yaml .
```

All four scanners (vuln, misconfig, secret, license), all severities,
exit-code 1. License policy governs OUR manifests here — which is exactly
why the image scan drops it.

**Image scan** — an image this repo builds:

```text
trivy image --config /dev/null --scanners vuln --ignore-unfixed \
  --exit-code 1 --severity UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL <image>
```

`--config /dev/null` is load-bearing: without it trivy silently reads the
repo's `trivy.yaml` and the fs-scan policy (license gating, skip-dirs)
leaks into the image gate.

## Decisions

- **`--ignore-unfixed` on image scans, all severities.** An OS-layer image
  otherwise fails on CVEs with no available patch — unfixable, therefore
  ungateable, therefore noise that trains people to ignore the gate
  (ADR 0003 class). Failing on everything that HAS a fix, at every
  severity, is still max enforcement of the actionable set, and it is a
  no-op for a `FROM scratch` image, so it costs nothing to standardise.
  The fs scan does NOT take the flag: lockfile-level fixes are always
  actionable (bump the dep), so the full set gates there.
- **Scanner split: fs gets everything, images get vuln only.** License
  policy and secret/misconfig detection govern our manifests and tree via
  the fs scan; the image gate is package CVEs. The split is deliberate,
  not an accident of one script.
- **Gate images you BUILD; images you PULL are report-only.** A fixable
  finding in an image we build is actionable (rebuild resolves it, or a
  pin bump does). A "fixable" finding in a pulled image means the distro
  fixed it but the publisher has not rebuilt — nothing a gate of ours can
  express. Measured in monumental-archive#165 (2026-08-04): under the
  canonical invocation the pulled tier gates red on 3 of 4 images
  (postgrest ~907 fixable findings, martin 46, electric 15, redis 0) with
  zero available actions. Pulled tiers get a scheduled report-only scan
  (weekly, `scheduled.yml` class — the input is the advisory database and
  upstream rebuilds, which change without commits), never a gate.
- **Where the gate belongs: pull requests, not the release path.** The
  image gate runs in CI against the built candidate. A CVE published
  tomorrow must not retroactively fail today's release, and a release must
  not die because a scanner had a bad day — which is how iiif-server's
  v0.1.0 first failed. Publish workflows may scan in their prove step as
  report-only; the red that blocks a merge is the PR's.
- **Never fingerprint a CVE scan.** Its inputs include the trivy
  vulnerability database, which changes daily — a source-checksum skip is
  unsound for a security gate. Both adopting repos got this right
  independently; recorded here once. (Same reasoning puts the scan family
  in any weekly full-gate cron a repo runs.)
- **No trivy DB cache in CI.** Measured in monumental-archive#150: the DB
  is stale after 6h, so a weekly cache key guarantees a miss — ~53s of
  restore/save overhead and 6.9GB of cache budget per repo, to avoid a
  3.7s download. Let every run download it fresh.
- **mise pin moves in lockstep.** All consumers pin the same trivy version
  (0.72.0 at canonization); Renovate bumps land per-repo but the version
  is not a per-repo decision.

## False positives / exemptions

Same rule as `.hadolint.yaml`: no inline suppressions in invocations, no
`.trivyignore` files (monumental-archive's `check-no-waivers.sh` bans them
by name, correctly). Anything a repo genuinely cannot satisfy is an entry
in its `trivy.yaml` with a recorded reason — and an unfixable finding in
an image is remediated by owning the installation (monumental-archive
ADR 0030), not by waiving it.

## Consumers

monumental-archive (fs scan; db + iiif-server image scans), iiif-server
(scratch image — floor to be raised from CRITICAL,HIGH on next touch),
edtf (pending edtf#82 container publish), monumental-archive-db (pending —
monumental-archive#167).
