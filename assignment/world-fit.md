# World Fit — 왜 이 업무에 World Model이 필요한가

## 내가 고른 업무

- 업무 이름: Business Insight Discovery (Google Insight Specialist의 가설 검증 기반 인사이트 발굴 업무)
- 한 문장 설명 (이 World가 완수하려는 일 = `world.yaml`의 mission):
  Qualify candidate hypotheses with real evidence before testing them, confirm or reject them
  against observed data, and allow only the validated findings that are non-obvious and
  independently reviewed to be published as client-facing insight.

## 질문

1. 에이전트가 시간이 지나도 다시 돌아오는 실제 대상이 있는가?
   → **예.** 같은 Hypothesis가 제안 → 검증 → (때로는) 발행까지 여러 날/여러 세션에 걸쳐 같은 identity로 추적됨.

2. 그 대상의 상태가 의미 있게 바뀌는가?
   → **예.** candidate → qualified → investigating → supported/rejected/inconclusive →
   insight_ready → published. 각 단계가 "지금 이 claim에 대해 무엇을 믿을 수 있는지"를 바꿈.

3. 어떤 행동이 다음에 가능한 일을 바꾸는가?
   → **예.** qualify를 통과 못한 가설은 investigate할 수 없고, 근거(Observation) 없이는
   support/reject 판정을 못 내리고, SUPPORTED가 안 된 가설은 insight로 못 올라감.

4. 역할·승인·한도·정책이 중요한가?
   → **예.** 발행(publish)은 제안자 본인이 아닌 lead만 가능 — 혼자 일하는 구조라 실무에
   실제로 있는 규칙은 아니지만, 확증편향을 막기 위해 "있어야 하는 규칙"으로 도입.

5. 되돌리기 어렵거나 비용이 큰 외부 효과가 있는가?
   → **예, 간접적으로.** published된 insight가 고객에게 전달되는 것 자체가 되돌리기 어려운
   효과임 — 한번 전달한 관점을 "사실 아니었다"고 철회하면 고객 신뢰에 타격이 큼. 그래서
   published 이전에 novelty/materiality/decision_relevance를 미리 걸러내는 게 중요함.

6. 여러 사람·에이전트·시스템이 같은 현실에 대해 행동하는가?
   → **예.** Agent는 자유롭게 가설을 제안하지만, 그게 공식적으로 "검증됨" 또는
   "발행 가능"으로 인정되는지는 World(Guard)가 결정하고, 최종 발행은 별도 역할(lead)이 함.

7. 실제로 일어나기 전에 가능한 미래를 시험해 보는 것이 가치 있는가?
   → **예.** "이 가설이 뻔한 얘기는 아닌지", "증거가 충분한지"를 고객에게 전달하기 **전에**
   미리 걸러내는 것 자체가 이 업무의 핵심 가치임 (뻔한 인사이트를 전달하면 신뢰를 잃음).

## 결론

이 업무는 "한 번 묻고 답하면 끝"이 아니라, 같은 Hypothesis가 여러 단계(제안 → 검증 →
발행 가치 평가 → 발행)를 거치며 상태가 바뀌고, 각 단계 전환마다 통과해야 하는 조건이 다르다.
특히 "검증된 사실(SUPPORTED)"과 "고객에게 전달할 가치가 있는 관점(published)"은 서로 다른
축의 판단이라서, 이 둘을 분리해서 강제하지 않으면 Agent가 참이지만 뻔한 주장을 그대로
고객에게 전달해버리는 위험이 생긴다. World 없이 프롬프트로만 "뻔한 인사이트는 만들지 마"라고
지시하면 이 규칙이 매번 지켜질 거라고 보장할 수 없고, 누가 발행을 승인했는지에 대한 권한
구분도 프롬프트로는 강제되지 않는다. 그래서 이 업무는 상태·전환·근거·권한을 World가
결정론적으로 관리해야 하는 업무라고 판단했다.
