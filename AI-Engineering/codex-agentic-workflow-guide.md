# Codex Agentic Workflow Guide

### Prompt가 아니라 Workflow를 설계하라

> **핵심 메시지**
> 
> 
> Codex를 잘 쓰는 사람은 프롬프트만 잘 쓰는 사람이 아니다.
> 
> **환경, 권한, context, 검증 workflow를 설계하는 사람**이다.
> 

---

# 1. 왜 Codex 결과는 사람마다 차이가 큰가

## 설명

같은 모델을 써도 결과가 크게 달라진다.

이유는 단순하다.

> **AI coding agent의 결과는 모델 성능보다 “주어진 작업 환경”에 더 크게 영향을 받는다.**
> 

Codex는 단순히 질문에 답하는 모델이 아니라, 다음 정보를 바탕으로 행동한다.

| 요소 | 결과에 주는 영향 |
| --- | --- |
| 현재 working directory | 어떤 repo를 기준으로 판단하는지 |
| 열린 파일 / 지정 파일 | 어떤 코드를 우선 참고하는지 |
| AGENTS.md | 어떤 규칙을 따르는지 |
| approval mode | 어디까지 자동으로 실행 가능한지 |
| sandbox | 어떤 파일/명령/네트워크에 접근 가능한지 |
| 세션 히스토리 | 이전 대화가 현재 판단에 섞이는지 |
| 테스트 명령 | 검증을 어떻게 끝낼지 |
| prompt의 범위 | 얼마나 넓게 수정하려고 드는지 |

즉, 결과 차이는 보통 “프롬프트 문장력” 때문만이 아니다.

대부분은 이런 차이다.

```
A 개발자:
"이거 고쳐줘"

B 개발자:
"src/payment/checkout.ts의 할인 계산 버그를 고쳐줘.
관련 테스트는 tests/payment/checkout.test.ts야.
public API는 바꾸지 말고, 실패하는 케이스를 먼저 재현한 뒤 최소 수정해줘.
완료 조건은 npm test -- checkout 통과야."
```

두 사람은 같은 모델을 썼지만, 사실상 다른 작업 환경을 준 것이다.

## 실전 팁

Codex에게 좋은 결과를 얻으려면 질문을 잘하는 것보다 먼저 아래를 챙긴다.

| 체크 | 질문 |
| --- | --- |
| 작업 범위 | 어느 파일/폴더를 봐야 하는가? |
| 금지 범위 | 어디는 절대 건드리면 안 되는가? |
| 성공 조건 | 언제 끝났다고 볼 것인가? |
| 검증 방법 | 어떤 테스트/빌드/린트를 실행해야 하는가? |
| 위험도 | 자동 수정해도 되는 작업인가, 계획만 봐야 하는가? |

---

# 2. Codex는 단순 ChatGPT가 아니라 Agent다

## 설명

ChatGPT에게 코드를 물어보면 보통 답변을 받는다.

Codex는 다르다.

Codex는 다음을 할 수 있다.

| 기능 | 의미 |
| --- | --- |
| file read | 실제 코드베이스를 탐색한다 |
| file edit | 파일을 직접 수정한다 |
| terminal | 테스트, 빌드, grep, git diff 등을 실행한다 |
| tool use | shell, patch, browser, MCP, GitHub 등 도구를 호출한다 |
| self-review | 본인이 만든 diff를 다시 검토한다 |
| iterative loop | 실패하면 원인을 보고 다시 수정한다 |

Codex를 이해하는 핵심은 이 루프다.

```
생각 -> 실행 -> 관찰 -> 수정
```

조금 더 개발 workflow처럼 쓰면:

```
1. 문제 이해
2. 관련 파일 탐색
3. 수정 계획 작성
4. 작은 단위로 변경
5. 테스트 실행
6. 실패 로그 관찰
7. 재수정
8. diff 리뷰
9. 완료 조건 확인
```

즉 Codex의 본질은 “코드 생성기”보다 “작업 수행 agent”에 가깝다.

## 실전 팁

Codex에게 바로 코드를 쓰게 하지 말고, 먼저 루프를 설계한다.

```
먼저 관련 파일을 찾아서 원인을 설명해줘.
아직 수정하지 마.
수정 계획과 리스크를 정리한 뒤 내가 승인하면 구현해.
```

