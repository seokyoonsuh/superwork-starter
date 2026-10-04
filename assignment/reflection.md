# 회고

1. **모델링하며 가장 어려웠던 결정은 무엇이었고, 왜 그렇게 결정했나요?**
   Problem·Hypothesis·Insight를 각각 별도 Entity로 둘지였다. 처음엔 3개로 나눠서 시작했는데,
   실제 프로젝트 사례(한 커머스 클라이언트에서 "핵심 문제"를 탐색 중간에 발견한 경우, 여러
   가설을 종합해서 "관점"을 만든 경우)를 되짚어보니 셋 다 "하나의 claim이 검증·가치평가를
   얼마나 통과했는지"의 차이였을 뿐, 본질적으로 다른 종류의 객체가 아니라는 걸 발견했다.
   그래서 Hypothesis 하나의 생애주기(`candidate→...→published`)로 합쳤다.

2. **World 없이 에이전트에게 이 업무를 맡겼다면 무엇이 잘못될 수 있었을까요?**
   증거 없는 가설을 그대로 SUPPORTED로 처리하거나, 참이지만 뻔한 사실을 그대로 고객에게
   Insight로 전달하거나, 가설을 제안한 사람이 스스로 그걸 발행까지 승인해버리는 일이
   매번 다르게(에이전트의 그날 판단에 따라) 일어날 수 있었을 것이다.

3. **BREAK에서 무엇을 발견했나요? 그 빈틈은 프롬프트를 고쳐서도 막을 수 있었을까요? 왜 World 규칙으로 막는 것이 다른가요?**
   "혼자 일하는 구조라 analyst와 lead 역할을 한 사람이 겸해야 하는데, `publish`를 역할(role)
   체크만으로 막으면 셀프승인이 그대로 뚫린다"는 빈틈을 발견했다. 프롬프트로 "본인 가설은
   스스로 승인하지 마세요"라고 적어도 지켜질 거라는 보장이 없지만, `principal.id !=
   hypothesis.proposed_by` Guard는 역할이 어떻게 조합되든 기계적으로 항상 같게 적용된다.

4. **validate가 통과했는데도 설계가 틀릴 수 있다는 것을 어떻게 경험했나요?**
   `publish`의 guard를 빼버린 BEFORE 버전도 `npm run validate`가 그대로 "통과"했다. 스키마가
   맞다는 것과 업무 규칙이 안전하다는 것은 완전히 다른 질문이라는 걸 직접 확인했다
   (`evidence/validate-before.txt`, `validate-after.txt` 둘 다 PASS인데 안전성은 다름).

5. **이 World를 한 단계 더 발전시킨다면 무엇을 추가하겠나요?**
   지금은 `builds_on`/`primary_evidence`가 Hypothesis 1개만 가리킬 수 있는데, 실제로는
   3~4개의 Hypothesis·Observation을 종합하는 경우가 많아서 이 관계를 제대로 구조화하고 싶다.
   또한 "client_objective/google_objective 중 최소 하나에 영향을 줘야 insight로 qualify된다"는
   Guard를 추가해서, 광고주와 Google 양쪽의 win을 같이 고려하는 실제 업무 특성을 반영하고 싶다.
