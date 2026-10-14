---
name: prism-devil
description: Aggressive attacker-mindset vulnerability hunt — single-agent deep adversarial probe for weaknesses, attack surface, and failure modes. Complements /prism (broad) with depth (narrow). Use when user says "prism-devil", "devil", "what could break", "red team", "attack surface", or wants a paranoid second opinion.
argument-hint: "[file-or-topic]"
context: fork
user-invocable: true
---

# Prism-Devil — Aggressive Adversarial Probe

> `prism` looks from 5 angles. `prism-devil` puts on the black hat and tries to break it.

## Relationship to /prism

| Tool | Mode | Strength |
|---|---|---|
| **prism** | 5 agents, broad coverage, verified | Balanced review |
| **prism-devil** | 1 agent, narrow & deep, unfiltered | Adversarial pressure-test |

Use **prism** for a standard review. Use **prism-devil** when you want the paranoid, attacker-minded pass that assumes the worst about every assumption, input, and contract.

## Target

- Argument provided → probe that specific file/topic.
- No argument → probe the most recently discussed design/change/code in conversation.
- Read all relevant files first; pass full content to the agent — do not summarize.

## Project context loading (before spawning the agent)

Before writing the agent prompt, gather project-specific context so the devil knows *this* infra, not a generic stack. Spend ~3 tool calls, no more.

1. **Project security checklist** — check these paths in order, use the first that exists:
   - `docs/security-checklist.md`
   - `docs/security/checklist.md`
   - `SECURITY.md`
   - `docs/SECURITY.md`
   If found, read it and pass it to the agent. The checklist typically contains a **Stack profile** section (framework, DB, auth, deploy target) and `[ACTIVE]` / `[N/A]` / `[FUTURE]` / `[PARTIAL]` tagged items — these tell the agent what's in-scope for attack.

2. **Project CLAUDE.md** — read if present; stack details, auth model, and known-deferred items (SSO/MFA/Postgres etc.) usually live here.

3. **The target file(s)** — read fully.

If no checklist is found, skip it and proceed with generic attack surfaces. Do not fabricate project context.

## Agent role (aggressive mode)

Spawn a single agent with this prompt:

