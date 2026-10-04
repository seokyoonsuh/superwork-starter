# Entity 결정 — 가장 중요한 과제

## 결정 기록

| 후보 개념 | 최종 분류 | 이유 (체크리스트 중 무엇이 결정적이었나) |
|---|---|---|
| Hypothesis | **Entity** (owned_state) | 고유 identity, candidate→...→published까지 지속, 의미 있는 상태변화, qualify/support/reject/promote_insight/publish가 작용, Guard(정책)가 직접 가리키는 대상. 체크리스트 6개 전부 "예". |
| Project | **Entity** (owned_state 없음) | client+period로 구분되는 고유 identity, Task 여러 개를 묶는 상위 컨텍스트로 지속. 다만 이 World 범위에서 Project 자체의 상태변화는 다루지 않음 (Transition/Guard 없음). |
| Task | **Entity** (owned_state 없음) | K-Beauty 브랜드 고객사 사례(한 Project 안에 "남성유저 헤드룸" / "서브 브랜드 마케팅 전략" 2개 Task가 동시 존재)에서, Project 하나로는 Hypothesis들을 구분할 수 없다는 게 드러남. Hypothesis가 어느 질문에 귀속되는지 식별하려면 독립된 identity가 필요. |
| Observation | **Entity** (owned_state 없음, 조건부 존재) | 고유 identity(metric+period+source), `Hypothesis.primary_evidence`로 실제 참조됨. 텍스트 속성으로 접지 않고 별도 Entity로 둔 이유 3가지: (1) 여러 Hypothesis가 같은 사실을 중복 타이핑 없이 공유 — 수치가 정정되면 한 곳만 고치면 됨 (예: H-001과 H-003이 같은 O-001을 공유), (2) Source of Truth(어느 출처·어느 기간에서 왔는지)를 추적 가능하게 함, (3) 관찰된 사실(fact)과 해석/주장(claim, =Hypothesis)을 구조적으로 분리. 단, "분석 중 스쳐간 모든 숫자"가 아니라 "Hypothesis가 실제로 인용했을 때만" World에 존재한다는 경계를 둬서 상태기계는 없음. |
| Inquiry / Business Question | **제외** (Entity 아님) | 프로젝트당 0~2개로 빈도가 너무 낮고, 독립적인 lifecycle이나 "여러 객체가 구분해서 참조해야 할 필요"가 약함. Project/Task의 속성(ask_text)으로 충분히 흡수됨. |
| BusinessProblem | **제외** (Entity 아님, Hypothesis로 흡수) | 한 커머스 클라이언트 프로젝트에서 "탐색형 쇼핑 참여가 무너지고 있다"는 문제를 발견한 사례에서, "핵심 문제를 발견한다"는 행위가 "가설을 제안하고 검증한다"는 행위와 본질적으로 다르지 않다는 걸 확인함. "GMV는 지켰어도 앱 인게이지먼트는 무너졌을 것이다"라는 Hypothesis가 SUPPORTED되면 그게 곧 핵심 문제 프레임이 됨 — 별도 Entity로 분리할 근거가 사라짐. |
| InsightCandidate | **제외** (Entity 아님, Hypothesis의 published 상태로 흡수) | "Insight"는 Hypothesis와 다른 종류의 객체가 아니라, Hypothesis가 novelty/materiality/decision_relevance Guard까지 통과해서 `published`까지 올라간 것일 뿐. 여러 Hypothesis를 종합한 insight도 그 자체로 독립된 테스트 가능한 Hypothesis(`builds_on`으로 재료가 된 다른 Hypothesis를 가리킴)로 표현 가능해서, 별도 Entity가 필요 없어짐. |
| Opportunity | **제외** (Entity 아님) | BusinessProblem과 같은 이유 — "기회 발견"도 결국 테스트 가능한 Hypothesis 제안/검증일 뿐이라 별도 분리 불필요. |
| Analyst / Lead | **Role** (Entity 아님) | 행동하는 주체의 역할일 뿐, 고유하게 추적해야 하는 "것"이 아님. `principals`의 `roles`로 표현. |

최종 **Entity 4개** (Project, Task, Observation, Hypothesis) / **Entity 아닌 개념 5개**
(Inquiry, BusinessProblem, InsightCandidate, Opportunity, Analyst·Lead Role) — 최소 요건(3개/2개) 충족.

## 주인공 Entity

- 주인공 Entity: **Hypothesis**
- 왜 이것인가: 이 World의 Work(Business Insight Discovery)가 실제로 하는 일이 "가설을 만들고,
  검증하고, 그중 고객에게 전달할 가치가 있는 것만 추려내는 것"이기 때문에, 모든 의미 있는
  상태변화·Guard·권한 규칙이 Hypothesis 하나에 집중된다. 또한 `superwork.world/v1`의
  states/transitions가 Entity별로 scoping되지 않는(전역 state 이름 집합을 쓰는) 구조라서,
  owned_state Entity를 여러 개로 나누는 것은 검증되지 않은 리스크로 보고 Hypothesis 하나로
  의도적으로 좁혔다 (자세한 이유는 modeling-decisions.md 결정 1 참고).
