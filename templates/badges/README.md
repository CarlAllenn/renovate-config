# OpenSSF Scorecard + Best Practices baseline

What it costs to max OpenSSF Scorecard and the Best Practices badge as a
solo maintainer, and every trap found on the way — copy-template class,
canonized after edtf's hardening sprint (2026-08-08, edtf PRs #92–#102)
took that repo from 6.3 toward its ceiling. Next adopters: iiif-server,
monumental-archive-db, monumental-archive. Per-repo variance is real and
noted where it bites; the traps are ~90% shared.

## The solo ceiling, stated up front

Three Scorecard checks and the Best Practices **gold** badge are people
questions, not engineering ones. Do not burn time on them:

- **Code-Review: 0 forever.** You merge your own PRs.
- **Contributors: 0 forever.** Needs contributors from multiple orgs.
- **Maintained: 0 until 90 days** of sustained activity on a 90-day-old
  public repo. Nothing to do but exist.
- **Gold badge: unreachable** (bus factor ≥ 2, two unassociated
  significant contributors, 50 % non-author review). Fill the gold
  questionnaire anyway — the site shows a public per-level percentage and
  every Met answer counts toward it.

Everything else is reachable. A solo repo that does everything below
lands roughly **8.0–8.5** on Scorecard and **silver** on the badge.

## Scorecard traps — each found the expensive way once

- **Signed-Releases reads release *assets*, nothing else.** GitHub
  attestations living in the attestation store score ZERO — the check
  greps asset filenames for signature shapes. Publish the Sigstore
  bundles the attest steps already produce as `*.intoto.jsonl` release
  assets (capture `bundle-path` from each `actions/attest` step, attach
  while the release is a draft). Scores only heal as releases ship: the
  check reads the latest five. Push-and-cron image repos with no releases
  (monumental-archive-db) are scored N/A here, not zero — do not invent
  releases to feed it.
- **`downloadThenRun` means any pipe into an executor.** Two patterns to
  purge: `curl | sh` installers (pin the installer binary by version,
  verify its published SHA256, then execute — rustup-init has per-target
  `rustup-init.sha256` files), and `curl | python3 -c` over *data* —
  scanner can't tell JSON from code, so fetch to a file and parse the
  file. Zero-cost appeasement, and the first one is a real fix.
- **The solo-compatible Branch-Protection set** (~3 → ~6): strict status
  checks with required contexts, enforce-admins, required linear history,
  required conversation resolution, force-push and deletion blocked,
  `required_approving_review_count: 0`. Required reviewers is the tier
  you cannot have; everything else you can.
- **SAST is the cheapest 10 on the board.** One CodeQL workflow —
  `codeql.yml` in this directory. Rust uses `build-mode: none` (no
  toolchain install, no conflict with feature-exclusive crates); the
  `actions` language scans the workflows themselves. First PR after
  merging shows a one-off "configuration not found" warning until the
  slower language's analysis uploads a main baseline; it self-resolves.
- **Pinned-Dependencies wants hashes on GitHub Actions AND scripts.**
  Action SHA-pinning is already canon here; the scripts are where the
  stragglers hide (see downloadThenRun above). The check's shell parser
  chokes on some legal bash and reports "possibly incomplete results" —
  those infos are score-neutral; do not contort working shell for them.
- **Vulnerabilities counts unmaintained-crate notices.** An advisory
  reachable only through a framework dependency (edtf: RUSTSEC-2021-0127,
  serde_cbor via pgrx) docks a point until upstream moves. Document the
  ignore in deny.toml with the upstream issue, guard it with
  `unused-ignored-advisory = "deny"`, and accept the 9.
- **CII-Best-Practices is per-repo registration**, not a profile:
  bestpractices.dev → sign in with GitHub → Add Project → repo URL.
  Passing ≈ 5 Scorecard points, silver ≈ 7. The score lands on the next
  weekly scheduled Scorecard run, not immediately.

## Best Practices — the doc pack is the work

Passing is a form-filling afternoon against what these repos already do.
Silver's delta over passing is almost entirely named documents; the
skeletons in `doc-skeletons/` are edtf's, genericized — replace
OWNER/REPO and the bracketed prompts, delete what a repo honestly cannot
claim:

- `CONTRIBUTING.md` — contribution requirements, DCO-by-signoff, the
  coding standard BY REFERENCE to the enforced lint canon (never restate
  it), test policy ("tests are part of the change"), the code-review
  standard stated honestly for a solo project, small-tasks pointer.
- `GOVERNANCE.md` — the honest model ("benevolent dictator" beats a
  fictional committee) and **access continuity**, the criterion that
  looks unmeetable solo and isn't: no long-lived tokens (OIDC trusted
  publishing), a runbook a stranger can execute, estate arrangements +
  GitHub's deceased-user policy, and fork-and-continue licensing.
- Code of conduct: Contributor Covenant 2.1 verbatim with the contact
  filled — fetch it, don't paraphrase it. Wrap its long lines if the
  repo's markdownlint enforces line length (it ships unwrapped).
- `roadmap.md` — will-do / will-not-do for a year. Word exclusions as
  **use-case-gated, contributions welcome**, not "never": "no speculative
  X; stays out until a specific use case appears" keeps the door open
  without inviting scope creep.
- `assurance-case.md` — the silver criterion with an exact required
  shape: threat model, trust boundaries, secure-design argument,
  countered weakness classes. Write it to that outline and the criterion
  is Met by construction.
- A **security review** (gold, but cheap): the criterion needs *a*
  documented review in five years, not an external audit. A hardening
  sprint written up as scope → method → findings → dispositions IS one.
  Record it while it's fresh.
- SPDX headers in every source file (gold `copyright_per_file` +
  `license_per_file`): two comment lines, scriptable in one pass, after
  shebangs in shell.

Answer the questionnaire from the source, not memory: the criteria live
in `criteria/criteria.yml` and the question text in
`config/locales/en.yml` of
<https://github.com/coreinfrastructure/best-practices-badge>.

## Coverage — measure honestly or the numbers lie both ways

- Silver wants ≥ 80 % statements, gold ≥ 90 % statements and ≥ 80 %
  branches. **Branch coverage needs nightly**: `cargo llvm-cov --branch`
  passes `-Z coverage-options=branch`. With mise's stable rust first in
  PATH the rustup proxy is shadowed — run with the pinned nightly's bin
  dir prepended AND `RUSTC` set explicitly, or the inner rustc silently
  resolves to stable and errors.
- **Harness-invisible tests deflate the total.** A crate whose tests run
  inside another harness (pgrx spinning real Postgres, container smoke
  tests) shows near-0 % to llvm-cov while being thoroughly tested.
  Exclude it from the metric with `--ignore-filename-regex` and a comment
  naming the harness that does cover it — 2 % left on the report reads as
  a gap when it's an artifact.
- The last uncovered stretch is usually metadata accessors and defensive
  arms. The honest close-out: a test that holds metadata against the
  document it cites (docs-drift guard, not theatre), then per-line
  triage — prove reachable, delete as dead, or document as
  defensive-unreachable. Never write tests that call code purely to have
  called it.

## Consumers

edtf (canonical, first adopter — scheduled.yml scorecard job, publish.yml
bundle-asset steps, and the doc pack all live there as working examples);
iiif-server and monumental-archive-db pending.
