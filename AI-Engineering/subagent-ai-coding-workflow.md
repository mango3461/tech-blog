# 서브에이전트를 활용한 AI 코딩 워크플로우

> **핵심 메시지**
> 
> 
> AI 코딩의 생산성은 더 이상 “하나의 agent에게 얼마나 좋은 prompt를 쓰는가”만으로 결정되지 않는다.
> 
> 복잡한 작업에서는 **여러 서브에이전트의 역할, context, 작업 순서, 검증 루프를 설계하는 orchestration 능력**이 중요해진다.
> 

---

# 1. 왜 서브에이전트가 필요한가

## 설명

하나의 AI coding agent에게도 많은 일을 맡길 수 있다.

하지만 작업이 커질수록 다음 문제가 생긴다.

```
- context가 길어져 중요한 정보가 희석됨
- 탐색, 구현, 리뷰가 한 agent 안에서 섞임
- 앞 단계의 가정이 뒤 단계에 계속 영향을 줌
- 병렬로 처리할 수 있는 일을 순차적으로 기다림
- 구현한 agent가 자기 diff를 과신함
```

서브에이전트는 이 문제를 줄이기 위한 운영 방식이다.

> **서브에이전트는 AI를 더 많이 쓰는 기술이 아니라, AI 작업을 더 작고 명확한 책임으로 나누는 기술이다.**
> 

예를 들어 “결제 버그를 고쳐줘”라는 작업은 하나의 agent에게 전부 맡길 수도 있다.

하지만 실제로는 여러 역할로 나눌 수 있다.

| 역할 | 책임 |
| --- | --- |
| Explorer | 관련 파일, 현재 구조, 의심 지점을 찾는다 |
| Worker | 정해진 범위 안에서 코드를 수정한다 |
| Reviewer | diff를 읽고 regression, edge case, 과한 변경을 찾는다 |
| Verifier | 테스트, 로그, 재현 시나리오를 확인한다 |

이렇게 나누면 agent 하나의 사고가 전체 작업을 지배하지 않는다.

---

# 2. 서브에이전트란 무엇인가

## 설명

서브에이전트는 단순히 “작업을 대신하는 또 다른 AI”가 아니다.

실무적으로는 다음 세 가지를 가진 작업 단위로 보는 게 좋다.

```
1. 고유한 역할
2. 제한된 context
3. 명확한 산출물
```

예:

```
Explorer agent:
- 역할: 관련 파일과 원인 후보 찾기
- context: 코드 읽기 중심
- 산출물: 관련 파일, 현재 동작, 의심 지점, 구현 제안

Worker agent:
- 역할: 승인된 범위 안에서 코드 수정
- context: 수정 대상 파일, 제약 조건, 완료 조건
- 산출물: 변경 파일, diff 요약, 테스트 결과

Reviewer agent:
- 역할: 변경사항 검토
- context: diff, 요구사항, 금지사항
- 산출물: 발견한 문제, 위험도, 수정 제안
```

핵심은 “agent 수”가 아니라 “책임 경계”다.

## 서브에이전트가 유용한 조건

| 조건 | 이유 |
| --- | --- |
| 작업이 여러 영역에 걸침 | 탐색과 구현을 분리할 가치가 있음 |
| 리스크가 높음 | 독립 리뷰가 필요함 |
| 관련 파일이 많음 | 탐색을 병렬화할 수 있음 |
| 테스트 범위가 넓음 | 검증 전략을 따로 설계해야 함 |
| 긴 작업임 | context 오염을 줄여야 함 |

## 서브에이전트가 불필요한 조건

| 조건 | 이유 |
| --- | --- |
| 단일 파일의 작은 수정 | orchestration 비용이 더 큼 |
| 정답이 명확한 단순 변경 | 역할 분리 이득이 작음 |
| 바로 다음 단계가 탐색 결과에 의존 | 메인 agent가 직접 처리하는 편이 빠름 |
| 요구사항이 아직 모호함 | 사람과 먼저 범위를 정해야 함 |

---

# 3. Context Engineering: agent마다 다른 context를 준다

## 설명

서브에이전트 workflow에서 가장 중요한 것은 context engineering이다.

하나의 큰 context를 모든 agent에게 똑같이 주면 역할 분리의 의미가 줄어든다.

