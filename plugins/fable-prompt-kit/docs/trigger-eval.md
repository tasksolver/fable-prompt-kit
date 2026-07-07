# 스킬 트리거(description) 정량 평가 — v0.3.0

> 대상: 스킬 6종(prompt-enhance, guideline-check, deep-interview, case-ingest,
> fable-migrate, tip-ingest)의 SKILL.md `description`.
> 방법: 실사용자스러운 테스트 쿼리 30개를 설계하고, **6개 description만 보고**(본문
> 미참조) 각 쿼리의 라우팅을 판정. 오발동·미발동·모호 케이스를 찾아 description을
> 수정한 뒤 재판정. Before = v0.2.0 description 5종 + tip-ingest 초안(v1),
> After = v0.3.0에 반영된 최종 description.

## 판정 기준

- **정발동(✓)**: 기대 스킬이 유일하게 발동 / 네거티브 쿼리에서 아무 스킬도 발동 안 함
- **모호(△)**: 두 스킬이 대등하게 발동 가능 — 라우팅이 운에 좌우됨
- **오발동(✗)**: 기대와 다른 스킬이 발동하거나, 네거티브 쿼리에 스킬이 발동
- **미발동(✗)**: 기대 스킬의 description에 쿼리를 잡을 근거가 없음

## 매트릭스 (쿼리 30개)

### 정발동 기대 쿼리 (스킬당 3개)

| # | 쿼리 | 기대 | Before | After |
|---|---|---|---|---|
| Q1 | "이 프롬프트 강화해줘: [텍스트]" | prompt-enhance | ✓ enhance | ✓ enhance |
| Q2 | "Improve this prompt so Claude follows it better" | prompt-enhance | ✓ enhance | ✓ enhance |
| Q3 | "이 지시문 좀 더 명확하게 다시 써줘 [텍스트]" | prompt-enhance | ✓ enhance ("다듬어줘" 계열) | ✓ enhance ("재작성 요청" 명시) |
| Q4 | "이 프롬프트 Fable 5 가이드라인에 맞는지 진단해줘" | guideline-check | ✓ check | ✓ check |
| Q5 | "Check my prompt — don't rewrite it, just tell me what's wrong" | guideline-check | ✓ check | ✓ check |
| Q6 | "이 프롬프트 괜찮아? 문제만 알려줘" | guideline-check | ✓ check | ✓ check |
| Q7 | "대시보드 만들고 싶은데 뭐부터 정해야 할지 모르겠어" | deep-interview | ✓ interview | ✓ interview (트리거 문구로 직접 명시) |
| Q8 | "요구사항이 애매한데 질문해 가면서 구체화해줘" | deep-interview | ✓ interview | ✓ interview |
| Q9 | "Help me spec this out — I only have a rough idea" | deep-interview | ✓ interview | ✓ interview |
| Q10 | "https://x.com/... 이 게시물 사례로 등록해줘" | case-ingest | ✓ case | ✓ case |
| Q11 | "이 유튜브 데모 영상 케이스로 추가해줘" | case-ingest | ✓ case | ✓ case |
| Q12 | "Ingest this link as a FableIt case" | case-ingest | ✓ case | ✓ case |
| Q13 | "우리 CLAUDE.md가 Opus 시절 거야. Fable 5 기준으로 저장소 점검해줘" | fable-migrate | ✓ migrate | ✓ migrate |
| Q14 | "Sonnet용 프롬프트들을 Fable 5로 마이그레이션하고 싶어" | fable-migrate | ✓ migrate | ✓ migrate |
| Q15 | "Scan this repo for Fable 5 anti-patterns" | fable-migrate | ✓ migrate | ✓ migrate |
| Q16 | "이 블로그 글 팁으로 정리해서 사이트에 올려줘" | tip-ingest | ✓ tip (v1 초안) | ✓ tip |
| Q17 | "effort 낮춰 쓰는 노하우 글인데 팁으로 등록해줘 <URL>" | tip-ingest | ✓ tip (v1 초안) | ✓ tip |
| Q18 | "팁 제보 이슈 들어온 거 처리해줘" | tip-ingest | ✗ 미발동 — v1 초안에 Issue 제보 모드 문구 없음 | ✓ tip ("팁 제보 이슈 처리해줘" 명시) |

### 스킬 간 혼동 유발 쿼리