또는:

```
실패하는 테스트를 먼저 재현해.
그 다음 최소 수정하고 같은 테스트를 다시 실행해.
마지막에 변경된 파일과 검증 결과를 요약해.
```

---

# 3. AGENTS.md 깊게 이해하기

## 설명

`AGENTS.md`는 쉽게 말해 **AI용 README**다.

사람 개발자가 repo에 들어오면 `README.md`, `CONTRIBUTING.md`, architecture doc을 읽는다.

Codex에게는 그 역할을 `AGENTS.md`가 한다.

> **반복해서 prompt에 쓰는 규칙은 AGENTS.md로 올려야 한다.**
> 

예를 들어 매번 이렇게 쓰고 있다면:

```
TypeScript strict 지켜줘.
테스트는 pnpm test로 실행해.
public API는 바꾸지 마.
UI는 기존 디자인 시스템을 따라.
```

이건 prompt가 아니라 repo 규칙이다.

`AGENTS.md`로 빼야 한다.

## 왜 중요한가

Codex는 작업 시작 전 `AGENTS.md`를 읽고, 그 지침을 기본 context로 삼는다.

효과는 크다.

| 문제 | AGENTS.md로 해결되는 방식 |
| --- | --- |
| 매번 같은 설명 반복 | repo 기본 규칙으로 고정 |
| agent가 엉뚱한 테스트 실행 | 검증 명령 명시 |
| 스타일이 흔들림 | 코드 스타일/아키텍처 규칙 제공 |
| 큰 refactor 남발 | 변경 범위 제한 |
| 팀마다 다른 방식 | 팀의 작업 방식을 문서화 |

## global / repo / folder hierarchy

Codex의 instruction은 계층적으로 적용된다.

| 위치 | 용도 |
| --- | --- |
| Global `~/.codex/AGENTS.md` | 개인 공통 작업 원칙 |
| Repo root `AGENTS.md` | 프로젝트 전체 규칙 |
| Subfolder `src/frontend/AGENTS.md` | 특정 영역 규칙 |
| Subfolder `src/backend/AGENTS.md` | 특정 영역 규칙 |

중요한 점:

> **가까운 폴더의 AGENTS.md일수록 더 구체적인 규칙을 담는다.**
> 

예시 구조:

```
my-service/
  AGENTS.md
  frontend/
    AGENTS.md
    src/
  backend/
    AGENTS.md
    src/
  packages/
    billing/
      AGENTS.md
```

## 좋은 규칙 작성법

좋은 `AGENTS.md`는 구체적이고 검증 가능하다.

| 좋은 규칙 | 이유 |
| --- | --- |
| `Run pnpm test -- checkout after modifying checkout logic.` | 어떤 테스트를 돌릴지 명확 |
| `Do not change public API response shapes unless requested.` | 위험한 변경 차단 |
| `Prefer src/components/ui before creating new UI components.` | 기존 패턴 유지 |
| `Keep diffs focused; avoid unrelated refactors.` | agent 폭주 방지 |

## 안 좋은 규칙 예시

```markdown
# Bad AGENTS.md

-Write clean code.
-Make it scalable.
-Follow best practices.
-Improve architecture where needed.
-Be smart.
```

왜 안 좋은가?

| 규칙 | 문제 |
| --- | --- |
| clean code | 사람마다 의미가 다름 |
| scalable | 과한 추상화 유도 |
| best practices | 구체적 행동이 없음 |
| improve architecture | 원치 않는 대규모 refactor 유발 |
| be smart | 아무 의미 없음 |

## 실무 활용 패턴

### 패턴 1. “반복 prompt 제거”

매번 쓰던 말을 `AGENTS.md`로 이동한다.

```
Before:
"기존 패턴 따르고 테스트 돌리고 public API 바꾸지 마."

After:
AGENTS.md에 고정.
prompt에는 이번 task만 작성.
```

### 패턴 2. “팀의 코드 리뷰 기준 문서화”

```markdown
## Review checklist

-Is the diff focused?
-Are edge cases covered?
-Are loading/error states handled?
-Are public contracts preserved?
-Is there a test or clear reason why not?
```

### 패턴 3. “위험 명령 차단”