> **좋은 orchestration은 agent마다 필요한 정보만 다르게 주는 것이다.**
> 

## 같은 작업, 다른 context

예를 들어 결제 할인 버그를 처리한다고 하자.

Explorer에게는 넓은 탐색 context가 필요하다.

```
목표:
checkout 할인 계산 흐름을 파악해줘.

볼 범위:
- src/checkout/**
- src/coupon/**
- tests/checkout/**

산출물:
- 관련 파일
- 할인 계산 흐름
- 의심 지점
- 수정 후보

주의:
- 아직 파일 수정하지 마.
```

Worker에게는 좁은 구현 context가 필요하다.

```
목표:
Explorer가 찾은 applyCoupon 중복 적용 문제를 최소 수정해줘.

수정 범위:
- src/checkout/applyCoupon.ts
- tests/checkout/coupon.test.ts

금지:
- public API 변경 금지
- DB schema 변경 금지
- 새 dependency 추가 금지

완료 조건:
- 중복 쿠폰 적용 테스트 추가
- checkout 관련 테스트 통과
```

Reviewer에게는 diff 중심 context가 필요하다.

```
목표:
현재 diff를 리뷰해줘.

확인할 것:
- 요구 범위를 넘어선 변경이 있는가
- public API가 바뀌었는가
- edge case가 빠졌는가
- 테스트가 실제 버그를 검증하는가

산출물:
- 심각도별 finding
- 수정이 필요한 파일/라인
- 남은 리스크
```

## Context를 나누는 기준

| 기준 | 질문 |
| --- | --- |
| 역할 | 이 agent는 무엇을 해야 하는가? |
| 범위 | 어디까지 봐야 하는가? |
| 금지 | 무엇은 하면 안 되는가? |
| 산출물 | 무엇을 남겨야 하는가? |
| 합류점 | 결과가 다음 agent에게 어떻게 전달되는가? |

## Context pollution 방지

서브에이전트의 장점은 context를 분리할 수 있다는 점이다.

하지만 잘못 쓰면 오히려 오염이 늘어난다.

나쁜 예:

```
모든 agent에게 전체 대화, 전체 로그, 전체 diff, 전체 요구사항을 다 전달함
```

좋은 예:

```
Explorer:
- 관련 코드와 문제 설명

Worker:
- 승인된 수정 계획과 수정 대상 파일

Reviewer:
- 요구사항과 diff

Verifier:
- 테스트 명령과 실패 로그 핵심 부분
```

---

# 4. Workflow Design: 작업을 단계로 설계한다

## 설명

서브에이전트는 workflow가 있을 때 효과가 난다.

workflow 없이 agent만 늘리면 다음처럼 된다.

```
- 여러 agent가 같은 파일을 동시에 고침
- 서로 다른 결론을 냄
- 누가 최종 판단했는지 불명확함
- 결과를 합치는 비용이 커짐
```

그래서 먼저 작업을 단계로 나눈다.

```
1. 문제 정의
2. 탐색
3. 계획
4. 구현
5. 검증
6. 리뷰
7. 통합
```

각 단계마다 agent를 꼭 따로 써야 하는 것은 아니다.

중요한 것은 단계의 책임이 섞이지 않게 하는 것이다.

## 기본 workflow

```
Human:
문제, 제약, 완료 조건 정의

Explorer:
관련 파일과 원인 후보 탐색

Human 또는 Main Agent:
수정 계획 선택

Worker:
정해진 범위 안에서 구현

Verifier:
테스트 실행과 실패 로그 분석

Reviewer:
diff 검토

Human:
최종 판단, merge 또는 추가 수정 결정
```

## 단계별 산출물

| 단계 | 산출물 |
| --- | --- |
| 문제 정의 | 목표, 범위, 금지사항, 완료 조건 |
| 탐색 | 관련 파일, 현재 동작, 원인 후보 |
| 계획 | 수정 대상, 구현 순서, 리스크, 테스트 전략 |
| 구현 | 변경 파일, diff 요약 |
| 검증 | 실행한 명령, 결과, 실패 로그 |
| 리뷰 | finding, severity, 수정 필요 여부 |
| 통합 | 최종 diff, 남은 리스크, 후속 작업 |

## Workflow prompt 예시

