# Break → Fix → Re-run

> **I changed the World, not the prompt.**

## 1. BREAK — World를 깨뜨려 보기

공격 유형: **선행 조건 누락** — "클라이언트에게 사실이고 의미 있는 주장"이라는 조건만으로
insight로 승격돼버리고, 이 업무의 가장 중요한 원칙 중 하나인 "광고주의 win뿐 아니라
Google의 win도 함께 고려한다"는 조건은 전혀 체크하지 않는다.
(`../scenarios/adversarial-google-objective.yaml`)

처음에는 "본인이 제안한 가설을 본인이 발행까지 승인하는" 셀프승인 문제를 찾았는데, 혼자
일하는 구조라 실질적으로는 덜 중요했다. 그래서 실제 세일즈팀 요청 케이스(클라이언트가
효율화 기조로 전환하며 특정 채널 예산을 줄이려 하고, 공격적 투자를 설득할 데이터 기반
내러티브가 필요했던 사례 — 익명화)를 대입해서 다시 찾아보니, 더 본질적인 빈틈이 나왔다.

## 2. BEFORE

| 항목 | 기록 |
|---|---|
| 시나리오 (파일 · 이름) | `scenarios/adversarial-google-objective.yaml` · insight-against-google-objective-should-not-promote |
| 지키려는 안전 속성 | "클라이언트에게 유리해도 Google의 objective에 반하는 주장은 insight로 승격되면 안 된다" |
| 시작 레코드와 상태 | H-004, status: supported, statement: "특정 채널 예산을 줄이는 것이 클라이언트의 ROAS를 추가로 개선한다", advances_google_objective: no |
| 행위자 / 역할 | analyst1 — roles: [analyst, lead] |
| 시도한 Transition | promote_insight (principal_role: analyst, guards: [C5_hyp_novelty, C6_hyp_materiality, C7_hyp_decision_relevance]) |
| 관련 Constraint | 없음 (Google objective를 체크하는 Constraint 자체가 존재하지 않았음) |
| **예상 판정** | **ALLOW** |
| 이유 (desk-check) | State: supported = promote_insight.from → 통과. Role: analyst는 principal_role과 일치 → 통과. Guards: novelty(high)/materiality(high)/decision_relevance(high) 전부 통과 → ALLOW. |
| 문제 | ALLOW — 이 Hypothesis는 클라이언트 입장에서는 사실이고 의미 있지만, 결론이 "Google 광고비를 더 줄이자"라서 Google의 objective와 정반대다. "광고주 win + Google win을 함께 고려한다"는 이 업무의 가장 기본적인 원칙이 Guard 어디에도 반영돼 있지 않았다. |

## 3. FIX — World 규칙을 바꾸기

프롬프트가 아니라 `world.yaml`의 Constraint와 `promote_insight` Transition의 guard를 바꿨다.

- 바꾼 것: 새 속성 `Hypothesis.advances_google_objective` 추가, 새 Constraint
  `C9_hyp_not_against_google` 추가, `promote_insight`의 `guards`에 연결.
- 설계하면서 한 번 더 고친 부분: 처음엔 `advances_google_objective`를 `yes`/`no` 둘 중
  하나만 받게 하려 했는데, 성격이 다른 실제 케이스(순수 가격·운영 의사결정 — 프로모션
  유지 여부, Google 광고와 아예 무관)를 대입해보니 "굳이 안 도와도 되는" 경우까지 억지로
  `yes`를 요구하게 돼서 과도하게 막아버린다는 걸 발견했다. 그래서 `not_applicable`을 추가해
  "안 도와도 되지만, 적극적으로 해쳐서는 안 된다"로 Guard를 완화했다
  (`../scenarios/google-objective-not-applicable-check.yaml`에서 이 케이스가 정상적으로
  ALLOW 되는 것까지 확인).
- 변경 전 → 변경 후:

```yaml
# before
entities:
  Hypothesis:
    attributes:
      # ...
      decision_relevance: string
      status: state

transitions:
  promote_insight:
    guards: [C5_hyp_novelty, C6_hyp_materiality, C7_hyp_decision_relevance]

# after
entities:
  Hypothesis:
    attributes:
      # ...
      decision_relevance: string
      advances_google_objective: string   # yes | no | not_applicable
      status: state

transitions:
  promote_insight:
    guards: [C5_hyp_novelty, C6_hyp_materiality, C7_hyp_decision_relevance, C9_hyp_not_against_google]

constraints:
  C9_hyp_not_against_google:
    expr: hypothesis.advances_google_objective in ["yes", "not_applicable"]
    message: An insight that works against Google's own objective cannot be promoted — advancing the client's win alone is not enough.
```

- `npm run validate` 결과 (수정 전/후) → `../evidence/validate-before.txt`, `../evidence/validate-after.txt`
  (둘 다 "READY FOR STUDIO IMPORT ✓" — **스키마는 수정 전에도 이미 유효했다.**)

## 4. AFTER — 같은 시나리오, 다시 판정

| 항목 | 기록 |
|---|---|
| 같은 시나리오 | `scenarios/adversarial-google-objective.yaml` · insight-against-google-objective-should-not-promote |
| **새 예상 판정** | **DENY (C9_hyp_not_against_google)** |
| 이유 — 이제 어떤 규칙이 판정을 바꾸는가 | State·Role·C5/C6/C7은 이전과 동일하게 통과하지만, Guard C9에서 `advances_google_objective(no) in [yes, not_applicable]` → false → DENY. 동시에 `scenarios/google-objective-not-applicable-check.yaml`(H-005, Google과 무관한 순수 운영 결정)은 같은 Guard에서 `not_applicable`이 허용돼 그대로 ALLOW — 과잉 차단 없이 "Google 이익에 반하는 경우만" 정확히 막는다. |

## 5. 리뷰에서 (강사 기록란 — 비워 두세요)

| | 여러분의 예측 | 실제 Runtime 결과 |
|---|---|---|
| BEFORE | ALLOW | |
| AFTER | DENY (C9_hyp_not_against_google) | |

---

## 참고 — 추가로 발견한 빈틈 (보조)

셀프승인 문제도 별도로 발견해서 고쳐뒀다 (`scenarios/adversarial-self-approval.yaml`): 혼자
일하는 구조라 `analyst1`이 `analyst`/`lead` 역할을 둘 다 가져야 하는데, `publish`를 역할
체크만으로 막으면 본인이 제안한 Hypothesis를 본인이 그대로 발행할 수 있었다.
`C8_hyp_no_self_publish: principal.id != hypothesis.proposed_by` Guard로 막아뒀다. 다만
이건 위 Google-objective 빈틈보다 업무적으로 덜 중요하다고 판단해서 메인 BREAK 사례로는
다루지 않았다.
