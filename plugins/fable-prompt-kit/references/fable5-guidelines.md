# Fable 5 프롬프팅 가이드라인 — 스킬용 압축 레퍼런스

> 원천: Anthropic 공식 문서 (prompting-claude-fable-5, migration-guide). 웹사이트
> `web/content/guide/*.mdx`와 동일 원천(R-1 조사, 지침 20건). 내용 갱신 시 양쪽 함께 커밋.
> 형식: 카테고리 6종 × 각 지침의 [규칙 / 판별 기준 / 수정 패턴].

> **적용 대상·실행 모델**: 이 레퍼런스는 "**Fable 5로 보낼 프롬프트**"를 다듬기 위한 것이며,
> 스킬을 **실행하는 세션의 모델(Opus·Sonnet 등)과는 무관**하다 — Fable에 보내기 전 준비
> 단계에서 참조한다. 지침은 두 부류다. **Fable 전용**(대상이 Fable 5가 아니면 적용하지
> 않음): §1.4 de-prescribe, §3.1 thinking·샘플링 파라미터, §3.2 effort, §4.1 긴 턴 전제,
> §6 refusal 대응. **전 모델 공통**(대상 모델 무관): §1.1~1.3 명확성, §2 구조화, §5 출력
> 형식. 대상이 Opus·Sonnet이면 전용 항목은 건너뛰고 공통 항목만 적용한다.

## 1. clarity — 지시 명확성

### 1.1 목표·제약·성공 기준으로 말하라
- **규칙**: 실행 순서(step-by-step)를 나열하지 말고 원하는 결과·지켜야 할 제약·완료 판단
  기준을 서술한다. Fable 5는 계획을 스스로 세울 때 더 좋은 결과를 낸다.
- **판별**: 프롬프트에 "먼저/그다음/마지막으로" 같은 순서 지시가 있는가? 성공 기준이
  없는가?
- **수정**: 단계 목록 → `목표: … / 제약: … / 성공 기준: …` 구조로 재편. 순서가 정말
  중요한 경우(마이그레이션 등)만 예외로 유지.

### 1.2 행동의 경계를 명시하라 (State the boundaries)
- **규칙**: 문제 서술·질문·생각 정리와 변경 요청을 구분하게 한다. 진단만 원하면 "수정은
  요청 시에만"을 명시. 상태를 바꾸는 액션 전 근거 확인을 요구.
- **판별**: 모델이 요청 밖 행동(자발적 수정, 방어적 백업)을 할 여지가 있는가?
- **수정 패턴** (공식 스니펫): "When the user is describing a problem, asking a
  question, or thinking out loud rather than requesting a change, the deliverable
  is your assessment. Report your findings and stop. Don't apply a fix until they
  ask for one."

### 1.3 요청의 이유를 함께 줘라 (Give the reason)
- **규칙**: "왜 이걸 요청하는지"(더 큰 작업, 대상, 결과물의 용도)를 알려주면 모델이
  의도를 다른 정보와 연결해 더 나은 판단을 내린다.
- **판별**: 요청이 맥락 없이 동작만 지시하는가? 산출물의 사용자·용도가 없는가?
- **수정 패턴** (공식 템플릿): "I'm working on [the larger task] for [who it's for].
  They need [what the output enables]. With that in mind: [request]."

### 1.4 이전 모델용 지시를 걷어내라 (De-prescribe)
- **규칙**: Opus 등 이전 모델용 과잉 지시(세밀한 체크리스트, 검증 리마인더, 페르소나
  과장)는 Fable 5에서 오히려 품질을 떨어뜨릴 수 있다. 기본 성능으로 충분한 지시는 삭제.
- **판별**: "반드시 테스트 돌려 확인해", "너는 세계 최고의 ~야", 모든 엣지 케이스 나열이
  있는가?
- **수정**: 삭제 또는 원칙 한 문장으로 압축. 프로젝트 고유 컨텍스트(스택·컨벤션·금지
  사항)는 유지.

## 2. structure — 구조화

### 2.1 전달 채널을 구조화하라 (send_to_user)
- **규칙**: 장기 실행 에이전트에서 사용자가 원문 그대로 봐야 할 내용(부분 산출물, 직접
  답변)은 send_to_user류 전용 도구로 분리한다. 도구 정의 + 시스템 프롬프트 사용 지시가
  세트.
- **판별**: 긴 자율 작업인데 중간 전달 수단이 없는가? 도구만 있고 사용 지시가 없는가?
- **수정 패턴** (공식 지시문): "Between tool calls, when you have content the user
  must read verbatim (a partial deliverable, a direct answer to their question),
  call the send_to_user tool with that content. Use send_to_user only for
  user-facing content, not for narration or reasoning."

### 2.2 프롬프트를 스펙 문서처럼 조직하라 (일반 원칙)
- **규칙**: 섹션 분리(개요/제약/요구사항/성공 기준), 예시는 지시 근처 배치, 핵심 제약은
  EXACT 등으로 강조. 전 모델 공통 best practice이며 Fable 5에서도 유효.
- **판별**: 요구사항이 한 문단에 뒤섞여 있는가? 예시와 지시가 분리되어 있는가?
- **수정**: 마크다운 헤딩/목록으로 재조직. 해석 여지가 큰 항목에 반례("~의 정반대")를
  병기.

## 3. thinking-effort — 사고 제어

### 3.1 thinking·샘플링 파라미터를 생략하라
- **규칙**: Fable 5는 adaptive thinking 상시 활성 — `thinking: disabled`·
  `budget_tokens` 모두 400 오류. **샘플링 파라미터(`temperature`/`top_p`/`top_k`)도
  전송 시 400 오류**이며, **assistant prefill은 미지원**(4.6+ 패밀리 공통)이다. 사고
  깊이는 `output_config.effort`로만 조절. `max_tokens`는 thinking+응답 합산
  리밋이므로 이전 모델 값 그대로 쓰면 응답이 잘릴 수 있다 — 재검토.