```
이 작업은 서브에이전트 workflow로 진행할 거야.

1. Explorer는 관련 파일과 원인 후보만 찾아줘. 수정 금지.
2. 내가 계획을 확인한 뒤 Worker가 최소 수정해.
3. 수정 후 Verifier는 관련 테스트를 실행하고 실패 로그를 요약해.
4. Reviewer는 diff를 요구사항 기준으로 리뷰해.

각 단계 산출물은 다음 agent가 바로 사용할 수 있게 짧고 구체적으로 작성해줘.
```

---

# 5. Orchestration: 순서, 병렬성, 합류 지점 설계

## 설명

Orchestration은 여러 agent를 그냥 실행하는 것이 아니다.

다음을 결정하는 일이다.

```
- 어떤 일을 병렬로 할 것인가
- 어떤 일은 순차로 할 것인가
- 누가 어떤 파일을 소유할 것인가
- 결과를 어디서 합칠 것인가
- 충돌이 나면 누가 판단할 것인가
```

> **Orchestration의 핵심은 agent를 많이 쓰는 것이 아니라, 의존성과 책임을 명확히 하는 것이다.**
> 

## 병렬로 하기 좋은 일

| 작업 | 이유 |
| --- | --- |
| 서로 다른 모듈 탐색 | 파일 충돌이 없음 |
| 테스트 실패 로그 분석 | 구현과 병렬 가능 |
| 문서/사용법 조사 | 코드 수정과 분리 가능 |
| 독립적인 리뷰 | 구현 agent와 context 분리 |
| frontend/backend 영향 분석 | 영역이 분리됨 |

## 병렬로 하면 위험한 일

| 작업 | 위험 |
| --- | --- |
| 같은 파일 수정 | merge conflict, 의도 충돌 |
| shared 타입 변경 | 여러 agent의 가정이 깨짐 |
| DB schema 변경 | 영향 범위가 넓음 |
| public API 변경 | frontend/backend 동시 파급 |
| 대규모 rename | 충돌 가능성이 큼 |

## 합류 지점

서브에이전트 workflow에서 가장 중요한 순간은 합류 지점이다.

```
Explorer 결과 -> 구현 계획으로 합류
Worker 결과 -> diff로 합류
Verifier 결과 -> pass/fail 판단으로 합류
Reviewer 결과 -> 수정 여부 결정으로 합류
```

각 합류 지점에는 사람이 개입하거나, 메인 agent가 판단해야 한다.

## Orchestration anti-pattern

나쁜 예:

```
Agent A: checkout 고쳐줘
Agent B: checkout 테스트 고쳐줘
Agent C: checkout 리팩토링해줘
```

문제:

```
- 모두 같은 영역을 건드림
- 책임이 겹침
- 결과를 합치기 어려움
- 테스트 실패 원인이 불명확함
```

좋은 예:

```
Agent A:
checkout 할인 계산 흐름을 읽고 원인 후보만 정리. 수정 금지.

Agent B:
coupon test에서 현재 실패를 재현할 수 있는 케이스 후보 정리. 수정 금지.

Main:
A와 B 결과를 바탕으로 수정 범위 결정.

Agent C:
정해진 파일만 수정.

Agent D:
diff 리뷰.
```

---

# 6. 역할 패턴

## Explorer

Explorer는 읽는 agent다.

```
목표:
- 코드 구조 파악
- 관련 파일 찾기
- 원인 후보 정리
- 구현 방향 제안

금지:
- 파일 수정
- 테스트 수정
- architecture 변경
```

좋은 prompt:

```
Explorer 역할로 진행해줘.
src/billing과 tests/billing을 읽고 subscription cancel 흐름을 파악해.
아직 수정하지 말고, 관련 파일과 의심 지점만 정리해줘.
```

## Worker

Worker는 수정하는 agent다.

```
목표:
- 승인된 계획 구현
- 지정된 파일만 수정
- 테스트 또는 검증 실행

금지:
- 계획에 없는 refactor
- 새 dependency 추가
- public contract 변경
```

좋은 prompt:

```
Worker 역할로 진행해줘.
수정 범위는 아래 파일로 제한해.

- src/billing/cancelSubscription.ts
- tests/billing/cancelSubscription.test.ts

Explorer의 원인 분석에 따라 cancel 호출 누락만 최소 수정해.
```

