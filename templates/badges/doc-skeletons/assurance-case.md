# Security assurance case

Why this project's security requirements are met: the threat model, the
trust boundaries, the design argument, and the argument that common
implementation weaknesses are countered. The security requirements
themselves — what a user can and cannot expect — are stated in
SECURITY.md; this document is the evidence that they hold.

This outline IS the Best Practices `assurance_case` criterion's required
shape — fill every section and the criterion is Met by construction.

## Threat model

[Enumerate the threats in order of exposure: what the software accepts
from attackers (input classes), how it ships (supply-chain substitution),
how it builds (CI compromise), and any privilege boundary it sits on.
State the out-of-scope threats explicitly — volume DoS, host security,
confidentiality where nothing confidential is held.]

## Trust boundaries

[Name each boundary: where untrusted data becomes typed data, the process
boundary per deployment shape, and the publish boundary (what CI builds
vs what consumers fetch, and what proves them identical).]

## Secure design principles

[Argue the classics against the actual code: economy of mechanism,
fail-safe defaults (fail-closed parsing/validation), complete mediation
(one implementation, no secondary paths), least privilege (CI token
scopes, egress policy, no long-lived secrets).]

## Common implementation weaknesses, countered

[Per weakness class, name the countermeasure and its evidence: memory
safety (language guarantees, forbid-unsafe), crash/hang on input
(fuzzing, property tests), injection classes, known-vulnerable
dependencies (advisory gates, update automation, documented accepted
ignores), static analysis, supply-chain (pinning, attestation,
verification).]

## Why this is believed sufficient

[Name the residual risks honestly and why both fail toward
detectability rather than silence.]
