# Prism-Devil — 공격적 적대 검증

**Language**: [English](README.md) · **한국어**

> `prism`은 5각도로 둘러본다. `prism-devil`은 검은 모자 쓰고 부수려고 든다.

---

## 이런 분을 위한 도구입니다

- auth/결제/PII/비밀 다루는 코드의 취약점이 걱정되는 분
- 외부 입력을 파싱하는 코드 (xlsx, pdf, image, EML)
- Pre-production 게이트로 공격자 마인드 검토가 필요한 분
- 자기 코드에 너무 가까워서 구멍이 안 보일 때
- 인프라 코드 정기 보안 감사

## 이런 작업엔 쓰지 마세요

- 일반 코드 리뷰 → `prism` (균형 잡힌 5-agent)
- markdown/스펙 → `triad` (3관점 숙의)
- 순수 성능/설계 질문 → 범위 밖

## "공격적"이라는 게 무슨 뜻

- **칭찬 금지** — agent에게 명시적으로 sugarcoat/balance 금지 지시
- **Scenario 강제** — 모든 finding은 단계별 공격/장애 trigger 포함 ("이거 위험할 수 있음" 같은 모호한 표현 거부)
- **엄격한 severity 룰** — CRITICAL은 진짜 remote exploit / data loss / auth bypass에만 부여. 생산성 보이려고 dilution 금지
- **Attack chain 탐지** — 2개 findings 조합해서 더 큰 피해 (예: ID 누출 + 권한 누락 = 계정 탈취) 명시
- **체크리스트 cross-reference** — 모든 finding을 프로젝트 체크리스트 항목에 매핑. 매핑 안 되면 `[not-in-checklist]` 태그 (체크리스트 자체 확장 신호)

## 자동 probe되는 10개 공격 표면 (우선순위 순)

1. **Trust boundaries** — 신뢰 안 되는 input이 신뢰되는 로직으로 넘어가는 모든 지점
2. **Authentication & authorization** — session, token, privilege, race
3. **Concurrency weaponization** — TOCTOU, double-submit, lost update
4. **Data integrity attacks** — FK cascade 악용, 제약 우회, audit log 변조
5. **Failure mode exploitation** — crash 유발해 도달하는 state, retry 증폭, replay
6. **Secrets & disclosure** — log 누출, timing side channel, stack trace
7. **Dependency & supply chain** — pinning gap, untrusted file 파싱, deserialization
8. **Self-evaluation bias** — self-grading 메트릭, Goodhart, false success
9. **False confidence** — happy-path-only test, silent except, log-and-continue
10. **Platform / environment** — Windows vs POSIX, timezone, locale 파서

## 출력 포맷 (finding당)

```
[ID] — severity: CRITICAL | HIGH | MEDIUM | LOW — class: <attack_class>
  Scenario: <단계별 공격 / 장애 trigger>
  Why it lands: <어느 line / 누락된 check / 잘못된 가정>
  Impact: <공격자가 얻는 것 / 망가지는 것>
  Fix: <구체적 최소 mitigation — "validation 추가" 같은 거 금지>
  Confidence: high | med | low
  Checklist: <체크리스트 항목 번호/태그 or [not-in-checklist] or [N/A: skipped]>
```

추가로 **Attack Chain Section** (findings 조합 시), **Checklist Coverage Section** (어느 `[ACTIVE]` 항목을 실제로 probe했는지, 어느 `[N/A]` 항목이 다시 활성화될 수 있는지).

## Top-line 요약

```
PRISM-DEVIL — src/services/auth.py — 7 findings (2C / 3H / 1M / 1L) — chains: 1 — checklist: 12/77 active items mapped, 2 new gaps
```

## 프로젝트 체크리스트 지원

프로젝트에 이런 형식의 체크리스트가 있으면:

```markdown
- [ACTIVE] #14 — 모든 session cookie에 Secure + HttpOnly + SameSite
- [N/A: 결제 없음] #41 — Stripe webhook signature 검증
- [FUTURE: AWS 마이그레이션 시] #59 — service별 IAM least-privilege
```

prism-devil이:
- finding을 항목 번호에 매핑
- 새로 추가된 코드가 `[N/A]` 항목을 재활성화하면 알림 (예: Stripe 추가됨 → `[N/A: 결제 없음]` 항목 모두 재검토)
- 시도했지만 깨끗한 항목은 `verified-clean` 표시
- 체크리스트 밖 findings는 `[not-in-checklist]` 태그 — 이게 곧 체크리스트 확장 신호

체크리스트 없으면 generic attack surface로 fallback. 절대 가짜로 만들지 않음.

## `/prism`과의 페어링

`prism-devil`은 verifier가 없어서 false positive 가능 — 의도된 설계 (적대성 보존). 보안 민감 코드의 high-confidence 흐름:

```
/prism-devil src/security/auth.py     # paranoid pass — worst case 리스트 생성
/prism src/security/auth.py           # 5-agent verified pass — false positive 필터
```

교집합이 high-signal. `prism-devil`-only findings도 읽어볼 가치는 있지만 verdict가 아닌 의심으로 다룰 것.

## 자매 도구

- **[prism](https://github.com/minwoo-data/prism)** — 5-agent verified 광범위 리뷰 (부모 도구)
- **[mangchi](https://github.com/minwoo-data/mangchi)** — Codex CLI와 cross-model 반복 다듬기
- **[triad](https://github.com/minwoo-data/triad)** — markdown용 3관점 숙의

## 설치

```
~/.claude/skills/prism-devil/SKILL.md
```

또는 `commands/prism-devil.md`를 `.claude/commands/` (프로젝트) / `~/.claude/commands/` (글로벌)에 두면 슬래시 명령.

## Created by

Minwoo Park