## Reviewer

Reviewer는 의심하는 agent다.

```
목표:
- diff의 문제 찾기
- 요구사항과의 불일치 찾기
- 테스트 누락 찾기
- regression 위험 찾기
```

좋은 prompt:

```
Reviewer 역할로 현재 diff를 봐줘.
칭찬이나 요약보다 finding을 먼저 제시해.
특히 public API 변경, edge case 누락, 불필요한 refactor를 확인해.
```

## Verifier

Verifier는 검증하는 agent다.

```
목표:
- 테스트 실행
- 실패 로그 요약
- 재현 여부 확인
- 검증 범위의 충분성 판단
```

좋은 prompt:

```
Verifier 역할로 진행해줘.
pnpm test -- billing을 실행하고,
실패하면 핵심 로그와 의심 원인을 정리해.
코드 수정은 하지 마.
```

## Planner

Planner는 작업을 나누는 agent다.

```
목표:
- 큰 작업을 작은 단계로 분해
- 병렬 가능한 일과 순차 작업 구분
- 각 agent의 책임과 파일 소유권 정의
```

좋은 prompt:

```
Planner 역할로 이 작업을 서브에이전트 workflow로 나눠줘.
각 단계마다 담당 agent, 입력 context, 산출물, 파일 소유권, 검증 방법을 포함해.
아직 구현하지 마.
```

---

# 7. 실전 시나리오

## 시나리오 1. 버그 수정

### 문제

```
회원 탈퇴 후에도 subscription이 active로 남는다.
```

### Workflow

```
1. Explorer:
   account deletion과 billing cancel 흐름 탐색

2. Main:
   원인 후보 중 수정 범위 결정

3. Worker:
   cancel 호출 누락 또는 transaction 순서 문제 최소 수정

4. Verifier:
   관련 테스트 실행

5. Reviewer:
   public API 변경, rollback, idempotency 검토
```

### Explorer prompt

```
Explorer 역할로 진행해줘.
회원 탈퇴 흐름과 subscription cancel 흐름을 읽고,
subscription이 active로 남을 수 있는 지점을 찾아줘.

볼 범위:
- src/account/**
- src/billing/**
- tests/account/**
- tests/billing/**

아직 수정하지 마.
산출물은 관련 파일, 현재 흐름, 의심 지점, 최소 수정 후보로 정리해.
```

### Worker prompt

```
Worker 역할로 진행해줘.
Explorer 결과에 따라 회원 탈퇴 시 billing cancel이 누락되는 문제만 수정해.

수정 범위:
- src/account/deleteAccount.ts
- tests/account/deleteAccount.test.ts

금지:
- API response shape 변경 금지
- billing provider wrapper 변경 금지
- DB schema 변경 금지

완료 조건:
- cancel 호출 검증 테스트 추가
- 관련 테스트 통과
```

## 시나리오 2. 신규 기능

### 문제

```
관리자 페이지에 사용자 상태 필터를 추가한다.
```

### Workflow

```
Explorer A:
frontend user table 구조 분석

Explorer B:
backend user search API query parameter 구조 분석

Main:
frontend-only 1단계와 backend 연결 2단계로 분리

Worker A:
필터 UI와 URL query sync 구현

Worker B:
API query parameter 연결

Reviewer:
UX, API contract, 테스트 누락 검토
```

### Orchestration 포인트

frontend와 backend 탐색은 병렬로 가능하다.

하지만 구현은 순서가 필요하다.

```
1단계: UI와 URL 상태
2단계: API query 연결
3단계: 테스트와 리뷰
```

왜냐하면 API contract가 확정되기 전 frontend가 임의의 파라미터를 만들면 나중에 수정 비용이 커지기 때문이다.

## 시나리오 3. 대규모 리팩토링

### 문제

```
billing module의 service/repository 경계가 흐려져 있다.
```

### Workflow

```
Planner:
리팩토링 단계를 ExecPlan으로 작성

Explorer:
현재 dependency graph와 호출 흐름 파악

Human:
목표 구조 승인

Worker 1:
동작 변경 없이 파일 이동과 import 정리

Worker 2:
service/repository 책임 분리

Verifier:
typecheck와 billing test 실행

Reviewer:
public API, transaction boundary, rollback 가능성 검토
```

