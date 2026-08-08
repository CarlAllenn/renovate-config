# OpenSSF Scorecard + Best Practices baseline

What it costs to max OpenSSF Scorecard and the Best Practices badge as a
solo maintainer, and every trap found on the way — copy-template class,
canonized after edtf's hardening sprint (2026-08-08, edtf PRs #92–#102)
took that repo from 6.3 toward its ceiling. Next adopters: iiif-server,
monumental-archive-db, monumental-archive. Per-repo variance is real and
noted where it bites; the traps are ~90% shared.

Two later sections cover the badges that are **not** OpenSSF — coverage
publication and citability. They are here because they are what is left
once the OpenSSF pair hit the solo ceiling: the score does not move
further, so the remaining honest wins are a number nobody had published
and a citation nobody could make.

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
  **Several gold criteria need a URL, not prose.** `hardened_site`,
  `hardening` and `test_invocation` render as unmet with "URL required,
  but no URL found" when the justification is an explanation rather than
  a link — the practice being genuinely in place changes nothing. Cheap
  percent, invisibly lost: put a real link in every justification (a docs
  anchor, a workflow file, a headers report) and keep the prose after it.

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
- **The solo-compatible Branch-Protection set** (3 → 4, measured on edtf
  2026-08-08 against scorecard v5.5.0 — an earlier revision of this file
  guessed ~6): strict status checks with required contexts,
  enforce-admins, required linear history, required conversation
  resolution, force-push and deletion blocked,
  `required_approving_review_count: 0`. Required reviewers is the tier
  you cannot have; everything else you can.
- **Express it as a repository ruleset, not classic branch protection.**
  This is the whole reason the set scores at all: classic branch
  protection is NOT publicly readable, so the OpenSSF scan (which runs
  unprivileged) cannot see it and hedges — `PRs are not required to make
  changes on branch 'main'; or we don't have data to detect it`. Rulesets
  are always public. Migrating edtf's identical settings into a ruleset,
  changing no behaviour whatsoever, took Branch-Protection 3 → 4: the
  scan stopped guessing and credited stale-review dismissal,
  up-to-date-branches and PRs-required, all of which were already on.
  Both mechanisms can coexist; GitHub applies the stricter.
- **Leave `require_last_push_approval` OFF on a solo repo.** It reads
  like a free toggle at `required_approving_review_count: 0` and is not.
  GitHub documents it as a sub-option of the review requirement meaning
  "at least one other authorized reviewer has approved any changes" — so
  enabling it demands an approval that a lone maintainer cannot give
  (self-approval is forbidden), and what it blocks is merging into the
  default branch, i.e. the release path. It buys no Scorecard points
  either, since the tier fails on the approver count regardless.
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
  It also counts **container images**, and that is where a reasoned
  architecture can legitimately lose a point: edtf publishes an OCI image
  whose `FROM postgres:${PG}-trixie` floats deliberately, because the
  image is consumed as a build stage and no consumer inherits its base
  layers — there is no CVE-rebuild obligation to honour. The tag is also
  a build ARG, so one digest cannot serve five majors; pinning would mean
  threading `BASE_IMAGE` through from the digest table. Decide once,
  write the reason in the Dockerfile header, and accept the 9. **Do not
  reverse a documented position to buy a Scorecard point** — the check
  cannot see the argument, which is a limit of the check, not a defect.
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

## Publishing the number — Codecov without the gate