```markdown
## Safety

-Never run database migrations against production.
-Never print `.env` values.
-Never run `git push --force` without explicit approval.
-Ask before deleting files.
```

### 패턴 4. “긴 작업은 계획 문서와 연결”

```markdown
## Execution plans

For complex features or refactors, create an implementation plan first.

The plan must include:

-Goal
-Relevant files
-Proposed changes
-Risks
-Test strategy
-Rollback considerations

Do not implement until the plan is reviewed.
```

## 왜 반복 프롬프트보다 강력한가

반복 prompt는 세션마다 누락된다.

`AGENTS.md`는 repo에 남는다.

| 반복 prompt | AGENTS.md |
| --- | --- |
| 매번 사람이 기억해야 함 | repo가 기억함 |
| 세션이 바뀌면 사라짐 | 지속됨 |
| 개인 습관에 의존 | 팀 규칙으로 공유 가능 |
| 길어질수록 귀찮음 | 기본 context로 자동 주입 |
| 누락 시 결과 흔들림 | 결과 일관성 증가 |

---

# 4. Slash Commands

## 설명

Slash command는 Codex 세션을 제어하는 운영 명령어다.

프롬프트가 “작업 지시”라면, slash command는 “agent 운영 제어”에 가깝다.

> **고수들은 Codex에게 말만 잘 거는 게 아니라, 세션 상태를 적극적으로 조작한다.**
> 

## 주요 Slash Commands

| Command | 언제 쓰나 | 실전 의미 |
| --- | --- | --- |
| `/plan` | 구현 전 계획이 필요할 때 | 바로 코딩하지 않고 탐색/계획 모드로 전환 |
| `/review` | 변경 후 diff 점검 | Codex에게 별도 리뷰어 역할을 시킴 |
| `/clear` | 맥락을 완전히 비우고 싶을 때 | 오염된 context 제거 |
| `/compact` | 긴 세션을 요약 유지하고 싶을 때 | 핵심만 남기고 token 절약 |
| `/model` | 작업 난이도에 따라 모델 변경 | 빠른 모델/깊은 모델 전환 |
| `/fast` | 빠른 응답 tier 사용 | 단순 반복 작업이나 빠른 탐색에 유리 |

## `/plan`

### 언제 쓰는가

- 작업이 크거나 애매할 때
- architecture 영향이 있을 때
- 여러 파일을 건드릴 가능성이 있을 때
- 먼저 방향을 검토하고 싶을 때

### 예시

```
/plan

회원 등급별 할인 정책을 추가하려고 해.
관련 도메인은 billing, checkout, subscription이야.
먼저 현재 구조를 탐색하고 구현 계획만 제안해줘.
아직 수정하지 마.
```

### 왜 중요한가

Codex의 가장 큰 실패는 “확신에 찬 잘못된 구현”이다.

`/plan`은 그 전에 멈추게 한다.

### 실전 팁

```
계획에는 반드시 다음을 포함해줘.

- 수정 대상 파일
- 변경 이유
- 대안
- 리스크
- 테스트 전략
- 구현 순서
```

## `/review`

### 언제 쓰는가

- Codex가 수정한 직후
- 내가 직접 수정한 diff를 점검받을 때
- PR 올리기 전
- regression 위험이 있을 때

### 예시

```
/review

현재 working tree를 리뷰해줘.
특히 다음을 봐줘.

- 의도하지 않은 public API 변경
- 테스트 누락
- 에러 처리 누락
- 불필요한 refactor
```

### 왜 중요한가

Codex는 구현자 역할과 리뷰어 역할을 분리해서 쓸 때 더 좋다.

```
Agent 1: 구현
Agent 2: 리뷰
Human: 최종 판단
```

## `/clear`

### 언제 쓰는가

- 이전 대화가 현재 작업에 방해될 때
- 전혀 다른 task로 넘어갈 때
- agent가 이전 가정을 계속 끌고 올 때
- context가 오염되었다고 느껴질 때

### 실전 의미

`/clear`는 “머리 비우기”다.

긴 세션에서 이런 현상이 생기면 clear를 고려한다.

```
- 이미 끝난 요구사항을 계속 고려함
- 예전 파일 구조를 기준으로 말함
- 방금 취소한 방향으로 다시 구현함
- 불필요하게 이전 논의를 반복함
```

