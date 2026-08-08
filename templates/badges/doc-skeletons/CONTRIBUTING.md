# Contributing

Contributions are welcome — issues, discussion and pull requests alike.
This document is the contract for what an acceptable contribution looks
like; the enforcement layer described below applies it mechanically, so
nothing here is aspirational.

## Development quick start

Tooling is pinned by [mise](https://mise.jdx.dev) and tasks run via
[Task](https://taskfile.dev):

```bash
git clone https://github.com/OWNER/REPO
cd REPO
mise install   # installs the pinned toolchain and the git hooks
task ci        # the full gate: every linter + the full test suite
```

`mise install` also installs the lefthook git hooks; from then on every
commit and push runs the same gate CI runs. There are no bypass flags —
fixing tool output is never optional.

## Requirements for an acceptable contribution

- **One pull request per issue.** Branch from `origin/main`.
- **Conventional commits**, enforced by the commit-msg hook.
- **Signed commits.** A signed-commit ruleset is active on the repository.
- **Developer Certificate of Origin.** By adding `Signed-off-by` to your
  commits (`git commit -s`) you assert the
  [DCO](https://developercertificate.org/): that you are legally entitled
  to contribute the change under the project's licence(s).
- **Tests are part of the change, not a follow-up.** New functionality
  comes with tests that exercise it; bug fixes come with a regression
  test that fails without the fix.
- **The gate must be green.** `task ci` is identical to GitHub CI.
- [REPO-SPECIFIC: any decision registers, spelling registers, or domain
  rules a contribution must respect.]

## Coding standard

The coding standard is the enforced lint configuration itself — cite the
workspace lint config, formatter config, and per-format linter files by
path rather than restating them. It is enforced twice: locally by the
hooks, remotely by required status checks on `main`, so a change that
violates it cannot merge.

## Code review

Every change — including the maintainer's own — lands through a pull
request against `main` with the full gate green; direct pushes are
blocked by branch protection. Review checks, in order of importance:

1. [REPO-SPECIFIC: the domain-correctness check that matters most here.]
2. **Tests carry the claim**: the diff's tests must fail without the
   change; fixes need a regression test.
3. **The gate**: reviewers do not re-litigate what the gate enforces
   mechanically.
4. **Release-pipeline changes** get adversarial review: a pipeline diff
   is reviewed by asking what it can no longer catch.

External contributions are reviewed by the maintainer against the same
list. The project is honest that maintainer-authored changes are
self-reviewed against it (see GOVERNANCE.md); tool assistance is used
liberally, but the accountable reviewer is the maintainer.

## Small tasks for new contributors

Issues labelled `good first issue` are scoped to be tractable without
deep context. [REPO-SPECIFIC: name the perennially good entry points.]

## Reporting problems

Bugs and feature requests go to the issue tracker. Security problems go
through private vulnerability reporting — see SECURITY.md, which includes
the response process and reporter credit policy.
