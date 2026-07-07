# fable-prompt-kit

Claude Fable 5 툴킷 — 프롬프트를 Fable 5 방식으로 다듬고, 기존 프로젝트를 이행하고, 작업 결과를 검증하는 Claude Code 플러그인.

```text
/plugin marketplace add tasksolver/fableit
/plugin install fable-prompt-kit@fableit
```

모든 지침은 Anthropic 공식 문서(Prompting Claude Fable 5, Migration guide)가 원천이며, [`references/fable5-guidelines.md`](./references/fable5-guidelines.md) 한 파일로 압축되어 모든 스킬이 공유합니다. Fable 5 / Opus 4.8에서 검증됨.

## 스킬 6종

### `/prompt-enhance <프롬프트>`
프롬프트를 목표·제약·성공 기준 구조로 재작성. 원문에 없는 요구사항은 지어내지 않고 `[선택: ...]`으로 위임.
- `--minimal` 최소 수정 · `--target=<code|claude-ai|api>` 서피스 최적화
- 이미 훌륭한 프롬프트에는 "수정 불필요"로 응답 (억지 수정 없음)

### `/guideline-check <프롬프트>`
재작성 없이 진단만 — 6개 카테고리(명확성/구조화/사고 제어/에이전트/출력 형식/안전) 판정표 + 심각도순 발견 사항 + 우선 수정 Top 3.

### `/deep-interview <막연한 요구>`
최대 3라운드 선택지형 인터뷰로 요구사항 구체화. 취향·되돌리기 비싼 결정·방향 분기만 골라 묻고 나머지는 근거와 함께 기본값으로 채움. 모든 질문에 권장안이 1번 — "다 추천대로" 한마디로 즉시 완성.

### `/fable-migrate [경로] [--apply]`
쓰던 프로젝트의 모델을 Fable 5로 바꾸기 전·직후에 실행 — 구모델 시절 CLAUDE.md·스킬·API 코드에 남은 안티패턴(400 오류·refusal·품질 저하 원인)을 감사:

| 심각도 | 안티패턴 |
|---|---|
| 높음 | 추론 전사 지시(refusal 유발), 무효 API 파라미터(400 오류) |
| 중간 | 짧은 타임아웃, step-by-step 강제, max_tokens 미조정 |
| 낮음 | 검증 리마인더, 페르소나 과장 |

기본은 리포트만, `--apply`로 안전한 항목 자동 적용.

### `/case-ingest <URL>`
X/YouTube/GitHub/블로그 링크를 분석해 [FableIt 사례 스키마](./skills/case-ingest/references/case-schema.md) JSON 생성. 확인 불가 필드는 null(추측 금지), 미디어는 출처 라이선스에 따라 임베드/다운로드 분기. 일반 팁·노하우 글은 `/tip-ingest` 담당.

### `/tip-ingest <URL | --issue>`
블로그 글·GitHub Issue 제보를 분석해 [FableIt 팁 스키마](./skills/tip-ingest/references/tip-schema.md) MDX 생성. 출처 필수, verified(official/community/unverified) 판정과 캐비어트 승계, 관련 가이드 카테고리 연결까지. 프롬프트 시연·결과물 중심 게시물(사례)은 `/case-ingest` 담당.

## 에이전트

**`fable-verifier`** — 구현 맥락에 오염되지 않은 신선한 컨텍스트로 산출물을 스펙과 대조하는 검증 전용 서브에이전트. 공식 가이드의 "자기비판보다 별도 검증 서브에이전트가 낫다" 권고의 구현체. 긴 작업 중 "fable-verifier로 지금까지 작업 검증해줘"라고 쓰면 됩니다.

## 커맨드

**`/fable-snippets [전부 | 번호 | 키워드]`** — 공식 권장 스니펫 11종(행동 경계, 범위 제한, 진행 보고 접지, 조기 중단 방지 등)을 골라 프로젝트 CLAUDE.md에 영어 원문 그대로 설치. 중복 설치 방지 내장.

## 언제 어떤 모델에서 쓰나

이 스킬들은 대부분 **Fable에 보내기 전 준비·정비 작업**이다 — 실행 세션은 아무 모델이나 되며, 구독 플랜에서 Fable 사용이 끝난 뒤에는 **저렴한 Opus·Sonnet 세션에서 준비 → Fable 세션/API에서 실행 → 검증**이 표준 흐름이다. 산출물이 향하는 **대상 모델**과 스킬을 **실행하는 모델**은 별개이며, `/prompt-enhance`·`/guideline-check`는 대상이 Fable 5가 아니면 Fable 전용 규칙을 빼고 범용 규칙만 적용한다.

```
[준비: 아무 세션 — Opus·Sonnet 권장]        [실행: Fable 세션·API]        [검증]
/deep-interview  요구 구체화             →
/prompt-enhance  프롬프트 강화(대상=Fable) →  강화된 프롬프트로 실행     →  fable-verifier로 대조 검증
/fable-migrate   프로젝트 이행           →  이행된 지시문으로 실행
/fable-snippets  새 프로젝트 스니펫 설치  →
/case-ingest·/tip-ingest  콘텐츠 파이프라인 (실행 모델 무관)
```

- 새 프로젝트: `/fable-snippets 전부` → 필요할 때 `/prompt-enhance`
- 기존 프로젝트: `/fable-migrate` → 리포트 보고 `--apply`
- 요구가 막연: `/deep-interview` → 산출물을 `/prompt-enhance`로 마감
- 콘텐츠 운영: 사례 링크는 `/case-ingest`, 팁 글·제보는 `/tip-ingest`

## 요구사항·버전

- Claude Code (Fable 5로 실행 시 v2.1.170+). 스킬은 사용자 본인의 요금으로 실행됩니다
- 버저닝: semver — 가이드라인 내용 변경 시 minor 증가. 변경 이력: [CHANGELOG](./CHANGELOG.md)