## `/compact`

### 언제 쓰는가

- 세션은 이어가고 싶은데 대화가 너무 길 때
- 긴 디버깅 후 핵심만 남기고 싶을 때
- context window를 아끼고 싶을 때
- 지금까지의 결정사항은 유지하고 싶을 때

### `/clear`와 차이

| Command | 의미 |
| --- | --- |
| `/clear` | 대화 context를 새로 시작 |
| `/compact` | 지금까지의 핵심을 요약해서 유지 |

### 실전 팁

`/compact` 전에 이렇게 요청하면 좋다.

```
지금까지 결정된 내용, 수정한 파일, 남은 작업, 검증 결과를 중심으로 compact에 적합한 요약을 만들어줘.
```

## `/model`

### 언제 쓰는가

- 간단한 반복 수정은 빠른 모델
- 복잡한 디버깅/설계는 깊은 reasoning 모델
- 비용/속도/정확도 trade-off를 조절하고 싶을 때

### 실전 팁

| 작업 | 추천 방향 |
| --- | --- |
| 단순 rename, import 정리 | 빠른 모델 |
| 테스트 실패 원인 분석 | reasoning 강한 모델 |
| architecture 설계 | reasoning 강한 모델 |
| 대량 반복 변경 | 빠른 모델 + 좁은 범위 |
| 보안/권한 로직 | reasoning 강한 모델 + manual approval |

## `/fast`

### 언제 쓰는가

- 빠른 피드백이 중요한 작업
- 사소한 수정
- 반복적인 코드 정리
- 초벌 탐색

### 주의

빠른 것이 항상 좋은 것은 아니다.

```
빠른 모델/빠른 tier:
- 탐색
- 단순 수정
- 반복 작업

깊은 reasoning:
- 설계
- 복잡한 버그
- 보안
- 데이터 정합성
```

---

# 5. Approval / Sandbox

## 설명

Codex는 권한 있는 agent다.

그냥 텍스트 답변만 하는 것이 아니라:

```
- 파일을 읽고
- 파일을 수정하고
- shell command를 실행하고
- 테스트를 돌리고
- dependency를 설치하려 하고
- 경우에 따라 network 접근을 요청한다
```

그래서 중요한 질문은 이것이다.

> **Codex에게 어디까지 맡길 것인가?**
> 

## Approval mode 개념

버전과 실행 환경에 따라 이름은 조금씩 다를 수 있지만, 실무적으로는 아래처럼 이해하면 된다.

| 모드 | 의미 | 적합한 상황 |
| --- | --- | --- |
| manual / read-only / suggest | 읽기 중심, 수정/실행 전 승인 | 코드 리뷰, 분석, 학습 |
| ask / on-request | 기본 작업은 진행, 위험 작업은 승인 요청 | 일반 개발 작업 |
| auto | 읽기/쓰기/명령을 더 자율적으로 수행 | 테스트 루프, 반복 수정 |
| full auto 계열 | sandbox 안에서 최대한 자동 수행 | 긴 작업, 프로토타입, 낮은 위험 작업 |

## Sandbox란?

Sandbox는 Codex가 실행하는 command와 파일 접근을 제한하는 경계다.

| 제어 대상 | 예시 |
| --- | --- |
| file read | 현재 workspace 외부 읽기 제한 |
| file write | repo 밖 파일 수정 제한 |
| shell command | 명령 실행 범위 제한 |
| network | package install, API 호출 제한 |
| destructive action | 삭제, reset, migration 등 승인 요구 |

중요한 구분:

| 개념 | 역할 |
| --- | --- |
| sandbox | 기술적으로 어디까지 가능한가 |
| approval | 언제 사람에게 물어봐야 하는가 |

## 왜 위험한가

Codex는 선의로 위험한 일을 할 수 있다.

예:

```
- 테스트 고치려고 production-like config 수정
- dependency 충돌 해결하려고 package lock 대규모 변경
- migration 파일 생성
- generated file 직접 수정
- git clean, reset, rm 실행
- .env 내용 출력
- 네트워크로 외부 패키지 설치
```

이런 일은 “AI가 나빠서”가 아니라, agent가 목표를 달성하려고 행동하기 때문에 생긴다.

## 실전 운영 기준