| # | 쿼리 | 기대 | Before | After |
|---|---|---|---|---|
| Q19 | "이 프롬프트 좀 봐줘" | guideline-check (수정 미요청 → 진단이 기본, 통제권 사용자) | △ 모호 — enhance "다듬어줘" vs check "괜찮아?" 대등 | ✓ check ("수정 요청이 명시되지 않은 '좀 봐줘/어때?'는 이 스킬이 기본" + enhance 쪽에 "재작성 요청" 한정) |
| Q20 | "이 링크 FableIt에 올려줘" (사례인지 팁인지 내용 미상) | case 또는 tip — 어느 쪽이 받아도 내용 판정 후 핸드오프 | ✗ 오라우팅 위험 — case-ingest가 무조건 수신, 팁 문서여도 사례 JSON으로 오처리 | ✓ 양쪽 description에 경계("세팅·기법·노하우 글은 tip-ingest" / "프롬프트 시연·결과물은 case-ingest") + tip-ingest 절차 3의 판정·핸드오프 |
| Q21 | "CLAUDE.md에 넣을 이 문단, Fable 5에서 문제 없을까? [붙여넣기]" | guideline-check (단일 붙여넣기 — 파일 스캔 아님) | ✓ check — migrate 쪽 기존 경계("붙여넣은 단일 프롬프트의 진단은 guideline-check")가 방어 | ✓ check — check 쪽에도 역방향 경계("프로젝트 파일 일괄 스캔은 fable-migrate") 추가로 양방향 고정 |
| Q22 | "이 프롬프트 평가하고 더 좋게 고쳐줘" | prompt-enhance (재작성까지 요청됨) | △ 모호 — "평가해줘"는 check, "고쳐줘"는 enhance | ✓ enhance ("진단과 수정을 함께 원하는 요청" 명시 + check 쪽 "재작성까지 원하면 prompt-enhance") |
| Q23 | "커뮤니티에서 본 사례 프롬프트인데 분석해줘 [텍스트만, URL 없음]" | guideline-check | △ 모호 — "사례" 명사로 case-ingest 오발동 여지 | ✓ check (case-ingest에 "URL 없이 붙여넣은 프롬프트의 분석·진단은 guideline-check" 추가) |
| Q24 | "아이디어는 있는데 아직 프롬프트로 못 만들었어. 도와줘" | deep-interview (개선할 원문이 없음) | ✓ interview ("아이디어 구체화") | ✓ interview (경계 "이미 작성된 프롬프트의 개선은 prompt-enhance"로 더 선명) |

### 네거티브 쿼리 (어떤 스킬도 발동 금지)

| # | 쿼리 | 기대 | Before | After |
|---|---|---|---|---|
| Q25 | "DB 마이그레이션 스크립트 짜줘" | 발동 없음 | ✓ 없음 — migrate의 "(DB/프레임워크 마이그레이션과 무관)" 경계 | ✓ 없음 |
| Q26 | "이 코드 리뷰해줘" | 발동 없음 | ✓ 없음 — 전 스킬이 대상을 "프롬프트/링크/저장소 지시문"으로 한정 | ✓ 없음 |
| Q27 | "Next.js+shadcn으로 관리자 대시보드 만들어줘. 기능은 A, B, C" | 발동 없음 (요구가 이미 구체적 — 바로 구현) | ✓ 없음 — interview는 "요구사항이 불명확할 때"로 게이트 | ✓ 없음 (interview에 "요구가 이미 구체적인 요청에는 발동하지 않는다" 명문화로 더 견고) |
| Q28 | "프롬프트 엔지니어링이 뭐야? 설명해줘" | 발동 없음 (지식 질문) | ✓ 없음 — 전 트리거가 행동 요청형 | ✓ 없음 |
| Q29 | "Next.js 14에서 15로 마이그레이션 해줘" | 발동 없음 | ✓ 없음 (migrate 경계) | ✓ 없음 |
| Q30 | "Fable 5 잘 쓰는 팁 알려줘" | 발동 없음 (팁을 원하는 질문 ≠ 팁 등록) | ✗ 오발동 위험 — v1 초안("팁 추가/등록") 이 "팁" 명사만으로 매칭될 여지 | ✓ 없음 ("팁을 알려달라는 일반 질문에는 발동하지 않는다 — 문서를 사이트 콘텐츠로 변환하는 파이프라인" 명시) |

## 결과

| | 정발동 | 모호(△) | 오발동·미발동(✗) | 정확도 |
|---|---|---|---|---|
| **Before** | 24 | 3 (Q19, Q22, Q23) | 3 (Q18, Q20, Q30) | **24/30 = 80%** |
| **After** | 30 | 0 | 0 | **30/30 = 100%** |

## Description 변경 요약 (v0.2.0 → v0.3.0)

