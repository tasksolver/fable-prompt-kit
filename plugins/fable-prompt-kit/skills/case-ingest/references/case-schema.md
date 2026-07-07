# FableIt 사례 스키마 (사본)

> SSOT는 `web/lib/schemas.ts`의 zod `CaseSchema`. 스키마 변경 시 이 파일을 재생성할 것.
> 저장 위치: `web/content/cases/{slug}.json` (slug = 파일명)

```jsonc
{
  "slug": "string — ^[a-z0-9-]+$, 파일명과 일치, 주제 요약형 (예: one-shot-3d-game)",
  "title": "string ≤80자 — 한국어 제목",
  "titleOriginal": "string | null — 원제 (번역 시)",
  "category": "coding | frontend | agent | docs | writing | research | etc",
  "surface": "claude-code | claude-ai | api | etc",
  "description": "string ≤400자 — 2-3문장. findings 캐비어트(자기 보고, 미검증 등)는 끝에 한 문장으로 명시",
  "date": "YYYY-MM-DD | null — 원문 게시일, 확인 불가 시 null",
  "addedAt": "YYYY-MM-DD — 사이트 등록일 (오늘)",
  "featured": "boolean — 기본 false, 운영자만 true 지정",
  "author": { "name": "string — 핸들 또는 이름", "url": "URL | null" },
  "source": { "url": "URL — 원문 (필수)", "site": "X | Reddit | YouTube | Blog | GitHub 등" },
  "media": [
    {
      "type": "image | video-embed",
      "src": "image: /media/cases/{slug}.jpg 로컬 경로 · video-embed: 원본 게시물/영상 URL (렌더러 지원: youtube.com/youtu.be, x.com/twitter.com, .mp4/.webm 직링크 — 그 외 도메인은 '원문에서 보기' 폴백)",
      "thumb": "string | null — 빌드 생성 WebP 경로 (/media/cases/thumbs/{이름}.webp)",
      "alt": "string — 필수 (접근성)"
    }
  ],
  "prompt": {
    "isPublic": "boolean",
    "text": "string | null — isPublic=true면 필수 (원문 그대로, 번역·요약 금지)",
    "language": "ko | en | mixed | null"
  },
  "settings": {
    "model": "string | null — 예: claude-fable-5",
    "effort": "low | medium | high | xhigh | max | null",
    "notes": "string | null"
  }, // 전체가 null 가능 — 언급된 것만
  "analysis": {
    "summary": "string — 이 프롬프트가 잘한 점 한 줄",
    "highlights": [ // 1~6개
      {
        "quote": "string — ⚠ prompt.text의 부분 문자열과 정확 일치해야 함 (빌드 검증)",
        "guideline": "clarity | structure | thinking-effort | agentic | output-format | safety-refusal",
        "note": "string ≤200자 — 왜 잘했는지"
      }
    ]
  } // prompt 비공개면 null
}
```

## 검증 규칙 (빌드에서 강제)

1. `prompt.isPublic === true`이면 `prompt.text` 필수
2. `analysis.highlights[].quote`는 `prompt.text`의 부분 문자열이어야 함 — 인용을
   만들 때 반드시 원문에서 복사할 것 (공백·따옴표·백틱까지 그대로)
3. `media[].src`가 로컬 경로(`/`로 시작)면 `web/public` 아래 파일이 존재해야 함
4. `media.type: "image"`는 로컬 경로만 허용 (원격 이미지 핫링크 금지)