| 상황 | 추천 권한 |
| --- | --- |
| 처음 보는 repo 분석 | read-only/manual |
| 작은 버그 수정 | ask/on-request |
| 테스트 실패 자동 반복 | workspace-write + 제한된 command |
| dependency 설치 | approval 필요 |
| DB migration | manual approval |
| production 관련 작업 | read-only에서 시작 |
| 대규모 refactor | plan 먼저, approval 엄격하게 |

## AGENTS.md에 넣을 안전 규칙

```markdown
## 안전 규칙 (Safety Rules)

-명시적인 승인 없이 파괴적인 git 명령어를 실행하지 마세요. (예: 강제 푸시, 브랜치 삭제 등)
-비밀번호, API 키 등 보안 정보(Secrets)나 환경 변수를 출력하지 마세요.
-리포지토리(Repository) 외부의 파일을 수정하지 마세요.
-새로운 의존성(Dependencies/라이브러리)을 설치하기 전에 먼저 확인을 받으세요.
-데이터베이스 마이그레이션을 생성하거나 수정하기 전에 먼저 확인을 받으세요.
-명시적인 요청이 없는 한, CI/CD 설정을 변경하지 마세요.
```

---

# 6. Context Engineering

## 설명

Prompt Engineering은 “어떻게 말할까”에 가깝다.

Context Engineering은 “무엇을 보게 할까”에 가깝다.

> **Codex에서는 무엇을 수정할지보다, 어디를 봐야 하는지 알려주는 게 더 중요하다.**
> 

## Prompt Engineering vs Context Engineering

| 구분 | Prompt Engineering | Context Engineering |
| --- | --- | --- |
| 초점 | 문장, 지시 방식 | 정보, 범위, 작업 환경 |
| 질문 | 어떻게 말할까? | 무엇을 보게 할까? |
| 예시 | “단계별로 생각해줘” | “이 파일과 이 테스트만 봐줘” |
| 위험 | 말은 좋은데 근거 부족 | 잘 설계하면 결과 안정 |
| Codex에서 중요도 | 중요 | 더 중요 |

## Context pollution

Context pollution은 불필요하거나 오래된 정보가 agent 판단을 흐리는 현상이다.

예:

```
- 이미 폐기한 설계 논의
- 이전 버그의 로그
- 관련 없는 파일 내용
- 너무 긴 에러 출력
- 과거 요구사항
- "아까 말한 것처럼"의 누적
```

증상:

```
- Codex가 이전 요구사항을 계속 반영함
- 수정 범위가 점점 커짐
- 관련 없는 파일을 고침
- 대화가 길어질수록 답변이 흐려짐
- 이미 해결된 문제를 다시 설명함
```

## 범위 제한이 중요한 이유

큰 repo에서 실패하는 이유는 모델이 멍청해서가 아니다.

대부분은 search space가 너무 넓다.

```
나쁜 요청:
결제 버그 고쳐줘.

좋은 요청:
결제 완료 후 coupon이 중복 적용되는 문제야.
먼저 아래 파일만 봐줘.

- src/domain/checkout/applyCoupon.ts
- src/domain/checkout/calculateTotal.ts
- tests/checkout/coupon.test.ts

수정은 checkout domain 내부로 제한해.
```

## 파일 지정의 힘

Codex에게 파일을 지정하면 다음이 달라진다.

| 파일 미지정 | 파일 지정 |
| --- | --- |
| repo 전체 탐색 | 관련 영역부터 탐색 |
| 추측 증가 | 근거 기반 분석 |
| 수정 범위 커짐 | diff 작아짐 |
| 시간/토큰 증가 | 빠르고 안정적 |
| architecture 오판 가능 | 기존 패턴 유지 가능 |

## 작은 task 분리

큰 작업을 한 번에 맡기면 실패 확률이 올라간다.

```
나쁜 작업:
관리자 페이지 전체 개편해줘.

좋은 분리:
1. 현재 관리자 페이지 구조 분석
2. navigation 구조만 개선 계획
3. sidebar 컴포넌트만 수정
4. user table 필터만 수정
5. 테스트/시각 검증
6. review
```

## session reset 전략