Measuring coverage and publishing it are separate jobs, and the second one
gets forgotten: edtf measured for weeks while writing `lcov.info` into a
build artefact nobody ever opened. Wiring the badge is an afternoon. Every
trap below cost a run to find (edtf #141, PRs #143/#146/#148, 2026-08-09).

- **Codecov's defaults reinstate the gate you just refused.** It posts
  `codecov/project` and `codecov/patch` commit statuses against an
  auto-computed target — a merge-blocking coverage threshold in everything
  but name. Where the repo's position is "measure, don't gate", a
  `codecov.yml` turning both off is not a nicety, it is the entire point.
  **Quote the values**: `off` is a YAML truthy, so a strict yamllint
  rejects `project: off`. Write `project: 'off'`.
- **Mirror the llvm-cov exclusion into Codecov's `ignore:`.** The
  harness-invisible-tests trap above, one layer up — exclude a crate from
  the metric but not from Codecov and the badge reads lower than the job
  summary, for a reason nobody will find quickly.
- **OIDC, not a token.** `use_oidc: true` means no long-lived
  `CODECOV_TOKEN` secret has to exist. Job-level `id-token: write` costs
  nothing on Scorecard Token-Permissions — measured on edtf, whose
  publish.yml already carries it while that check reads 10/10. zizmor at
  `--persona=pedantic` wants the explanatory comment **inline on the
  permission line**; a comment block above it still trips
  `undocumented-permissions`.
- **The egress hosts, measured rather than guessed** (harden-runner
  `egress-policy: block`): `cli.codecov.io` (CLI download), `keybase.io`
  (GPG signature verify), `ingest.codecov.io` (the POST) and
  `storage.googleapis.com` (presigned PUT of the report body). **Not**
  `api.codecov.io`, which is never resolved, and **not** `codecov.io`,
  which appears only as the OIDC audience string. edtf shipped those two
  wrong ones first, and every upload failed silently.
- **`disable_telem: true`.** The CLI otherwise calls Sentry
  (`o26192.ingest.us.sentry.io`); a blocking policy drops it, and the audit
  log fills with a request nobody wanted made.
- **`fail_ci_if_error: false` means a broken upload merges green.** The
  right trade for a non-gating job, and the cost is not theoretical: edtf
  merged the entire feature with the upload failing on every run and found
  out only by reading the step log. Read that log once after the first run
  instead of trusting the check mark.
- **Activation is a second blocker that looks identical to the first.**
  Until a human clicks Configure on codecov.io, uploads answer
  `{"message":"Repository not found"}` while the action still reports
  success. Workflow-correct and repo-registered are independent conditions;
  expect to satisfy them one at a time.
- Fork PRs get no OIDC token — the action detects the fork and skips.
- **Free for public repos**: unlimited uploads and users, no card. Harness
  acquired Codecov from Sentry in June 2026; terms unchanged as of
  2026-08-09. Badge: `img.shields.io/codecov/c/github/<owner>/<repo>`.

## Filling the questionnaire — from edtf's registration (2026-08-08)

The form-filling afternoon compresses to under an hour with the site's
own automation mechanism, and two of its behaviours will eat the last
percent if you don't know them:

- **URL-prefill is the fast path.** `/en/projects/<id>/<level>/edit`
  accepts `?<criterion>_status=Met&<criterion>_justification=...` query
  parameters — batch a dozen criteria per URL (the server rejects URIs
  past ~8 KB), load, hit one Submit per batch. Works on all metal and
  baseline levels. **It only fills criteria still at `?`** — flipping an
  already-answered criterion needs the actual radio button.
- **The analyzer can force-override two criteria on save.**
  `license_location` and `release_notes` are re-detected server-side and
  a human Met answer is reverted to Unmet ("changed from 'Met' to
  'Unmet'" in the flash message) if the detector disagrees. Note it did
  NOT fire on any of three edtf saves on 2026-08-08, so it appears
  conditional on actual disagreement rather than unconditional on every
  save — but re-read both fields after saving regardless.
- **Do NOT add a `LICENSE.md` pointer file.** Two earlier revisions of
  this file recommended it; both were wrong, and the second was wrong in
  the more expensive direction — it conceded the Scorecard point as
  "unpaid" when LICENSE.md was in fact *causing* the loss. Measured on
  edtf 2026-08-08, removing it (edtf#139) moved **Scorecard License
  9 → 10** and **GitHub's own detection `NOASSERTION` → `Apache-2.0`**,
  while `license_location` stayed Met throughout. So it bought neither of
  the two things it was kept for.
  The mechanism: every detector ranks a file named `LICENSE`/`LICENSE.md`
  above `LICENSE-<name>`, so the pointer is the file that gets
  classified — and a pointer contains no licence text. Removing it lets
  both land on `LICENSE-APACHE`. This is simply what the dual-licensed
  Rust ecosystem does; serde-rs/serde and rust-lang/regex carry exactly
  `LICENSE-APACHE` + `LICENSE-MIT` with no `LICENSE.md`, and both report
  `Apache-2.0` to GitHub and score License 10/10.
  **Canonical layout for a dual-licensed repo:** `LICENSE-APACHE` and
  `LICENSE-MIT` at the root, a `## License` section in the README
  carrying the SPDX expression and the contribution clause, `license =
  "MIT OR Apache-2.0"` in the workspace manifest, and SPDX headers in
  sources. Nothing named `LICENSE.md`.
  **If you are removing an existing LICENSE.md, grep the badge record
  for it first** — justification prose that cites the file becomes a live
  404. On edtf it appeared twice: `license_location` and the Baseline
  criterion `OSPS-LE-03.02`, which duplicates the same paragraph.
- **Three gold criteria sit Met-but-incomplete until the justification
  contains a URL**: `test_invocation`, `hardening` and `hardened_site`.
  The form reports "URL required but missing" and silently withholds the
  credit — on edtf that was **13 % of gold** (70 → 83) sitting unclaimed
  behind prose that was already true. Cite the repo docs that prove each
  one; a scan URL is fine for `hardened_site`, but verify the headers
  yourself (`curl -I`) so the justification names what is actually served
  rather than asserting it.
- **Silver is reachable with bus factor 1**: `bus_factor` is a SHOULD at
  silver, so Unmet-with-justification (pointing at GOVERNANCE.md access
  continuity) still awards the badge. It becomes a MUST at gold.
- **The Baseline series is nearly free once metal is done.** Same
  evidence, different criterion IDs (`osps_ac_01_01`-style params).
  Baseline-1 and -2 complete from existing answers; Baseline-3 caps at
  ~95 % solo (OSPS-QA-07.01 is non-author review) with one genuinely
  fixable row: OSPS-VM-04.02 wants deny.toml non-exploitability
  rationale as a formal VEX document — a small OpenVEX JSON with a
  `not_affected` statement, referenced from SECURITY.md, mirroring the
  guarded ignore (edtf: `.vex/edtf.openvex.json`).

## REUSE compliance — one manifest if SPDX headers already exist

The SPDX-headers pass (gold `copyright_per_file`/`license_per_file`)
leaves a repo one file short of [REUSE](https://reuse.software)
compliance: a `REUSE.toml` with a blanket project-licence annotation at
`precedence = "closest"` (in-file headers win) covers everything that
cannot carry a header, plus canonical texts in `LICENSES/` (exclude that
directory from editorconfig-checker — verbatim licence text is not ours
to reindent). CI: `fsfe/reuse-action`, SHA-pinned, one docker pull
(harden-runner needs the Docker Hub endpoints). The api.reuse.software
badge requires registering the repo — owner's email, confirmation link.
Working example: edtf PR #114.

## Citability — CITATION.cff and a Zenodo DOI

Worth it only where people actually cite the software: libraries, archives,
museums, digital humanities, research tooling. For those audiences it beats
another green checkmark, because "which version did you run?" is a question
their papers have to answer. Skip it for a web service. Canonized from
edtf #142 / PRs #145 and #150 (2026-08-09).

- **Zenodo is free** — CERN-operated, DOIs minted through DataCite, no tier
  to buy. Sign in with GitHub, then toggle the repo in Settings → GitHub.
- **It never backfills.** Only releases created *after* the toggle are
  archived, so no DOI exists until the next release, and the "Enabled
  Repositories" panel stays empty until a deposit does. That empty panel is
  the expected state and looks exactly like a failure — say so in the
  runbook or it gets debugged.
- **Concept DOI, never a version DOI.** Zenodo mints a version DOI per
  release under a stable concept DOI that resolves to the newest. The badge
  and `CITATION.cff` want the concept one; a version DOI is wrong within a
  single release.
- **Skip `.zenodo.json`.** It overrides `CITATION.cff` and buys finer
  control at the price of two metadata files describing one project.
  Record the omission as a decision, or someone adds it back later.
- **`version:` and `date-released:` rot silently.** release-plz cannot see
  the file. Put the sync in whatever script already does the things the
  release tool cannot — and name the asymmetry while you are there: the
  other entries in such a script fail the lint gate when forgotten, this
  one just goes on claiming an old version, which is the worse failure.
- **Two tooling blind spots, because `.cff` is YAML wearing a non-YAML
  extension.** yamllint discovers `*.yml`/`*.yaml` only, so name the file
  explicitly in the lint command. The canonical editorconfig config-formats
  glob does not match it either, so it inherits the `[*]` indent — add a
  repo-local `[*.cff]` block rather than editing the canon glob.
- **Gate the schema, and not with the obvious tool.** yamllint proves the
  file is YAML, not that it is CFF, and the edits that break it are
  misspelled keys, a wrong `authors` shape, a malformed `identifiers:`
  block (which is exactly what adding the DOI later involves). cffconvert
  is the format's canonical validator and last shipped **September 2021** —
  do not pin five-year-old tooling into a repo that sets
  `unmaintained = "all"`. Use
  `check-jsonschema --builtin-schema vendor.citation-file-format`: 0.37.4
  (June 2026), `pipx:` backend, and the schema travels inside the tool, so
  the gate stays offline and no vendored schema copy drifts here.
  **Negative-test it** — misspelling `license:` as `licence:` must exit 1.
  That typo is likely rather than hypothetical in any repo running an en-GB
  prose register over an en-US key.
- The failure being gated is silent. An invalid file breaks no build, no
  release and no user; GitHub simply stops rendering "Cite this
  repository", and nobody notices until they go looking for a citation.

## Consumers

edtf (canonical, first adopter — scheduled.yml scorecard job, publish.yml
bundle-asset steps, the doc pack, `.github/codecov.yml` and `CITATION.cff`
all live there as working examples); iiif-server and monumental-archive-db
pending.

Coverage publication and citability are independent of the OpenSSF work and
of each other — adopt either alone. Citability in particular is audience-led
rather than baseline: iiif-server and monumental-archive-db plausibly want
it, a private service does not.
