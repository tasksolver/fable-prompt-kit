---
name: case-ingest
description: >
  URL(X/Reddit/YouTube/블로그/GitHub)을 분석해 FableIt 사례 스키마 JSON을 생성한다.
  "이 링크 사례로 정리해줘", "사례 수집", "케이스 추가", "ingest this link" 같은
  요청이나 /case-ingest 호출에 사용. 주 사용자는 FableIt 운영자(콘텐츠 파이프라인)지만
  일반 사례 정리 도구로도 동작. 프롬프트 시연·결과물 중심 게시물(사례)이 대상이며,
  세팅·기법·노하우 글 등 일반 팁 문서는 tip-ingest 담당. URL 없이 붙여넣은 프롬프트의
  분석·진단은 guideline-check. Fable 5 / Opus 4.8에서 검증됨.
---

# case-ingest — 링크 분석·사례 수집

## 목적

URL에서 확보 가능한 모든 데이터를 추출해 FableIt 사례 스키마(JSON)에 맞는 항목을
생성한다. 프롬프트가 공개된 경우 가이드라인 분석(highlights)까지 만든다.

## 실행 모델·시점

- **실행 모델**: 아무 모델 — 실행 모델과 무관한 콘텐츠 파이프라인이다. 어떤 세션에서
  돌려도 결과가 같다.

## 입력 처리

- `/case-ingest <URL>` — X/Reddit/YouTube/블로그/GitHub
- `/case-ingest --issue <issue URL 또는 본문 붙여넣기>` — GitHub Issue 제보 처리
  모드. 사이트의 case-submission Issue Form 필드 id가 스키마 필드명과 동일하므로 본문을
  그대로 파싱한다

## 절차

1. `${CLAUDE_PLUGIN_ROOT}/skills/case-ingest/references/case-schema.md`를 읽어 스키마를 확인한다
2. WebFetch로 URL을 수집한다. X 등 접근 차단 시: "게시물 본문을 붙여넣어 주세요" 폴백
   요청
3. **추출**: 제목 / 작성자(핸들·URL) / 게시일 / 본문 / 프롬프트 원문 / 미디어 URL /
   사용 세팅 언급 (모델·effort 등)
   - GitHub 사례의 게시일: 해당 디렉터리/파일의 최초 커밋 날짜로 확정 가능 (커밋
     히스토리 조회) — 확인되면 null 대신 사용
   - `surface` 판별: 원문에 사용 도구가 명시된 경우만 구체 값(claude-code 등), 저장소
     구조 증거(.claude/ 디렉터리 등)가 있으면 그것을 근거로 기록, 불명이면 `etc` + 확신도
     표에 "표면 미상" 기록
4. **분석 생성** (프롬프트가 공개된 경우만): `${CLAUDE_PLUGIN_ROOT}/references/fable5-guidelines.md`를 읽고, 프롬프트가 잘 적용한 구간 1–6개를
   `analysis.highlights`로 만든다. **인용(quote)은 프롬프트 원문에서 그대로 복사한
   부분 문자열이어야 한다** — 요약·의역하면 사이트 빌드 검증(zod)에 걸린다
5. JSON 생성. **확인 불가 필드는 null — 추측 금지.** 각 필드의 근거를 확신도 표로
   정리한다
6. **미디어 처리 — 출처 기반 분기 (저작권 리스크 관리)**:
   - **SNS 출처** (X/Twitter, Reddit, Instagram 등): 이미지·영상 모두 **다운로드
     금지, 공식 임베드만**. `media.type: "video-embed"`, `src`에 원본 게시물 URL(X는
     트윗 URL, YouTube는 watch URL). 트윗 내 스크린샷도 재호스팅하지 않고 게시물 자체를
     임베드한다
   - **오픈소스 저장소 출처** (GitHub 등 라이선스가 명확한 경우): 스크린샷·정적 이미지는
     `media.type: "image"`로 `web/public/media/cases/{slug}.jpg`에 다운로드 가능
     (빌드가 WebP 썸네일 자동 생성). 단 저장소의 데모 영상(mp4 등)은 이미지와 달리
     다운로드하지 않고 원본 링크를 `video-embed`로 임베드
   - **출처가 애매하거나 라이선스 확인 불가**: 항상 더 안전한 쪽인 **임베드**를 기본값으로
     하고, 확신도 표에 "라이선스 미확인 → 임베드로 대체"라고 기록
7. **저장** (FableIt 저장소 안에서 실행 중일 때만): `web/content/cases/{slug}.json`
   저장 + (다운로드 대상이 있으면) 미디어 저장 + `web/`에서 `pnpm validate:content`
   실행해 통과를 확인한다. 저장소 밖이면 JSON만 출력한다
8. 다운로드한 경우(오픈소스 이미지)에도 원저작자 표기·출처 링크가 필수임을 결과에
   리마인드한다

## 출력 형식

- 생성된 JSON (코드 블록)
- 필드별 확신도 표: `필드 | 확인됨 / 추정(근거) / null 처리`
- 저장까지 한 경우: 파일 경로 + validate 결과

## 엣지 케이스

- **프롬프트 비공개 게시물**: `prompt.isPublic: false`, `text: null`, 분석 생략
- **접근 불가 URL**: 본문 붙여넣기 폴백 안내
- **동일 slug 존재**: 기존 파일과의 병합 여부를 사용자에게 질문
- 캐비어트 승계: 원문이 자기 보고 수치(비용 절감률, 사용자 수 등)를 주장하면
  description 끝에 "…는 자체 보고 수치로 독립 검증되지 않았다" 한 문장을 붙인다