| 상황 | 선택 |
| --- | --- |
| 완전히 다른 작업 시작 | `/clear` |
| 같은 작업인데 세션이 길어짐 | `/compact` |
| 이전 논의가 방해됨 | `/clear` |
| 결정사항만 유지하고 싶음 | `/compact` |
| 실험적 방향을 분리하고 싶음 | `/fork` 또는 새 세션 |

## Noise 제거

Codex에게 주지 말아야 할 것:

```
- 전체 로그 5,000줄
- 관련 없는 stack trace
- 오래된 요구사항
- "혹시 몰라서" 붙인 대량 파일
- 이미 버린 설계안
- unrelated diff
```

대신 이렇게 준다.

```
아래는 실패 로그의 핵심 부분이야.

Expected: 10000
Received: 9000

관련 테스트:
tests/checkout/discount.test.ts

관련 함수:
src/checkout/calculateDiscount.ts
```

## Why large repo fails

대형 repo에서 Codex가 흔들리는 전형적인 이유:

| 원인 | 결과 |
| --- | --- |
| 관련 파일을 못 찾음 | 잘못된 레이어 수정 |
| 비슷한 패턴이 많음 | 오래된 패턴 복사 |
| 테스트 명령이 불명확 | 검증 실패 |
| generated file 존재 | 생성물 직접 수정 |
| domain rule이 문서화 안 됨 | 비즈니스 로직 오해 |
| context가 너무 큼 | 중요한 정보 희석 |

---

# 7. 실전 Prompt 패턴

## 기본 구조

좋은 Codex prompt는 보통 이 네 가지를 포함한다.

```
Goal:
Context:
Constraints:
Done when:
```

예시:

```
Goal:
회원 탈퇴 후에도 billing subscription이 active로 남는 버그를 고쳐줘.

Context:
관련 파일은 다음이야.
- src/account/deleteAccount.ts
- src/billing/cancelSubscription.ts
- tests/account/deleteAccount.test.ts

Constraints:
- public API response shape는 바꾸지 마.
- billing provider SDK wrapper는 수정하지 마.
- 최소 변경으로 해결해.

Done when:
- deleteAccount 테스트에 subscription cancel 검증이 추가됨
- pnpm test -- deleteAccount 통과
- 변경 파일과 검증 결과 요약
```

## 나쁜 prompt vs 좋은 prompt

| 나쁜 prompt | 문제 |
| --- | --- |
| `이거 고쳐줘` | 이게 뭔지 모름 |
| `전체적으로 개선해줘` | 범위 무한대 |
| `깔끔하게 리팩토링해줘` | 기준 없음 |
| `테스트도 알아서` | 어떤 테스트인지 모름 |
| `성능 좋게 해줘` | 측정 기준 없음 |

| 좋은 prompt | 좋은 이유 |
| --- | --- |
| `src/auth/session.ts의 만료 계산만 수정해줘` | 범위 명확 |
| `public API는 바꾸지 마` | 위험 차단 |
| `실패 테스트 먼저 재현해` | 검증 루프 시작 |
| `수정 전 계획만 제안해` | 폭주 방지 |
| `완료 조건은 pnpm test -- auth 통과` | 종료 기준 명확 |

## 수정 범위 제한

```
수정 범위는 아래로 제한해.

- src/features/checkout/**
- tests/checkout/**

다음은 수정하지 마.

- prisma/**
- src/api/public/**
- package.json
- pnpm-lock.yaml
```

## 금지 영역 지정

```
주의:
- DB schema는 바꾸지 마.
- migration 생성하지 마.
- 새 dependency 추가하지 마.
- 공통 Button 컴포넌트는 수정하지 마.
- generated 파일은 직접 수정하지 마.
```

## 성공 조건 정의

```
완료 조건:
- 기존 실패 테스트가 통과해야 함
- 신규 테스트 1개 이상 추가
- pnpm lint 통과
- 변경 파일 목록과 이유 설명
- 남은 리스크가 있으면 명시
```

## 분석 먼저 시키기

```
먼저 원인 분석만 해줘.
아직 파일 수정하지 마.

분석 결과에는 다음을 포함해줘.
- 관련 파일
- 현재 동작
- 기대 동작
- 버그 원인
- 최소 수정 지점
- 테스트 전략
```

## 테스트 먼저 작성시키기

