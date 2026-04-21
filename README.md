# Prism-Devil — Aggressive Adversarial Probe

**Language**: **English** · [한국어](README.ko.md)

> `prism` looks from 5 angles. `prism-devil` puts on the black hat and tries to break it.

---

## What this is

A Claude Code skill that runs a **single agent** in deliberately hostile, paranoid mode to find vulnerabilities, attack surface, and failure modes. Unlike `prism` (which is balanced), `prism-devil` is **deliberately one-sided** — it does not praise, it does not balance, it tries to break the code on paper.

Critically, it **auto-loads your project's security checklist** if one exists (`docs/security-checklist.md` and similar paths). Each finding is cross-referenced to checklist items, so the report is grounded in *your* threat model, not generic OWASP boilerplate.

## When to use it

- Code touching auth, payments, PII, credentials, or anything money-adjacent
- File parsers handling untrusted input (xlsx, pdf, image, EML)
- Pre-production gates — pair with `/prism --verify` afterward
- Whenever you suspect you're too close to your own code to see the holes
- Periodic security audit on infrastructure code

## When not to use it

- General code review — `prism` (broad, balanced) is the right tool
- Markdown / spec review — `triad` is the right tool
- Pure performance or design questions — out of scope here

## What makes it "aggressive"

- **No sugarcoating** — the agent is told explicitly to skip praise and balance
- **Scenario required** — every finding must include a step-by-step attack/failure trigger, not just "this might be risky"
- **Strict severity rubric** — CRITICAL is reserved for actual remote exploit / data loss / auth bypass; the agent can't dilute severity to seem productive
- **Attack chain detection** — when 2+ findings compose into a worse outcome (e.g. ID leak + missing auth check = account takeover), it names the chain explicitly
- **Checklist cross-reference** — every finding maps to a project checklist item or is tagged `[not-in-checklist]` (so the checklist itself can grow)

## The 10 attack surfaces probed (in priority order)

1. **Trust boundaries** — input crossing untrusted → trusted
2. **Authentication & authorization** — session, token, privilege, race
3. **Concurrency weaponization** — TOCTOU, double-submit, lost update
4. **Data integrity attacks** — FK cascade abuse, constraint bypass, audit tamper
5. **Failure mode exploitation** — crash-induced state, retry amplification, replay
6. **Secrets & disclosure** — log leakage, timing side channels, stack traces
7. **Dependency & supply chain** — pinning gaps, untrusted file parsing, deserialization
8. **Self-evaluation bias** — self-grading metrics, Goodhart, false success signals
9. **False confidence** — happy-path-only tests, silent except, log-and-continue
10. **Platform / environment** — Windows vs POSIX, timezone, locale parsers

## Output format (per finding)

```
[ID] — severity: CRITICAL | HIGH | MEDIUM | LOW — class: <attack_class>
  Scenario: <concrete step-by-step attack or failure trigger>
  Why it lands: <which line / missing check / flawed assumption>
  Impact: <what the attacker gains or what breaks>
  Fix: <specific, minimum-viable mitigation — not "add validation">
  Confidence: high | med | low
  Checklist: <item number/tag, or [not-in-checklist], or [N/A: skipped]>
```

Plus an **Attack Chain Section** when findings compose, and a **Checklist Coverage Section** showing which `[ACTIVE]` items were exercised vs which `[N/A]` items might have been re-activated by recent changes.

## Top-line summary

```
PRISM-DEVIL — src/services/auth.py — 7 findings (2C / 3H / 1M / 1L) — chains: 1 — checklist: 12/77 active items mapped, 2 new gaps
```

## Project checklist support

If your project has a security checklist with tagged items like:

```markdown
- [ACTIVE] #14 — All session cookies set Secure + HttpOnly + SameSite
- [N/A: no payments] #41 — Stripe webhook signature verification
- [FUTURE: when AWS migration lands] #59 — IAM least-privilege per service
```

prism-devil will:
- Map findings to specific item numbers
- Flag if newly-added code re-activates an `[N/A]` item (e.g. you added Stripe → all `[N/A: no payments]` items are now suspect)
- List items it tried to exercise but found clean (`verified-clean`)
- Tag findings outside the checklist as `[not-in-checklist]` — these are signals to grow the checklist itself

If no checklist exists, it falls back to generic attack surfaces. Never fabricated.

## Pairing with `/prism`

`prism-devil` is allowed to cry wolf — it has no verifier. The recommended high-confidence flow for security-sensitive code:

```
/prism-devil src/security/auth.py     # paranoid pass — generates the worst-case list
/prism src/security/auth.py           # 5-agent verified pass — filters false positives
```

The intersection is high-signal. `prism-devil`-only findings are still worth reading but treat them as suspicions, not verdicts.

## Companion tools

- **[prism](https://github.com/minwoo-data/prism)** — broad 5-agent review with verifier (parent tool)
- **[mangchi](https://github.com/minwoo-data/mangchi)** — iterative cross-model file refinement with Codex CLI
- **[triad](https://github.com/minwoo-data/triad)** — 3-perspective deliberation for markdown

## Install

Drop the skill into your Claude Code skills directory:

```
~/.claude/skills/prism-devil/SKILL.md
```

Or wire as a slash command via `commands/prism-devil.md` in `.claude/commands/` (per project) or `~/.claude/commands/` (global).

## Created by

Minwoo Park