| 스킬 | 변경 |
|---|---|
| prompt-enhance | "재작성 요청"으로 한정 + "평가하고 고쳐줘"(진단+수정 동시) 명시적 수신 + guideline-check/fable-migrate 경계 추가 (Q19, Q22) |
| guideline-check | 수정 미요청 검토("좀 봐줘/어때?")의 기본 수신자 선언 + prompt-enhance/fable-migrate 역방향 경계 추가 (Q19, Q21, Q22) |
| deep-interview | "뭐부터 정해야 할지 모르겠어" 트리거 추가 + "요구가 이미 구체적이면 미발동" + prompt-enhance 경계 (Q7, Q24, Q27) |
| case-ingest | "프롬프트 시연·결과물 중심(사례)" 대상 한정 + "일반 팁 문서는 tip-ingest" + "URL 없는 붙여넣기 진단은 guideline-check" (Q20, Q23) |
| fable-migrate | 변경 없음 — 기존 경계(단일 프롬프트→check/enhance, DB/프레임워크 무관)가 이미 유효 (Q13–15, Q21, Q25, Q29 방어) |
| tip-ingest (신규) | v1 초안 대비: "--issue 제보 처리" 트리거 추가(Q18), "팁 알려달라는 질문 미발동" 네거티브 가드(Q30), case-ingest 핸드오프 경계(Q20) |

## v0.3.1 재판정 — "실행 모델·시점" 재정렬 (D-23)

> 변경된 description: prompt-enhance("Opus·Sonnet 세션에서 Fable로 보내기 전 준비 단계로
> 실행해도 되며, 대상 모델이 Fable 5가 아니면 범용 규칙만 적용"), guideline-check("Opus·
> Sonnet 세션에서 Fable로 보내기 전에 점검하는 용도로도"), deep-interview("실행 세션 모델과
> 무관하며 큰 작업 착수 전 준비 단계로"). case-ingest·tip-ingest·fable-migrate description은
> 무변경. 우려: 새로 들어온 "Opus·Sonnet" 문구가 fable-migrate의 "Opus 시절 CLAUDE.md"
> 계열(Q13–15)로 오라우팅을 유발하는지.

### 기존 30쿼리 재판정 결과

| 구간 | 결과 |
|---|---|
| Q1–Q3 (enhance) | ✓ 유지 — 추가 문구는 실행 세션·대상 모델 축이라 재작성 트리거를 흐리지 않음 |
| Q4–Q6 (check) | ✓ 유지 |
| Q7–Q9 (interview) | ✓ 유지 — "준비 단계" 문구는 "요구 불명확" 게이트를 넘지 않음 |
| Q10–Q12 (case) | ✓ 유지 (description 무변경) |
| Q13–Q15 (migrate) | ✓ 유지 — **핵심 확인**: enhance/check의 "Opus·Sonnet 세션"은 *실행 세션 모델*을 가리키고, Q13–15는 "저장소/CLAUDE.md 스캔·마이그레이션"(파일 단위)이라 migrate의 "구버전 CLAUDE.md 이행"·"저장소 점검" 경계가 그대로 방어. enhance는 단일 프롬프트 재작성으로 한정돼 오라우팅 없음 |
| Q16–Q18 (tip) | ✓ 유지 (무변경) |
| Q19–Q24 (혼동) | ✓ 유지 — 진단↔재작성·사례↔팁·단일↔스캔 경계 문구가 그대로 유지됨 |
| Q25–Q30 (네거티브) | ✓ 유지 — 새 문구는 모두 행동 요청형 준비 트리거라 지식질문(Q28)·구체 구현(Q27)·팁 질문(Q30)을 잡지 않음 |

**결과: 30/30 = 100% 유지.** 회귀 없음.

### 추가 쿼리 (모델 축 신규 — 정발동 3 + 혼동 1)

| # | 쿼리 | 기대 | 판정 |
|---|---|---|---|
| Q31 | "이 프롬프트 Sonnet에서 쓸 건데 다듬어줘" | prompt-enhance (대상=other, 여전히 재작성) | ✓ enhance — "다듬어줘" 재작성 트리거 + `--for=other` 분기는 본문 처리 |
| Q32 | "Opus 세션인데 이 프롬프트 Fable로 보내기 전에 점검만 해줘" | guideline-check | ✓ check — "점검만"(수정 미요청) + "Fable로 보내기 전" 신규 문구 매칭 |
| Q33 | "큰 작업 시작 전에 요구사항부터 잡고 싶어. 실행은 나중에 Fable로 할 거야" | deep-interview | ✓ interview — "요구사항 잡기"+"착수 전" 매칭, 실행 모델 언급이 라우팅을 바꾸지 않음 |
| Q34 (혼동) | "이 CLAUDE.md Opus 시절 거라 Fable로 이행해줘" | fable-migrate (파일 이행 — enhance 아님) | ✓ migrate — "CLAUDE.md 이행"이 migrate 경계에 정확히 걸리고, enhance의 "Opus 세션" 문구는 *실행 세션*이지 *이행 대상 파일*이 아니라 오발동 없음 |

추가 4쿼리 포함 **34/34 = 100%**.

## 유지보수 규칙

- 스킬을 추가·수정하면 이 매트릭스에 해당 스킬의 정발동 2개 이상 + 혼동 1개 이상을
  추가하고 전체를 재판정할 것
- 판정은 description만 보고 한다 — 본문 절차로 방어되는 케이스라도 description에서
  라우팅이 갈리면 모호로 계산