```
TDD 방식으로 진행해줘.

1. 현재 버그를 재현하는 실패 테스트를 먼저 추가
2. 테스트 실패를 확인
3. 최소 수정
4. 테스트 통과 확인
5. diff 요약
```

## self-review 시키기

```
수정 후 스스로 리뷰해줘.

리뷰 기준:
- 변경 범위가 요청과 일치하는가
- public API가 바뀌지 않았는가
- edge case가 빠지지 않았는가
- 테스트가 충분한가
- 불필요한 refactor가 있는가
```

---

# 8. 실전 Workflow: 고수들은 어떻게 쓰는가

## 기본 workflow

```
1. /clear
2. 문제를 작은 단위로 정의
3. /plan
4. 관련 파일 탐색
5. 구현 범위 합의
6. 작은 diff로 수정
7. 테스트 실행
8. 실패 시 로그 기반 재수정
9. /review
10. 최종 diff 확인
11. commit 또는 PR
```

## 예시 workflow: 버그 수정

```
/clear
```

```
/plan

checkout에서 쿠폰이 중복 적용되는 버그를 고치려고 해.
관련 파일은 checkout domain과 coupon test야.
먼저 원인 분석과 수정 계획만 제안해줘.
```

계획 확인 후:

```
계획대로 진행해.
수정 범위는 src/checkout과 tests/checkout으로 제한해.
먼저 실패 테스트를 추가하고, 그 다음 최소 수정해.
```

수정 후:

```
관련 테스트를 실행해.
실패하면 로그를 보고 수정해.
```

마지막:

```
/review

현재 diff를 리뷰해줘.
특히 불필요한 refactor, edge case 누락, public contract 변경 여부를 봐줘.
```

## 예시 workflow: 신규 기능

```
/clear
```

```
/plan

관리자 페이지에 사용자 상태 필터를 추가하려고 해.
먼저 현재 user list 구조를 분석하고 구현 계획을 작성해줘.

조건:
- API response shape는 바꾸지 않음
- 기존 table 컴포넌트 재사용
- URL query param으로 필터 상태 유지
- 테스트 전략 포함
```

계획 승인 후:

```
1단계만 구현해.
필터 UI와 URL query sync만 추가하고 API 변경은 하지 마.
```

그 다음:

```
2단계로 API query parameter 연결을 구현해.
```

마지막:

```
관련 unit/component/e2e 테스트 중 가장 적절한 검증을 실행해.
```

## 예시 workflow: 대규모 refactor

대규모 refactor는 바로 구현시키지 않는다.

```
/plan

이 refactor는 바로 구현하지 말고,
먼저 ExecPlan 형태로 작성해줘.

포함할 것:
- 현재 구조
- 문제점
- 목표 구조
- 단계별 migration plan
- 각 단계의 검증 방법
- rollback 전략
- public API 영향 여부
```

그 다음 작은 단계만 실행한다.

```
ExecPlan의 1단계만 구현해.
동작 변경 없이 파일 이동과 import 정리만 해.
테스트는 typecheck와 관련 unit test만 실행해.
```

## 고수 workflow의 특징

| 초보 | 고수 |
| --- | --- |
| 한 번에 크게 시킴 | 작게 나눔 |
| 바로 구현 요청 | 먼저 분석/계획 |
| repo 전체 맡김 | 파일/폴더 지정 |
| 테스트는 나중 | 테스트를 workflow에 포함 |
| 결과만 봄 | diff와 검증 로그 확인 |
| 세션 계속 이어감 | 필요하면 clear/compact |
| prompt 반복 | AGENTS.md로 시스템화 |
| 권한 무제한 | approval/sandbox 설계 |

---

# 9. 잘 망하는 사례

## 사례 1. 한 번에 너무 큰 작업

```
관리자 페이지 전체를 리뉴얼해줘.
```

결과:

```
- 파일 수십 개 수정
- 기존 패턴 무시
- 테스트 깨짐
- 리뷰 불가능
- 되돌리기도 어려움
```

대안:

```
관리자 페이지의 user table 필터 UI만 먼저 개선해줘.
수정 범위는 src/admin/users 하위로 제한해.
```

## 사례 2. vague prompt

```
성능 개선해줘.
```

문제:

```
- 어떤 성능인지 모름
- 측정 기준 없음
- 과한 caching 추가 가능
- architecture 변경 가능
```

