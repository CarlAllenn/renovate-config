# Universal linter baseline

Canonical configs for the nine linters pinned in every active repo's
mise.toml: yamllint, taplo, shellcheck + shfmt, codespell, markdownlint-cli2,
editorconfig(-checker), actionlint, zizmor. Copy-template class: repos copy
these files verbatim; propagation is a deliberate drift-audit, never
automatic. Established in issue #4 (2026-07-26). Non-universal linters
(biome, ruff, sqlfluff, trivy, hadolint, clippy/rustfmt/cargo-deny) are
issue #8 scope and have no templates here.

The bar for every file: **maximum enforcement, by the docs, no exemptions,
impossible to skip.** Every rule is on unless a reason is recorded below;
every repo-local exclude carries an inline reason.

## File map

| Template | Lands at | Variance allowed |
| --- | --- | --- |
| `.yamllint` | repo root | `ignore:` list only |
| `taplo.toml` | repo root | none |
| `.shellcheckrc` | repo root | none |
| `.codespellrc` | repo root | `skip` tail, `ignore-words-list` |
| `.markdownlint-cli2.jsonc` | repo root | none (gitignore covers repo dirs) |
| `.editorconfig` | repo root | `[*]` indent_size; extra blocks at end |
| `.editorconfig-checker.json` | repo root | `Exclude` entries (anchored regex) |
| `actionlint.yaml` | `.github/actionlint.yaml` | listed labels/variables |

zizmor needs no config file: since v1.20 its defaults already require
hash-pinning for every `uses:` and nothing is ignored. Its strictness lives
in the invocation (`lefthook/universal.yml`): `--persona=pedantic`, no
`--min-severity` filter, `.github/` collection scope (zizmor has no
exclude mechanism, so whole-tree `.` would walk `.git/` and `.claude/`).

## Decisions (rules deliberately off, and why)

- **yamllint `document-end`**: a `...` marker catches no error class and
  fights every third-party YAML convention.
- **yamllint `key-ordering` / taplo `reorder_keys` + `reorder_arrays`**:
  alphabetical ordering destroys semantic ordering (name/on/jobs in
  workflows, dependency grouping in TOML) and catches nothing.
- **yamllint `line-length.max` = 120, not 80**: the repo-wide column canon
  (matches markdownlint and taplo `column_width`); Renovate's digest-pin
  comments regularly exceed 80.
- **markdownlint MD013 `tables`/`code_blocks` exempt**: wide spec tables and
  code can't be reflowed without destroying them; `strict`/`stern` stay off
  so genuinely unbreakable lines (URLs) don't hard-fail with no fix.
- **codespell builtins `code` and `names` off**: they exist to catch
  code-words (`uint`) and proper names used *outside* code — in a codebase
  they flag legitimate identifiers by design. `en_to_en-OX` off: we write
  en-US, enforced by `en-GB_to_en-US`.
- **editorconfig `max_line_length` unset**: line length is owned per-language
  by the responsible linter (yamllint/markdownlint at 120, rustfmt at 100);
  a global ec check would hard-fail unfixable long string literals.
- **taplo schema validation not enabled in hooks**: remote schema fetches
  make commits network-dependent and nondeterministic; schema checking
  belongs to the editor/LSP layer.
- **zizmor `--offline` in the hook**: the online audits
  (known-vulnerable-actions etc.) are time-varying advisory lookups — the
  never-fingerprint class from issue #3 — so a commit that passed yesterday
  could fail today with no diff. They belong to scheduled scanning (issue
  #7), not commit gates. `--persona=auditor` off: documented as including
  expected false positives, which breaks "the hook must be satisfiable".
- **actionlint pyflakes not installed**: no workflow uses `shell: python`;
  shellcheck integration uses the mise-pinned binary from PATH.
- **shellcheck**: `enable=all` covers all optional checks;
  `external-sources=true` so sourcing is followed, not skipped.
- **editorconfig-checker `Exclude`**: defaults already cover `.git/`, lock
  files, and `node_modules/`; repo entries must be anchored regexes with a
  reason (machine-written files only).

## The .git/ rule

taplo, markdownlint-cli2, and yamllint all traverse `.git/` (the issue #3
session's "only two scanners" finding missed yamllint because the remote
cache didn't exist in the repo it tested); codespell joins them once
`check-hidden = true` is on. All four must exclude it — the lefthook
remote cache (`.git/info/lefthook-remotes/`) contains files formatted under
another repo's settings. `.claude/` (machine-local session state, possibly
whole worktrees of other checkouts) is excluded by the same logic in every
tree-walking linter, ec included.
