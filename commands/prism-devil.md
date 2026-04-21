---
name: prism-devil
description: "Aggressive attacker-mindset vulnerability hunt — single-agent deep adversarial probe. Auto-loads project security checklist when present."
argument-hint: "[file-or-topic]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Task
---

# /prism-devil Command

Single-agent aggressive red-team probe focused on attack surface, weaknesses, and failure modes. The agent puts on the black hat and tries to break the target on paper.

## Parse Arguments

| Argument Pattern | Action |
|---|---|
| `<target>` | Probe that file/topic with attacker mindset |
| `(no argument)` | Probe the most recently discussed design/code in conversation |

## Pre-flight context loading

Before spawning the agent, load project-specific context (≤3 tool calls):

1. **Project security checklist** — first hit wins:
   - `docs/security-checklist.md`
   - `docs/security/checklist.md`
   - `SECURITY.md`
   - `docs/SECURITY.md`
2. **Project CLAUDE.md** — for stack details and deferred-feature decisions
3. **The target file(s)** — full content, no summarization

If no checklist exists, fall back to generic attack surfaces. Never fabricate project context.

## Execution

The full agent prompt — including the 10 attack surface categories, severity rubric, output format, attack chain section, and checklist coverage section — lives in `skills/prism-devil/SKILL.md`. Read that file before executing.

## When to use

| Situation | Use prism-devil? |
|---|---|
| auth / payment / PII code | yes — primary use case |
| File parsers (xlsx, pdf, image) | yes — supply chain + injection surface |
| Pre-production gate | yes, then follow with `/prism --verify` |
| General code review | no — use `/prism` instead |
| Markdown / spec review | no — use `/triad` |

## Companion commands

- `/prism <target>` — broad 5-agent verified review
- `/mangchi <file>` — iterative file hardening with Codex CLI
- `/triad <file>` — 3-perspective deliberation for docs
