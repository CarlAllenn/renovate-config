# rustfmt baseline

Canonical `rustfmt.toml` for the Rust repos (edtf, iiif-server) —
copy-template class, drift-audited like every template here. Established in
issue #8 (2026-07-27); decisions agreed with Carl same day.

## The nightly decision

The max-enforcement half of rustfmt's option set (`imports_granularity`,
`group_imports`, `wrap_comments`, `format_code_in_doc_comments`,
`error_on_line_overflow`, …) is nightly-only. Stable rustfmt offers little
beyond defaults. Decision: **run the format gate on a pinned nightly
toolchain** — the second sanctioned nightly exception after cargo-fuzz, and
under the same rules:

- The pin is a dated toolchain (`nightly-YYYY-MM-DD`), installed via
  `rustup toolchain install <pin> --profile minimal --component rustfmt`,
  recorded once per repo (Taskfile env var `RUSTFMT_TOOLCHAIN`) so hooks, the
  Taskfile, and CI all read the same value.
- Only the `fmt` gate uses it. Everything else builds on the stable pin in
  mise.toml.
- Renovate moves the pin (regex manager on the Taskfile variable).

## Recorded non-adoptions

- `format_strings` — off. It wraps long string literals with mid-literal
  backslash continuations, which is strictly worse than hand-wrapping.
  `error_on_unformatted = true` still fails overlong strings, so the gate
  holds; a human chooses the split points.
- `hard_tabs`, `max_width`, `tab_spaces` — defaults (false/100/4) are the
  style-guide values; restating them would imply they were choices.

## Consumers

Copy `rustfmt.toml` to the repo root verbatim. The gate invocation is
`cargo +$RUSTFMT_TOOLCHAIN fmt --all --check` (plus per-extra-workspace
`--manifest-path` runs where a repo has nested workspaces, e.g.
edtf-postgres and both repos' fuzz/ workspaces).
