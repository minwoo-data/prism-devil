# Changelog

All notable changes to the Prism-Devil plugin are documented here.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

## [0.1.0] — 2026-04-20

Initial public release. Evolved from an earlier `review-devil` skill that was a
generic Devil's Advocate pass — this version is sharpened for security work and
auto-loads project security checklists.

### Added
- Single-agent aggressive red-team probe — deliberately one-sided, no praise/balance
- 10 attack surfaces probed in priority order: trust boundaries, authn/authz,
  concurrency weaponization, data integrity attacks, failure mode exploitation,
  secrets & disclosure, supply chain, self-evaluation bias, false confidence,
  platform/environment
- Strict severity rubric — CRITICAL reserved for actual remote exploit / data
  loss / auth bypass; agent forbidden from diluting severity
- Mandatory finding format: scenario + why-it-lands + impact + fix + confidence
  + checklist mapping. One-line findings rejected.
- **Attack chain detection** — when 2+ findings compose into a worse outcome,
  the chain is named explicitly
- **Project security checklist auto-loading** — checks `docs/security-checklist.md`,
  `docs/security/checklist.md`, `SECURITY.md`, `docs/SECURITY.md` in order
- **Checklist Coverage Section** — every report shows which `[ACTIVE]` items
  were probed (verified-clean or hit), and flags `[N/A]` items that recent
  changes might have re-activated
- Findings outside the checklist tagged `[not-in-checklist]` — direct signal
  to extend the checklist itself
- Top-line summary includes severity counts, chain count, and checklist coverage
  metrics

### Known Gaps
- No verifier — this tool is allowed to cry wolf by design. Pair with
  `/prism --verify` for filtering.
- Checklist parsing is convention-based (`[ACTIVE]`, `[N/A]`, `[FUTURE]` tags).
  Custom formats won't be recognized.
- Single agent only — no cross-agent agreement signal. A `prism-devil` finding
  is one reviewer's opinion, not consensus.

### Companion tools
- `prism` — broad 5-agent verified review (parent tool)
- `mangchi` — iterative cross-model file refinement (Codex CLI)
- `triad` — 3-perspective deliberation for markdown
