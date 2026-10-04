# 애매한 모델링 결정

## 결정 1

- **개념:** "핵심 문제 발견(Problem)"과 "고객 전달용 관점(Insight)"을 Hypothesis와 별도 Entity로 둘 것인가
- **고려한 대안:**
  - Problem과 Insight를 각각 독립된 owned_state Entity로 둔다 (Hypothesis, BusinessProblem, InsightCandidate — 3개)
  - Hypothesis 하나의 State 체인 안에 "진실 검증"과 "발행 가치 평가"를 이어 붙인다
  - Hypothesis 안에 두 개의 독립된 state 축(validation_state / insight_status)을 둔다
- **최종 선택:** 두 번째 — Hypothesis 하나의 단일 State 체인
  (`candidate→qualified→investigating→supported/rejected/inconclusive→insight_ready→published`)
- **이유:** 한 커머스 클라이언트 프로젝트에서 "탐색형 쇼핑 참여가 무너지고 있다"는 문제를
  발견한 사례를 되짚어보니, "문제 발견"이 가설을 제안하고 검증하는 것과 본질적으로
  같은 행위라는 걸 확인했고, "성장 병목은 acquisition volume이 아니라 quality다" 같은 종합
  insight도 그 자체로 독립된 테스트 가능한 Hypothesis(`builds_on`으로 재료 Hypothesis를 가리킴)로
  표현할 수 있음을 확인했다. 세 번째 대안(두 축)은 애초에 `superwork.world/v1`이 Entity당
  state 목록을 하나만 지원해서 기술적으로 불가능했다. 첫 번째 대안(별도 Entity 3개)은
  validator가 모든 Entity의 state 이름을 전역(flat) 집합으로 취급하는 걸 확인하면서, 검증되지
  않은 리스크로 보고 피했다.
- **ESTC에 미친 영향:**
  - Entity: Project/Task/Observation/Hypothesis 4개로 정리, owned_state는 Hypothesis 하나.
  - State: Hypothesis에 8개 상태(`insight_ready` 포함) 집중.
  - Transition: `promote_insight`, `publish`가 Hypothesis 자신의 Transition이 됨.
  - Constraint: novelty/materiality/decision_relevance가 별도 Entity 속성이 아니라 Hypothesis
    자신의 속성이자 Guard가 됨.

## 결정 2

- **개념:** Agent가 자유롭게 만든 가설 후보를 World에 어느 시점부터 받아들일 것인가
  (Qualification Gate를 World가 실제로 집행할 것인가)
- **고려한 대안:**
  - PROPOSED를 World State에서 완전히 빼고 Agent reasoning layer에만 둔다 (World는 이미
    검증된 것처럼 보이는 것부터 시작)
  - `candidate`라는 짧은 World State를 두고, `qualify` Transition에 Guard(relevant/testable/
    decision_value)를 걸어서 World가 직접 "검증할 가치가 있는가"를 판정하게 한다
- **최종 선택:** 두 번째 — `candidate` state + `qualify` Transition + Guard 3개
- **이유:** 첫 번째 대안으로 가면 "Agent는 자유롭게 제안하되, World는 최소 기준을 통과한 것만
  받는다"는 원칙이 말로만 있고 실제로 아무도 지키지 않아도 되는 상태가 된다. Guard는 반드시
  Transition에 붙어야 한다는 이 DSL의 제약을 확인한 뒤로는, "World가 실제로 검증을 집행한다"는
  걸 보여주려면 Transition이 있는 두 번째 방식이 유일한 선택이었다.
- **ESTC에 미친 영향:**
  - Entity: 영향 없음 (Hypothesis 그대로).
  - State: `candidate` 추가 (Hypothesis의 시작 상태).
  - Transition: `qualify` (candidate→qualified) 추가.
  - Constraint: `C1_hyp_relevant`, `C2_hyp_testable`, `C3_hyp_decision_value` 추가.

## 결정 3

- **개념:** Insight 발행 시 "셀프승인 금지"를 어떻게 모델링할 것인가
- **고려한 대안:**
  - Authority 레벨에서 역할(role)만 분리한다 (`publish`는 `lead` role만 가능)
  - 역할 분리에 더해, Guard로 `principal.id != hypothesis.proposed_by`를 직접 비교한다
- **최종 선택:** 두 번째 — role 제한 + Guard 비교 둘 다
- **이유:** role만으로는 "제안자 본인이 다른 상황에서 lead 역할도 겸하게 되는" 경우를 못 막는다.
  실제로는 혼자 일하는 구조라 지금 당장 `lead`라는 역할이 실존하지는 않지만, 확증편향을 막기
  위해 "이상적으로 있어야 하는 규칙"으로 의도적으로 도입하기로 했다.
- **ESTC에 미친 영향:**
  - Entity: 영향 없음.
  - State: 영향 없음.
  - Transition: `publish`의 `principal_role`을 `lead`로 지정.
  - Constraint: `C8_hyp_no_self_publish` 추가.
  - Principal: `lead1`(가상 role) 추가.
