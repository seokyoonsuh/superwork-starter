# 회고

1. **모델링하며 가장 어려웠던 결정은 무엇이었고, 왜 그렇게 결정했나요?**
   Problem·Hypothesis·Insight를 각각 별도 Entity로 둘지였다. 처음엔 3개로 나눠서 시작했는데,
   실제 프로젝트 사례(한 커머스 클라이언트에서 "핵심 문제"를 탐색 중간에 발견한 경우, 여러
   가설을 종합해서 "관점"을 만든 경우)를 되짚어보니 셋 다 "하나의 claim이 검증·가치평가를
   얼마나 통과했는지"의 차이였을 뿐, 본질적으로 다른 종류의 객체가 아니라는 걸 발견했다.
   그래서 Hypothesis 하나의 생애주기(`candidate→...→published`)로 합쳤다.

2. **World 없이 에이전트에게 이 업무를 맡겼다면 무엇이 잘못될 수 있었을까요?**
      증거 없는 가설을 그대로 SUPPORTED로 처리하거나, 참이지만 뻔한 사실을 그대로 고객에게
   Insight로 전달하거나, Novelty가 없는 가설을 내게 전달한다든가 하는 경우가 일어날 수 있었을 것.

3. **BREAK에서 무엇을 발견했나요? 그 빈틈은 프롬프트를 고쳐서도 막을 수 있었을까요? 왜 World 규칙으로 막는 것이 다른가요?**
   처음엔 "본인이 제안한 가설을 본인이 발행까지 승인하는" 셀프승인 빈틈을 찾았는데, 혼자
   일하는 구조라 실질적으로는 덜 중요한 문제였다. 그래서 실제 클라이언트 케이스를 대입해
   다시 찾아보니 더 본질적인 빈틈이 나왔다 — novelty/materiality/decision_relevance를 다
   통과해도 "이 insight가 Google의 objective에는 반하지 않는가"는 전혀 체크하지 않고 있었다.
   예를 들어 "특정 채널 예산을 줄이면 클라이언트의 ROAS가 개선된다"는 가설은 지금 Guard를
   다 통과하지만, 결론이 Google 광고비 축소라서 애초에 이 업무가 지켜야 하는 "광고주 win +
   Google win" 원칙과 반대다. 프롬프트로 "Google에도 도움이 되는지 확인하세요"라고 적어도
   매번 챙겨질 거라는 보장이 없지만, `advances_google_objective in [yes, not_applicable]`
   Guard는 기계적으로 항상 같게 적용된다.

4. **validate가 통과했는데도 설계가 틀릴 수 있다는 것을 어떻게 경험했나요?**
   `publish`의 guard를 빼버린 BEFORE 버전도 `npm run validate`가 그대로 "통과"했다. 스키마가
   맞다는 것과 업무 규칙이 안전하다는 것은 완전히 다른 질문이라는 걸 직접 확인했다
   (`evidence/validate-before.txt`, `validate-after.txt` 둘 다 PASS인데 안전성은 다름).

5. **이 World를 한 단계 더 발전시킨다면 무엇을 추가하겠나요?**
   지금은 `builds_on`/`primary_evidence`가 Hypothesis 1개만 가리킬 수 있는데, 실제로는
   3~4개의 Hypothesis·Observation을 종합하는 경우가 많아서 이 관계를 제대로 구조화(배열
   참조 또는 별도 관계 구조)하고 싶다. 또한 `advances_google_objective`를 포함한 모든
   Qualification 속성이 지금은 분석가 본인이 스스로 적어넣는 self-assertion이라, 그 판단이
   실제로 맞는지는 World가 검증하지 못한다 — 과거 발행된 Insight들과 비교하거나 재검토
   이력을 남기는 식으로 이걸 보강할 수 있는지 다음에 고민해보고 싶다.