### 중요한 원칙

대규모 리팩토링은 agent를 병렬로 많이 붙이기보다 단계별 합류 지점이 더 중요하다.

```
1. 동작 변경 없는 이동
2. 책임 분리
3. 테스트 보강
4. dead code 제거
```

각 단계는 독립적으로 리뷰 가능해야 한다.

---

# 8. 실패 패턴

## 실패 1. 모든 agent에게 같은 일을 시킴

```
Agent A: 원인 찾아줘
Agent B: 원인 찾아줘
Agent C: 원인 찾아줘
```

이 방식은 가끔 유용하지만, 산출물이 겹치면 판단 비용이 커진다.

대안:

```
Agent A: frontend 흐름 분석
Agent B: backend 흐름 분석
Agent C: 테스트 실패 로그 분석
```

## 실패 2. 같은 파일을 여러 worker가 수정

문제:

```
- merge conflict
- 서로의 가정 충돌
- 테스트 실패 원인 파악 어려움
```

대안:

```
Worker마다 파일 소유권을 명확히 한다.
같은 파일을 수정해야 하면 병렬이 아니라 순차로 진행한다.
```

## 실패 3. 탐색 agent에게 구현까지 시킴

Explorer가 원인을 찾다가 바로 고치기 시작하면 역할 경계가 무너진다.

대안:

```
Explorer prompt에 "수정 금지"를 명시한다.
수정은 계획 승인 후 Worker가 한다.
```

## 실패 4. 리뷰 agent가 요구사항을 모름

diff만 던지면 reviewer는 “변경 자체”만 본다.

대안:

```
Reviewer에게는 반드시 원래 요구사항, 금지사항, 완료 조건을 함께 준다.
```

## 실패 5. 병렬화할 수 없는 일을 병렬화

나쁜 예:

```
Worker A: API 타입 변경
Worker B: frontend 타입 적용
Worker C: 테스트 수정
```

API 타입이 확정되지 않았는데 frontend와 테스트를 동시에 바꾸면 재작업이 생긴다.

대안:

```
1. API contract 확정
2. frontend 적용
3. 테스트 수정
```

## 실패 6. 최종 통합자가 없음

여러 agent가 좋은 결과를 내도, 최종 판단자가 없으면 작업은 끝나지 않는다.

대안:

```
Main agent 또는 사람이 다음을 판단한다.

- 어떤 분석을 채택할지
- 어떤 diff를 유지할지
- 어떤 finding을 수정할지
- 어떤 테스트 결과를 충분하다고 볼지
```

---

# 9. 운영 체크리스트

## 서브에이전트를 쓰기 전

```
[ ] 이 작업은 단일 agent보다 역할 분리가 유리한가?
[ ] 병렬로 처리할 수 있는 독립 영역이 있는가?
[ ] 각 agent의 산출물이 명확한가?
[ ] 같은 파일을 여러 agent가 수정하지 않는가?
[ ] 최종 판단자는 누구인가?
```

## Context 설계

```
[ ] agent마다 다른 context를 주는가?
[ ] 불필요한 전체 로그를 전달하지 않았는가?
[ ] 수정 범위와 금지 범위를 명시했는가?
[ ] 다음 agent가 사용할 수 있는 형태로 산출물을 요구했는가?
```

## Workflow 설계

```
[ ] 탐색과 구현을 분리했는가?
[ ] 구현과 리뷰를 분리했는가?
[ ] 테스트 실행 지점이 정해져 있는가?
[ ] 실패 시 어느 agent가 다시 처리할지 정했는가?
[ ] 합류 지점마다 판단 기준이 있는가?
```

## Orchestration 설계

```
[ ] 병렬 가능한 작업과 순차 작업을 구분했는가?
[ ] 파일 소유권을 나눴는가?
[ ] shared contract 변경은 먼저 확정했는가?
[ ] 최종 diff를 누가 리뷰하는가?
[ ] 사람의 승인 지점이 필요한가?
```

## 작업 종료 전

```
[ ] 변경 파일이 요구 범위 안에 있는가?
[ ] 관련 테스트를 실행했는가?
[ ] reviewer finding을 처리했는가?
[ ] public API, DB schema, dependency 변경이 의도된 것인가?
[ ] 남은 리스크를 설명할 수 있는가?
```

