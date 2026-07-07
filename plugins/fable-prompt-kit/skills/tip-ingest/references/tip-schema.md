# FableIt 팁 스키마 (사본)

> SSOT는 `web/lib/schemas.ts`의 zod `TipFrontmatter`. 스키마 변경 시 이 파일을 재생성할 것.
> 저장 위치: `web/content/tips/{slug}.mdx` (slug = 파일명, MDX 프론트매터 + 한국어 본문)

```yaml
---
slug: "string — ^[a-z0-9-]+$, 파일명과 일치, 주제 요약형 (예: effort-selection)"
title: "string ≤80자 — 한국어 제목"
category: "model-settings | claude-code | system-prompt | cost | long-running | troubleshooting"
summary: "string ≤200자 — 인덱스 카드용 요약. 캐비어트(미검증 등)가 있으면 여기에도 짧게"
verified: "official | community | unverified"
date: "YYYY-MM-DD — 등록일 (오늘)"
author: # 전체 null 가능 — 운영자 작성이면 null
  name: "string — 핸들 또는 이름"
  url: "URL | null"
source: # 전체 null은 운영자 직접 작성일 때만 — 외부 유래 팁은 필수
  url: "URL — 원문"
  site: "X | Reddit | YouTube | Blog | GitHub 등"
relatedGuides: [] # clarity | structure | thinking-effort | agentic | output-format | safety-refusal — 0~2개, 억지 연결 금지
---
```

## category 선택 기준

| 값 | 대상 |
|---|---|
| `model-settings` | effort/thinking 등 모델 세팅 |
| `claude-code` | Claude Code 설정·커맨드·워크플로 |
| `system-prompt` | 시스템 프롬프트·CLAUDE.md 작성법 |
| `cost` | 비용 최적화 |
| `long-running` | 장기 실행·에이전트 운용 |
| `troubleshooting` | 문제 해결 (refusal 오탐, 버그 우회 등) |

## verified 판정 기준

- `official` — Anthropic 공식 문서·발표와 일치함을 **확인한** 경우만
- `community` — 커뮤니티 경험담·실측 (출처·재현 조건 구체적)
- `unverified` — 재현 조건 불명, 자기 보고 수치, 원문 재확인 불가

## 본문 작성 규약

- 한국어. 설정값·커맨드는 코드 블록, 영어 공식 스니펫은 원문 그대로(번역 금지)
- 사용 가능한 MDX 컴포넌트: `<Callout tone="info|warn" title="...">`,
  `<SettingsTable rows={[...]} />`, `<Fact name="..." />`
- **모델 수치 하드코딩 금지** (D-09): 가격·컨텍스트·전환일은 `<Fact name="pricing" />`
  `<Fact name="contextWindow" />` `<Fact name="subscriptionCutoff" />` 등으로 참조
- 내부 링크: `/tips/{slug}/`, `/guide/{category}/` (트레일링 슬래시 포함)
- 미검증·자기 보고 주장은 본문에 명시하고, 공식 권고와 충돌 시 `<Callout tone="warn">`으로 병기

## 검증 규칙 (빌드에서 강제)

1. 프론트매터가 zod `TipFrontmatter`를 통과해야 함 (`pnpm validate:content`)
2. `slug`는 파일명과 일치, `date`는 `YYYY-MM-DD`
3. `relatedGuides` 값은 가이드 카테고리 6종 enum만 허용
