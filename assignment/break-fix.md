# Break → Fix → Re-run

> **I changed the World, not the prompt.**

## 1. BREAK — World를 깨뜨려 보기

공격 유형: **셀프 승인** — Hypothesis를 제안한 사람이 스스로 그 Hypothesis의 발행(publish)까지
승인한다. (`../scenarios/adversarial-self-approval.yaml`)

실제 업무는 혼자 일하는 구조라서, `lead`(최종 발행 승인자) 역할을 실무에서 실존하는 다른
사람으로 분리할 수 없다 — analyst1이 analyst와 lead 역할을 **둘 다** 가져야 한다. 처음에는
`publish` Transition에 `principal_role: lead`만 걸고, 그 역할 분리만으로 셀프승인이 막힐
거라고 가정했다.

## 2. BEFORE

| 항목 | 기록 |
|---|---|
| 시나리오 (파일 · 이름) | `scenarios/adversarial-self-approval.yaml` · proposer-cannot-self-publish |
| 지키려는 안전 속성 | "Hypothesis를 제안한 사람은 스스로 그것을 발행할 수 없다" |
| 시작 레코드와 상태 | H-003, status: insight_ready, proposed_by: analyst1 |
| 행위자 / 역할 | analyst1 — roles: [analyst, lead] (혼자 일하는 구조라 두 역할을 겸함) |
| 시도한 Transition | publish (principal_role: lead, guards: 없음) |
| 관련 Constraint | 없음 |
| **예상 판정** | **ALLOW** |
| 이유 (desk-check) | State: insight_ready = publish.from → 통과. Role: analyst1은 lead 역할도 갖고 있어서 → 통과. Guard: 선언된 게 없어서 통과할 것이 없음 → ALLOW. |
| 문제 | ALLOW — "제안자 본인은 발행할 수 없다"는 규칙이 role 분리에만 암묵적으로 의존하고 있었는데, 혼자 일하는 구조에서는 그 분리가 성립하지 않아서 그대로 뚫린다. |

## 3. FIX — World 규칙을 바꾸기

프롬프트가 아니라 `world.yaml`의 Constraint와 `publish` Transition의 guard를 바꿨다.

- 바꾼 것: 새 Constraint `C8_hyp_no_self_publish` 추가, `publish`의 `guards`에 연결.
- 변경 전 → 변경 후:

```yaml
# before
transitions:
  publish:
    from: insight_ready
    to: published
    principal_role: lead
    guards: []

# after
transitions:
  publish:
    from: insight_ready
    to: published
    principal_role: lead
    guards: [C8_hyp_no_self_publish]

constraints:
  C8_hyp_no_self_publish:
    expr: principal.id != hypothesis.proposed_by
    message: The author cannot publish their own insight; a lead must review it.
```

- `npm run validate` 결과 (수정 후) → `../evidence/`에 저장 (SUPERWORK VALIDATION — Your world has no issues. ✓)

## 4. AFTER — 같은 시나리오, 다시 판정

| 항목 | 기록 |
|---|---|
| 같은 시나리오 | `scenarios/adversarial-self-approval.yaml` · proposer-cannot-self-publish |
| **새 예상 판정** | **DENY (C8_hyp_no_self_publish)** |
| 이유 — 이제 어떤 규칙이 판정을 바꾸는가 | State·Role은 이전과 동일하게 통과하지만, Guard C8에서 `principal.id(analyst1) != hypothesis.proposed_by(analyst1)` → false → DENY. role 분리가 아니라 **신원 비교 Guard**가 실제로 셀프승인을 막는 유일한 장치가 됨. |

## 5. 리뷰에서 (강사 기록란 — 비워 두세요)

| | 여러분의 예측 | 실제 Runtime 결과 |
|---|---|---|
| BEFORE | ALLOW | |
| AFTER | DENY (C8_hyp_no_self_publish) | |