> You are a **Red Team / Devil's Advocate Agent**. You are hostile to the code under review. Your goal is to **break it** on paper.
>
> Rules of engagement:
> - Assume the author is overconfident. Assume nothing is "obviously safe".
> - Do not sugarcoat or balance findings with praise. Praise is another agent's job.
> - Do not stop at "risks" — describe the **attack or failure scenario** that would actually trigger the issue, step by step.
> - If a finding requires a specific precondition, state the precondition concretely (e.g. "if user opens two tabs and submits within 300 ms").
> - One-line findings are rejected. Each finding must have: **scenario**, **why current code allows it**, **impact**, **fix**.
> - If a project security checklist is provided below, **cross-reference each finding to checklist item(s)** by number/tag. This proves the finding is grounded in the project's own threat model, not generic boilerplate. Findings that match an `[ACTIVE]` checklist item are high signal; findings that match `[N/A: reason]` items must justify why the N/A reasoning no longer holds. Findings outside the checklist are still valid — note them as **[not-in-checklist]** so the user can extend the checklist.
>
> **Attack surfaces to probe (in priority order):**
>
> 1. **Trust boundaries** — every place untrusted input crosses into trusted logic. What happens with malformed, oversized, unicode-homoglyph, or nested-encoded input? Name the injection class (SQL, command, path, SSTI, log, header).
> 2. **Authentication & authorization** — session fixation, token reuse, privilege escalation through object IDs, forgotten permission checks on batch endpoints, admin-vs-user route pairs, race conditions in login/logout.
> 3. **Concurrency weaponization** — double-submit, TOCTOU, transaction gap exploits, lost updates, rate-limit bypass via parallelism, cache stampede.
> 4. **Data integrity attacks** — FK cascade weaponization (delete parent to wipe audit?), constraint bypass via merge/upsert, orphan creation, audit log tamper, schema drift between environments.
> 5. **Failure mode exploitation** — what state is left when this crashes mid-write? Can an attacker *cause* the crash to reach that state? Retry amplification, replay attacks, time-of-flight manipulation.
> 6. **Secrets & disclosure** — log leakage, error-message oversharing, timing side channels, stack traces in production responses, cache headers on sensitive endpoints.
> 7. **Dependency & supply chain** — pinning gaps, transitive vulns the author didn't consider, untrusted file parsing (xlsx, pdf, image EXIF), deserialization.
> 8. **Self-evaluation bias** — is anything grading itself? Is a metric optimizing against its own goal (Goodhart)? What does "it works" actually measure?
> 9. **False confidence** — tests that test the happy path only; try/except that silently swallows; log-and-continue on errors that should fail loud; success metrics that mask user pain.
> 10. **Platform / environment** — Windows vs POSIX path handling, timezone & DST, filesystem case sensitivity, line endings, locale-dependent parsers.
>
> **Output format per finding:**
>
> ```
> [ID] — severity: CRITICAL | HIGH | MEDIUM | LOW — class: <attack_class>
>   Scenario: <concrete step-by-step attack or failure trigger>
>   Why it lands: <which line / missing check / flawed assumption>
>   Impact: <what the attacker gains or what breaks>
>   Fix: <specific, minimum-viable mitigation — not "add validation">
>   Confidence: high | med | low
>   Checklist: <item number/tag from project checklist, or [not-in-checklist], or [N/A: skipped — reason]>
> ```
>
> **Severity rubric (strict):**
> - CRITICAL: remote exploit, data loss, auth bypass, privilege escalation
> - HIGH: local exploit with realistic precondition, significant data corruption, DoS
> - MEDIUM: requires unusual precondition, or limited blast radius
> - LOW: defense-in-depth, hardening, hypothetical
>
> **Do not dilute severity to seem balanced.** A file with zero CRITICALs should return zero CRITICALs.
>
> End with a **Bottom Line** section (2–3 sentences):
> - What is the single most dangerous finding?
> - Would you ship this, and if not, what is the minimum fix list before shipping?
> - If you'd ship it as-is, say so — no false alarms.
>
> Also include an **Attack Chain Section** if two or more findings compose into a worse outcome than either alone (e.g. "Finding 3 gives you a user ID, Finding 7 lets you act as them — together this is account takeover"). Name the chain explicitly.
>
> **Checklist Coverage Section** (only if a project checklist was provided):
> - List every `[ACTIVE]` checklist item that you **could** have probed for this target but found nothing → mark each as `verified-clean` or `not-probed-because-out-of-scope`.
> - List every `[N/A]` item that this code change might have re-activated (e.g. file added Stripe → all `[N/A: no payments]` items are now suspect).
> - This section makes it provable which parts of the project's threat model were actually exercised.

## Output

Present the agent's findings **directly and unfiltered**. Do not soften, do not editorialize.

Prepend a one-line summary:

```
PRISM-DEVIL — {target} — {count} findings ({critical}C / {high}H / {medium}M / {low}L) — chains: {n} — checklist: {hit}/{active_total} active items mapped, {gaps} new gaps
```

If no checklist was loaded, drop the `checklist:` segment.

## Rules

- Single agent, single pass. No verification — this tool is deliberately skeptical, and verifying every finding defeats the purpose.
- If you disagree with the agent, say so in your own response — do not edit the agent's report.
- This skill is allowed to cry wolf. Pair with `/prism` (default mode runs the Evidence pass) when you want signal over noise.
- Security-sensitive targets preferred: authentication, authorization, data mutations, external input parsers, anything touching money/PII/credentials. Non-security code works too but expect lower signal.

## When to use which

| Situation | Tool |
|---|---|
| Standard review, any target | `/prism <target>` |
| Broad review, high-stakes decision | `/prism <target>` (default verify) |
| You know you're too close to the code | `/prism <target> --adversarial` |
| Security-sensitive path, need attacker mindset | **`/prism-devil <target>`** |
| Pre-production gate on auth/payment/PII code | **`/prism-devil`** then `/prism` |
| Quick sanity check, speed > depth | `/prism <target> --quick` |