대안:

```
user search API의 p95 latency가 느려.
src/api/users/search.ts에서 N+1 query가 있는지 확인해줘.
먼저 분석만 하고 수정 계획을 제안해.
```

## 사례 3. context overflow

긴 대화를 계속 이어가며 unrelated task를 계속 시킴.

증상:

```
- 이전 작업의 제약이 섞임
- 응답이 길고 흐려짐
- 이상한 파일을 참조함
- 이미 끝난 논의를 반복함
```

대안:

```
- 작업 단위가 바뀌면 /clear
- 같은 작업이 길어지면 /compact
- 핵심 결정사항은 별도 md에 기록
```

## 사례 4. repo 전체 수정

```
전체 코드 스타일 정리해줘.
```

위험:

```
- formatter 대량 변경
- unrelated diff 폭증
- merge conflict 증가
- 실제 변경 의도 파악 불가
```

대안:

```
src/features/billing 하위에서 lint error만 수정해줘.
동작 변경은 하지 마.
```

## 사례 5. architecture 변경을 너무 쉽게 허용

```
구조가 별로면 개선해줘.
```

위험:

```
- service/repository/controller 경계 붕괴
- 새 abstraction 남발
- 기존 convention 무시
- 팀 합의 없는 구조 변경
```

대안:

```
구조 변경이 필요하다고 판단되면 먼저 제안만 해.
내 승인 없이 architecture 변경은 하지 마.
```

## 사례 6. 검증 없이 merge

Codex가 “수정 완료”라고 했다고 merge하면 안 된다.

최소 확인:

```
- git diff 확인
- 관련 테스트 통과
- lint/typecheck 확인
- public API 변경 여부 확인
- 민감 파일 변경 여부 확인
```

## 사례 7. AGENTS.md가 너무 추상적임

```markdown
-Follow best practices.
-Make code clean.
-Be professional.
```

이런 규칙은 agent에게 거의 도움이 안 된다.

대안:

```markdown
-Do not add dependencies without approval.
-Use `src/components/ui` before creating new components.
-Run `pnpm test -- checkout` after changing checkout logic.
-Keep public API response shapes unchanged unless explicitly requested.
```

---

# 10. 핵심 결론

## 결론 메시지

Codex를 잘 쓰는 핵심은 prompt 문장력이 아니다.

> **Prompt Engineering보다 Environment Engineering이 중요해지고 있다.**
> 

앞으로 개발자의 생산성은 이런 역량에서 갈린다.

| 기존 역량 | AI agent 시대의 확장 |
| --- | --- |
| 코드를 직접 구현 | agent가 구현 가능한 문제로 쪼개기 |
| 라이브러리 사용법 암기 | agent가 볼 context 지정 |
| 디버깅 | 실패 로그 기반 수정 루프 설계 |
| 코드 리뷰 | agent review + human review 운영 |
| 문서 작성 | AGENTS.md, plan, workflow 문서화 |
| 개발 속도 | 검증 가능한 자동화 루프 설계 |

---

# 한 장 요약

```
Codex를 잘 쓰는 법

1. AGENTS.md로 repo 규칙을 고정한다.
2. /plan으로 바로 구현을 막고 먼저 생각하게 한다.
3. 작업 범위를 파일/폴더 단위로 제한한다.
4. approval/sandbox로 권한을 설계한다.
5. 큰 작업은 작은 task로 나눈다.
6. /compact와 /clear로 context를 관리한다.
7. 성공 조건과 테스트 명령을 명확히 준다.
8. 구현 agent와 review agent 역할을 분리한다.
9. 검증 없는 merge는 하지 않는다.
10. 반복되는 좋은 prompt는 workflow와 문서로 승격한다.
```

---

# 참고 자료

- OpenAI Codex CLI 문서: https://developers.openai.com/codex/cli
- Codex Slash Commands: https://developers.openai.com/codex/cli/slash-commands
- AGENTS.md 가이드: https://developers.openai.com/codex/guides/agents-md
- Codex Sandbox 개념: https://developers.openai.com/codex/concepts/sandboxing
- Agent approvals & security: https://developers.openai.com/codex/agent-approvals-security
- Codex Best Practices: https://developers.openai.com/codex/learn/best-practices