- **판별**: API 예시/프롬프트에 thinking 파라미터·사고 예산 지시·샘플링 파라미터·prefill이
  있는가? "생각하지 말고 바로 답해" 같은 무효 지시가 있는가?
- **수정**: 해당 파라미터 제거, 사고 깊이는 effort로 대체. "간단히 답해"는 유지
  가능(출력 형식 지시로서).

### 3.2 effort는 high에서 시작해 아래로
- **규칙**: 기본값 high가 대부분 작업에 적합. 루틴 작업은 medium/low(이전 모델 xhigh급
  성능), xhigh는 최난이도 전용, max는 한계 난이도. 일상 작업에서 high의 과잉 정리·맥락
  수집은 범위 제한 스니펫으로 억제.
- **판별**: 루틴 작업에 xhigh/max를 지정했는가? 과잉 리팩터링 억제 문구가 필요한가?
- **수정 패턴** (공식 범위 제한 스니펫 요지): "Don't add features, refactor, or
  introduce abstractions beyond what the task requires. … Do the simplest thing
  that works well."

## 4. agentic — 에이전트·장기실행

### 4.1 긴 턴 전제로 설계하라
- **규칙**: 단일 턴 수 분~수 시간. 타임아웃 상향, 스트리밍, 진행 표시, 비동기 확인 구조.
- **판별**: 프롬프트/하네스가 빠른 응답을 전제하는가?
- **수정 패턴** (과잉 계획 억제): "When you have enough information to act, act.
  Do not re-derive facts already established in the conversation…"

### 4.2 체크포인트는 짧은 지시로
- **수정 패턴**: "Pause for the user only when the work genuinely requires them: a
  destructive or irreversible action, a real scope change, or input that only they
  can provide."

### 4.3 진행 보고 접지
- **규칙**: 보고 전 도구 결과 대조 감사 지시 — 허위 진행 보고를 거의 제거(공식 테스트).
- **수정 패턴**: "Before reporting progress, audit each claim against a tool result
  from this session. Only report work you can point to evidence for…"

### 4.4 병렬 서브에이전트 활용
- **수정 패턴**: "Delegate independent subtasks to subagents and keep working while
  they run. Intervene if a subagent goes off track or is missing relevant context."

### 4.5 메모리 표면 제공
- **수정 패턴**: "Store one lesson per file with a one-line summary at the top. …
  update an existing note rather than creating a duplicate; delete notes that turn
  out to be wrong."

### 4.6 조기 중단 방지 (자율 파이프라인)
- **수정 패턴**: "You are operating autonomously. … Before ending your turn, check
  your last paragraph. If it is a plan, an analysis, a question, a list of next
  steps, or a promise about work you have not done, do that work now with tool
  calls."

### 4.7 컨텍스트 카운트다운을 노출하지 마라
- **수정 패턴** (부득이할 때): "You have ample context remaining. Do not stop,
  summarize, or suggest a new session on account of context limits."

### 4.8 자기 검증을 명시하라
- **수정 패턴**: "Establish a method for checking your own work at an interval of
  [X] as you build. Run this every [X interval], verifying your work with
  subagents against the specification."

### 4.9 난이도 상한선에서 시작하라
- **규칙**: 이전 모델 기준으로 잘게 쪼개지 말고, 가장 어려운 미해결 문제를 통째로 맡긴다.

## 5. output-format — 출력 형식

### 5.1 결론부터, 압축이 아니라 선택으로
- **규칙**: prefill 미지원 — 형식은 지시로 조종. 첫 문장이 TLDR에 답하게 하고, 짧게
  쓰는 방법은 fragment 압축이 아니라 내용 선택.
- **판별**: 산출물이 장황한가? 화살표 체인·약어 압축을 요구하고 있는가?
- **수정 패턴**: "Lead with the outcome. … The way to keep output short is to be
  selective about what you include, not to compress the writing into fragments,
  abbreviations, arrow chains like A → B → fails, or jargon."

### 5.2 최종 요약은 재정향으로
- **규칙**: 긴 에이전트 세션의 최종 요약은 작업 스레드의 연속이 아니라
  재정향(re-grounding) — 작업 중 만든 어휘·라벨을 버리고 완전한 문장으로.
- **수정 패턴**: "When you write the summary at the end, drop the working shorthand.
  Write complete sentences. Spell out terms. … If you have to choose between short
  and clear, choose clear."

## 6. safety-refusal — 안전·refusal 대응

### 6.1 refusal은 200 응답이다
- **규칙**: 분류기 거절 시 HTTP 200 + `stop_reason: "refusal"` +
  `stop_details.category`(cyber/bio/reasoning_extraction 등). fallback(서버사이드
  베타 또는 SDK)으로 자동 재시도 가능. 95%+ 세션에서 미발동.
- **판별**: 보안·생명과학 인접 주제인데 의도(방어 목적, 승인 맥락)가 없는가?
- **수정**: 의도 명시 문단 추가 — 방어/연구/승인된 환경임을 한 문단으로. 오탐 반복 시
  fallback 구성 안내.

### 6.2 추론 재현을 지시하지 마라
- **규칙**: 내부 추론을 에코·전사·설명하게 하는 지시는 reasoning_extraction refusal을
  유발한다. reflection류 기존 스킬 감사 대상.
- **판별**: "생각한 과정을 그대로 적어줘", "reasoning을 보여줘" 류 문구가 있는가?
- **수정**: 삭제 후 대체 — `thinking.display: "summarized"`(API) 또는 "핵심 판단 근거
  N가지 요약" 요청으로 전환.
