<div align="center">

# fable-prompt-kit

**Claude Fable 5 프롬프팅 툴킷** — 프롬프트를 Fable 5 방식으로 다듬고, 기존 프로젝트를 이행하고, 결과를 검증하는 Claude Code 플러그인

[![plugin](https://img.shields.io/badge/fable--prompt--kit-v0.3.2-2DD4BF)](./plugins/fable-prompt-kit)
[![license](https://img.shields.io/badge/license-MIT-blue)](./LICENSE)

[설치](#-설치-30초) · [무엇이 설치되나](#-무엇이-설치되나) · [사용 예시](#-사용-예시) · [언제 어떤 모델에서](#-언제-어떤-모델에서-쓰나)

</div>

> 이 저장소는 Claude Code 플러그인 마켓플레이스입니다. 아래 두 줄로 `fable-prompt-kit`을 설치하세요.
> Anthropic과 무관한 비공식 커뮤니티 리소스이며, Claude·Fable은 Anthropic PBC의 상표입니다.

## 🧩 설치 (30초)

Claude Code에서 두 줄이면 끝납니다:

```text
/plugin marketplace add tasksolver/fable-prompt-kit
/plugin install fable-prompt-kit@fableit
```

첫 줄이 이 저장소를 마켓플레이스로 등록하고, 둘째 줄이 플러그인을 설치합니다.

모든 지침은 Anthropic 공식 문서(Prompting Claude Fable 5, Migration guide)가 원천이며 [`references/fable5-guidelines.md`](./plugins/fable-prompt-kit/references/fable5-guidelines.md) 한 파일로 압축되어 모든 스킬이 공유합니다. **Fable 5 / Opus 4.8에서 검증됨.**

## 📦 무엇이 설치되나

**스킬 6종 + 에이전트 1종 + 커맨드 1종.**

| 호출 | 하는 일 |
|---|---|
| `/prompt-enhance <프롬프트>` | 프롬프트를 목표·제약·성공 기준 구조로 재작성. 원문에 없는 요구는 지어내지 않고 `[선택: ...]`으로 위임 (`--minimal`, `--target=<code\|claude-ai\|api>`, `--for=<fable\|other>`) |
| `/guideline-check <프롬프트>` | 재작성 없이 진단만 — 6개 카테고리(명확성/구조화/사고 제어/에이전트/출력 형식/안전) 판정표 + 심각도순 발견 사항 + 우선 수정 Top 3 |
| `/deep-interview <막연한 요구>` | 최대 3라운드 선택지형 인터뷰로 요구사항 구체화. 모든 질문에 권장안 제공 — "다 추천대로" 한마디로 즉시 완성 |
| `/case-ingest <URL>` | X/YouTube/GitHub/블로그 링크를 분석해 사례 스키마 JSON 생성. 확인 불가 필드는 null(추측 금지), 미디어는 라이선스에 따라 임베드/다운로드 분기 |
| `/tip-ingest <URL>` | 링크를 분석해 팁 MDX 문서 초안 생성. 출처·검증 상태(공식 일치/커뮤니티/미확인)를 프론트매터에 병기 |
| `/fable-migrate [경로] [--apply]` | 이전 모델용 CLAUDE.md·스킬·에이전트·API 코드를 스캔해 Fable 5 안티패턴 감사. 기본은 리포트만, `--apply`로 안전한 항목 자동 적용 |
| `fable-verifier` (에이전트) | 구현 맥락에 오염되지 않은 신선한 컨텍스트로 산출물을 스펙과 대조하는 검증 전용 서브에이전트 |
| `/fable-snippets [전부\|번호\|키워드]` (커맨드) | 공식 권장 스니펫 11종(행동 경계, 범위 제한, 진행 보고 접지, 조기 중단 방지 등)을 골라 CLAUDE.md에 영어 원문 그대로 설치 |

상세 문서: [plugins/fable-prompt-kit](./plugins/fable-prompt-kit)

## 💡 사용 예시

**막연한 요구를 실행 가능한 프롬프트로:**

```text
/deep-interview 우리 팀 코드 리뷰 봇 만들고 싶어
```
→ 대상 언어·리뷰 범위·출력 형식 등을 선택지로 물어보고, 답을 모으면 바로 쓸 수 있는 프롬프트로 마감합니다. 애매한 항목은 권장 기본값으로 채웁니다.

**이전 모델용 프로젝트를 Fable 5로 이행:**

```text
/fable-migrate --apply
```
→ CLAUDE.md와 스킬·API 코드에서 추론 전사 지시(refusal 유발), 무효 API 파라미터(400 오류) 같은 안티패턴을 심각도별로 리포트하고, 안전한 항목은 자동 수정합니다.

## 🧭 언제 어떤 모델에서 쓰나

이 스킬들은 대부분 **Fable에 보내기 전 준비 작업**이라 실행 세션은 아무 모델이나 됩니다 — 저렴한 Opus·Sonnet 세션에서 준비하고 Fable 세션/API에서 실행하는 흐름이 표준입니다. (`/prompt-enhance`·`/guideline-check`는 대상이 Fable 5가 아니면 Fable 전용 규칙을 빼고 범용 규칙만 적용합니다.)

```text
준비(아무 세션)                        실행(Fable)              검증
/deep-interview 요구 구체화        →
/prompt-enhance 프롬프트 강화      →  강화된 프롬프트로 실행  →  fable-verifier로 검증
/fable-migrate  기존 프로젝트 이행 →  이행된 지시문으로 실행
```

## 요구사항

- **Claude Code** (Fable 5로 실행 시 v2.1.170+). 스킬은 사용자 본인의 요금으로 실행됩니다.
- 버저닝은 semver — 가이드라인 내용이 바뀌면 minor 증가. 변경 이력: [CHANGELOG](./CHANGELOG.md)

## 🌐 레퍼런스 사이트: FableIt

플러그인과 짝을 이루는 한국어 레퍼런스 사이트 **[fableit.pages.dev](https://fableit.pages.dev)** — 플러그인이 참조하는 가이드라인(공식 프롬프팅 문서 지침 20건)의 원천 자료를 Before/After·출처와 함께 정리합니다.

## 🤝 기여·문의

- **플러그인 버그·개선 제안**: 이 저장소의 [Issues](https://github.com/tasksolver/fable-prompt-kit/issues)
- 지침은 Anthropic 공식 문서가 원천이며, 인용·권리는 각 원저작자에게 있습니다. 문제가 있으면 이슈로 알려주세요.

## 라이선스

코드 [MIT](./LICENSE).