---

# 10. 결론

## 결론 메시지

서브에이전트 workflow의 핵심은 agent를 많이 쓰는 것이 아니다.

> **핵심은 작업을 agent가 수행 가능한 작은 책임으로 나누고, context와 검증 루프를 설계하는 것이다.**
> 

AI 코딩에서 개발자의 역할은 이렇게 바뀐다.

| 기존 역할 | 확장된 역할 |
| --- | --- |
| 직접 구현자 | agent에게 줄 작업 단위 설계자 |
| 코드 리뷰어 | agent review workflow 운영자 |
| 디버거 | 실패 로그 기반 수정 루프 설계자 |
| 문서 작성자 | AGENTS.md와 작업 계약 설계자 |
| 테크 리드 | multi-agent orchestration 설계자 |

## 핵심 문장

> **서브에이전트는 병렬 처리 도구가 아니라 책임 분리 도구다.**
> 

> **Context engineering은 모든 agent에게 많은 정보를 주는 것이 아니라, 각 agent에게 필요한 정보를 정확히 주는 것이다.**
> 

> **Workflow design은 AI에게 일을 맡기기 전에 일이 끝나는 조건을 설계하는 것이다.**
> 

> **Orchestration은 누가 무엇을 할지뿐 아니라, 누가 무엇을 하지 말아야 하는지 정하는 일이다.**
> 

> **AI coding의 고수는 prompt를 잘 쓰는 사람을 넘어, agent 팀을 잘 운영하는 사람이다.**
> 

---

# 부록: 한 장 요약

```
서브에이전트를 활용한 AI 코딩 워크플로우

1. 큰 작업은 역할로 나눈다.
2. Explorer는 읽고, Worker는 고치고, Reviewer는 의심하고, Verifier는 검증한다.
3. 모든 agent에게 같은 context를 주지 않는다.
4. agent마다 입력 context와 산출물을 명확히 한다.
5. 병렬 가능한 일과 순차 작업을 구분한다.
6. 같은 파일을 여러 worker가 동시에 수정하지 않게 한다.
7. 탐색, 구현, 검증, 리뷰의 합류 지점을 만든다.
8. 최종 판단자는 항상 필요하다.
9. agent 수보다 workflow 품질이 중요하다.
10. 개발자는 coder에서 agent orchestrator로 확장된다.
```

---

# 부록: 실전 prompt 템플릿

## Planner 템플릿

```
이 작업을 서브에이전트 workflow로 나눠줘.

포함할 것:
- 단계
- 담당 agent 역할
- 입력 context
- 산출물
- 수정 가능 파일
- 수정 금지 파일
- 병렬 가능 여부
- 검증 방법

아직 구현하지 마.
```

## Explorer 템플릿

```
Explorer 역할로 진행해줘.

목표:
{문제 또는 기능}의 관련 파일과 현재 흐름을 파악한다.

볼 범위:
- {파일/폴더}

금지:
- 파일 수정 금지
- 테스트 수정 금지

산출물:
- 관련 파일
- 현재 동작
- 의심 지점
- 최소 수정 후보
- 테스트 전략
```

## Worker 템플릿

```
Worker 역할로 진행해줘.

목표:
{승인된 계획}을 최소 변경으로 구현한다.

수정 범위:
- {파일/폴더}

금지:
- public API 변경 금지
- DB schema 변경 금지
- 새 dependency 추가 금지
- 계획에 없는 refactor 금지

완료 조건:
- {테스트 또는 검증 조건}
- 변경 파일과 이유 요약
```

## Reviewer 템플릿

```
Reviewer 역할로 현재 diff를 리뷰해줘.

원래 요구사항:
{요구사항}

금지사항:
{금지사항}

확인할 것:
- 요구 범위 초과 변경
- public contract 변경
- edge case 누락
- 테스트 누락
- 불필요한 refactor

finding을 심각도 순서로 제시해.
```

## Verifier 템플릿

```
Verifier 역할로 진행해줘.

검증할 것:
- {테스트 명령}
- {재현 시나리오}

코드 수정은 하지 마.
실패하면 핵심 로그, 의심 원인, 다음 조치를 정리해.
```
