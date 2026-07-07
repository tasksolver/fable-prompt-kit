---
name: tip-ingest
description: >
  URL(블로그/공식 문서/X/Reddit)이나 GitHub Issue 제보를 분석해 FableIt 팁
  MDX(web/content/tips/{slug}.mdx)를 생성한다. "이 글 팁으로 등록해줘", "팁 추가",
  "팁 제보 이슈 처리해줘", "add this as a tip" 같은 사이트 콘텐츠 등록 요청이나
  /tip-ingest 호출에 사용. 팁을 알려달라는 일반 질문에는 발동하지 않는다 — 이 스킬은
  문서를 사이트 콘텐츠로 변환하는 파이프라인이다. 프롬프트 시연·결과물 중심
  게시물(활용 사례)은 case-ingest 담당. Fable 5 / Opus 4.8에서 검증됨.
---

# tip-ingest — 링크 분석·팁 수집

## 목적

URL 또는 GitHub Issue 제보에서 Fable 5 세팅·기법·노하우·문제 해결 정보를 추출해
FableIt 팁 스키마(MDX + 프론트매터)에 맞는 문서를 생성한다. 검증 상태(verified)를
판정하고 관련 가이드 카테고리를 연결한다.

## 실행 모델·시점

- **실행 모델**: 아무 모델 — 실행 모델과 무관한 콘텐츠 파이프라인이다. 어떤 세션에서
  돌려도 결과가 같다.

## 입력 처리

- `/tip-ingest <URL>` — 블로그/공식 문서/X/Reddit/GitHub 등
- `/tip-ingest --issue <issue URL 또는 본문 붙여넣기>` — GitHub Issue 제보 처리
  모드. 사이트의 tip-submission Issue Form 필드 id(title, category, body,
  sourceUrl, verified)가 스키마 필드명과 대응하므로 본문을 그대로 파싱한다

## 절차

1. `${CLAUDE_PLUGIN_ROOT}/skills/tip-ingest/references/tip-schema.md`를 읽어 스키마를
   확인한다
2. WebFetch로 URL을 수집한다. X 등 접근 차단 시: "게시물 본문을 붙여넣어 주세요" 폴백
   요청
3. **팁 여부 판정 (핸드오프 경계)**: 원문이 프롬프트 시연·결과물 자랑 중심(스크린샷,
   데모 영상, 공개된 프롬프트 원문)이면 **사례**다 — "이 링크는 활용 사례에 가깝습니다.
   `/case-ingest <URL>`로 처리할까요?"라고 안내하고 중단한다. 세팅값·기법·운용
   노하우·문제 해결이 중심이면 팁으로 진행한다. 애매하면(둘 다 포함) 사용자에게 어느
   쪽으로 등록할지 질문한다
4. **추출**: 제목 / 작성자(핸들·URL) / 게시일 / 핵심 주장 / 설정값·커맨드 /
   근거(공식 문서 인용 여부, 실측 데이터 여부)
5. **verified 판정** (근거를 확신도 표에 기록):
   - `official` — 주장이 Anthropic 공식 문서·발표와 일치함을 확인한 경우만
   - `community` — 커뮤니티 경험담·실측 (공식 문서로 확인 불가하지만 출처·재현 조건이
     구체적)
   - `unverified` — 재현 조건 불명, 자기 보고 수치, 원문 재확인 불가
   - **캐비어트 승계**: 원문이 자기 보고 수치(성능 향상률, 비용 절감 등)를 주장하면
     본문에 "…는 자체 보고 수치로 독립 검증되지 않았다" 류 문장을 명시하고, 공식 권고와
     충돌하는 주장은 `<Callout tone="warn">`으로 양쪽을 병기한다 — 원문 주장을 공식
     사실처럼 승격하지 않는다
6. **relatedGuides 연결**: `${CLAUDE_PLUGIN_ROOT}/references/fable5-guidelines.md`를
   읽고 팁 내용과 직접 관련된 가이드 카테고리(clarity / structure / thinking-effort /
   agentic / output-format / safety-refusal)를 0–2개 선정한다. 억지 연결 금지 —
   관련 없으면 빈 배열
7. **MDX 생성**: 프론트매터 + 본문(한국어, 설정값·커맨드는 코드 블록). 규칙:
   - **출처 필수**: URL 유래 팁은 `source.url`에 원문을 기록한다. Issue 제보에서
     sourceUrl이 비어 있으면(본인 경험담) Issue URL을 출처로 쓰고 제보자를 author로
     기록한다. 출처를 어디에도 확보할 수 없으면 **생성하지 않는다**
   - 모델 수치(가격·컨텍스트·전환일 등)는 하드코딩하지 않고 `<Fact name="..." />`
     컴포넌트를 쓴다 (D-09)
   - 확인 불가 필드는 null — 추측 금지. 각 필드의 근거를 확신도 표로 정리한다
8. **저장** (FableIt 저장소 안에서 실행 중일 때만): `web/content/tips/{slug}.mdx`
   저장 + `web/`에서 `pnpm validate:content`를 실행해 통과를 확인한다. 저장소 밖이면
   MDX만 출력한다

## 출력 형식

- 생성된 MDX (코드 블록)
- 필드별 확신도 표: `필드 | 확인됨 / 추정(근거) / null 처리` + verified 판정 근거 한 줄
- 저장까지 한 경우: 파일 경로 + validate 결과

## 엣지 케이스

- **사례성 게시물**: 절차 3의 핸드오프 — case-ingest 안내 후 중단
- **접근 불가 URL**: 본문 붙여넣기 폴백 안내
- **동일 slug 존재**: 기존 파일과의 병합 여부를 사용자에게 질문
- **기존 팁과 내용 중복**: `web/content/tips/`의 기존 slug 목록을 훑어 유사 주제가
  있으면 새 파일 대신 기존 팁 보강을 제안한다
- **공식 문서와 충돌하는 주장**: 삭제하지 말고 `<Callout tone="warn">`으로 공식
  권고와 커뮤니티 주장을 병기 (기존 팁 effort-selection의 처리 방식과 동일)

## 주의

이 스킬은 Claude Code 전용이다 — 웹사이트에서 AI를 실행하지 않는다(D-03). 생성된
MDX는 커밋 후 정적 빌드로 사이트에 반영된다.
