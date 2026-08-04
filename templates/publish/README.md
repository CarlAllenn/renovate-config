# Publishing baseline

Canonical invariants for every repo that publishes artifacts — copy-template
class. Canonized when the second image publisher arrived
(monumental-archive-db, monumental-archive#167) and re-proved what
iiif-server's release engineering had already paid for; third adopter is
edtf's container leg (edtf#82). The *release pipeline* (versioning, Release
PRs, changelogs) is `templates/release/`; this directory is about what any
publish run must do and know, whatever drives it.

## Two publish models — pick by artifact

- **Release-driven** (iiif-server, edtf): a version is decided, a tag is
  cut, phase 2 publishes *from the tag* so attestations name real bytes.
  For anything with a semver surface.
- **Push-and-cron** (monumental-archive-db): publish on main pushes to the
  artifact's inputs, plus a weekly `--no-cache` scheduled rebuild. For
  repackaging images with no version surface of their own — their
  "version" is a set of pins, their changes are pin bumps, and the weekly
  rebuild IS the OS-layer remediation (blocking it would preserve
  exposure). Tags are `:main` + `:sha-<commit>`; consumers pin digests and
  Renovate resolves against `:main`.

Do not give a repackaging image Release-PR ceremony, and do not let a
versioned artifact publish from a branch ref — an attestation recording
`refs/heads/main` tells a verifier nothing about which bytes were signed
(edtf's v1.0.0 attestations are permanently wrong this way; Sigstore is
append-only).

## The publish invariant (order is not rearrangeable)

> build → smoke → push → **pull the published bytes back by digest and
> prove them again** → sign → **verify the way a stranger would**

- Native hardware per architecture. No QEMU, ever — emulation tests the
  image on the wrong machine. `fail-fast: false` so one leg's failure
  leaves the other as arch-specificity evidence.
- Build `--load` and smoke BEFORE any push: a combined build-and-push has
  no moment where a testable artifact exists un-published, and a digest
  anyone has pulled exists forever.
- After the manifest exists, pull it back BY DIGEST and re-prove: the
  signature must assert something demonstrated of what a stranger pulls,
  not of a local twin.
- Sign last, and only on proof. Then verify the signature exactly as a
  consumer would (`cosign verify` against the workflow identity) — a
  signing step that succeeded is not a signature that verifies.
- Publish workflows carry **no cache of any kind** (cache-poisoning
  posture; the ci.yml exemption class never applies to a publisher).

## Gates vs the publish path

The CVE/scan gate lives in **ci.yml on pull requests against the built
candidate** — never on the publish path (`templates/trivy/`). A CVE
published tomorrow must not retroactively kill a release, and the weekly
remediation rebuild especially must not die because a scanner had a bad
day. Publish runs scan report-only in the prove step so every published
digest has its state on the run log.

## SBOM posture

**Attach to what you publish; scan nothing SBOM-shaped.** Image publishers
set `sbom: true` + `provenance: true` on the push step (SPDX rides the
image as an attestation); release publishers attach SBOMs as release
assets. SBOM-*scanning* jobs and dependency-snapshot submission are
rejected (monumental-archive#148/#166): trivy opens the actual artifact
and does the same CVE matching against the same databases — a snapshot
pipeline is a second notification surface for a gap that is not open.

## Cosign identity conventions

Keyless, OIDC issuer `https://token.actions.githubusercontent.com`. The
certificate identity names the publishing workflow, and consumers pin it
in their base-image policy (`check-base-images.sh` class):

- release-driven: `…/.github/workflows/publish.yml@refs/tags/v.*`
- push-and-cron: `…/.github/workflows/publish.yml@refs/heads/main`

Nothing weaker — a repo-level match would accept any workflow in the repo.

## Egress lore (derived from audit runs — do not rediscover)

Per iiif-server#69's rule: **never prune an allowlist by inspection** —
every reasoned removal there was wrong. Re-derive from an audit run or
leave it alone. Endpoints publish/CI legs have needed, each found the
expensive way once:

| Endpoint | Why |
| --- | --- |
| `production.cloudflare.docker.com` **and** `production.cloudfront.docker.com` | Docker Hub serves blobs from BOTH CDNs; the route varies by runner |
| `mirror.gcr.io` | bootstrap buildkit from here (`driver-opts: image=mirror.gcr.io/moby/buildkit:buildx-stable-1`) — the default Hub pull has dialed past DNS and been rightly blocked; also trivy's DB source |
| `timestamp.sigstore.dev` | cosign v3 bundles are RFC3161-timestamped (alongside fulcio/rekor/tuf) |
| `tmaproduction.blob.core.windows.net` | serves the bundles `gh attestation verify` fetches |
| `mise-versions.jdx.dev` | mise resolves which toolchain version to install |
| `registry.npmjs.org` | mise loads config hierarchically — a subdirectory invocation still installs the root mise.toml's npm tools |
| `apt.postgresql.org:80`, `deb.debian.org:80` | apt fetches over plain http |

`buildkitd`'s network namespace may not be governed by harden-runner at
all — allowlist its endpoints anyway rather than relying on that.

## Skeleton

`publish-image.yml` in this directory is the push-and-cron image publisher
(monumental-archive-db's, genericized). Copy, adapt the pins/paths/smoke
script, keep the shape and the step order. Release-driven repos keep their
phase-2 workflows but hold them against the invariant above.

## Consumers

monumental-archive-db (canonical push-and-cron), iiif-server
(release-driven; wrote the invariant), edtf (release-driven today;
container leg pending edtf#82 adopts the skeleton).
