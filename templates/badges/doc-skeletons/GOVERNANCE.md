# Governance

## Model

The project uses the simplest governance model that is honest about how it
operates: a single maintainer ([@OWNER](https://github.com/OWNER)) makes
all final decisions — a "benevolent dictator" model. Proposals,
disagreements and design discussion happen in the open, on the issue
tracker and in pull requests; the maintainer decides, and records
load-bearing decisions in [REPO-SPECIFIC: the decision register(s)] so
they bind future work rather than being re-litigated.

## Roles and responsibilities

- **Maintainer** (currently the only role held): triages issues, reviews
  and merges pull requests, cuts releases through the release pipeline,
  responds to security reports per SECURITY.md, and owns the decision
  registers.
- **Contributors**: anyone submitting issues or pull requests under the
  requirements in CONTRIBUTING.md. No CLA — the DCO sign-off is the
  legal mechanism.

Should the project gain regular contributors, committer status and this
document evolve with it; until then, documenting a committee that does
not exist would be less honest than documenting the dictatorship that
does.

## Access continuity

The practical single-maintainer risk is mitigated as follows:

- Everything needed to build, test and release lives in the repository:
  pinned toolchain, release pipeline, and a runbook written so that a
  stranger can reproduce the setup end to end.
- Publishing uses OIDC trusted publishing — there are no long-lived
  registry tokens to lose. Whoever controls the GitHub repository can
  release; any remaining secret is documented in the runbook with how to
  recreate it.
- The maintainer's estate arrangements cover credential succession for
  the GitHub account, and GitHub's
  [deceased user policy](https://docs.github.com/site-policy/other-site-policies/github-deceased-user-policy)
  provides a fallback path for transferring the repository.
- The licence(s) guarantee that, in the worst case, the project can be
  forked and continued by anyone without any legal transfer at all — and
  the runbook is written to make exactly that practical.
