# Changelog — fable-prompt-kit

포맷: [Keep a Changelog](https://keepachangelog.com/ko/1.1.0/) · 버저닝: semver (가이드라인 내용 변경 시 minor 증가).

## 0.3.2 — 2026-07-08

- deep-interview: 질문 선별 기준(취향·되돌리기 비용·방향 분기만 인터뷰, 나머지는 근거와
  함께 기본값)과 빈틈 탐지 3기법(실행 시뮬레이션·검수 프리모템·미정의 표현 추출) 명문화
- fable-migrate: 전형적 사용 시점 3가지(전환 직전 점검 / 전환 직후 문제 규명 / 물려받은
  자산 감사)를 SKILL.md·웹 소개 페이지에 명시

## 0.3.1 — 2026-07-08

### Changed
- **스킬 6종 "실행 모델·시점" 재정렬** (D-23): 스킬을 돌리는 **실행 모델**(아무 모델 — Opus·Sonnet 세션 권장, Fable에 보내기 전 준비 작업)과 산출물이 향하는 **대상 모델**을 분리해 각 SKILL.md에 명시. `prompt-enhance`·`guideline-check`는 대상 모델이 Fable 5가 아니면 Fable 전용 규칙(§3.1 파라미터, §1.4 de-prescribe)을 적용/지적하지 않고 범용 규칙만 쓰도록 절차에 분기 추가. `prompt-enhance`에 `--for=<fable|other>` 플래그 신설
- **`references/fable5-guidelines.md` 도입부**: 이 레퍼런스는 "Fable 5로 보낼 프롬프트"용이고 실행 모델과 무관함 + Fable 전용(§1.4/§3.1/§3.2/§4.1/§6) vs 전 모델 공통(§1.1~1.3/§2/§5) 지침 구분 명시
- **README 워크플로**: "저렴한 세션에서 준비 → Fable 세션/API에서 실행 → fable-verifier로 검증" 관점으로 도식 갱신 (플러그인 README + 루트 README)
- 6종 description에 상황 트리거 추가(Opus·Sonnet 세션·Fable로 보내기 전) — trigger-eval 30쿼리 재판정 100% 유지 (`docs/trigger-eval.md` v0.3.1 섹션)

## 0.3.0 — 2026-07-07

### Added
- **스킬 `tip-ingest`**: URL·GitHub Issue 제보를 분석해 FableIt 팁 MDX(`web/content/tips/{slug}.mdx`)를 생성 — verified 판정, relatedGuides 연결, 캐비어트 승계 (D-18: case-ingest 확장이 아닌 별도 스킬로 분리)
- 스킬 트리거 평가 매트릭스 `docs/trigger-eval.md` (쿼리 30개, before 80% → after 100%)

### Changed
- **스킬 6종 전체 description 트리거 최적화**: 상호 핸드오프 경계 명시 — 사례↔팁(case-ingest↔tip-ingest), 진단↔재작성(guideline-check↔prompt-enhance), 단일 프롬프트↔프로젝트 스캔(↔fable-migrate). 네거티브 가드 추가(팁 질문·구체적 구현 요청 미발동)
- `<owner>` 플레이스홀더를 `tasksolver`로 치환 (plugin.json, marketplace.json, README)

## 0.2.0 — 2026-07-07

### Added
- **스킬 `fable-migrate`**: 이전 모델용 CLAUDE.md·스킬·API 코드의 Fable 5 안티패턴 감사 (`--apply` 자동 적용)
- **에이전트 `fable-verifier`**: 신선한 컨텍스트 스펙 대조 검증 서브에이전트
- **커맨드 `/fable-snippets`**: 공식 권장 스니펫 11종 CLAUDE.md 설치
- plugin.json 메타데이터 보강 (license, keywords, repository, homepage)

### Changed
- 모든 스킬의 참조 경로를 `${CLAUDE_PLUGIN_ROOT}` 기반으로 통일 (이식성)
- 실호출 테스트 피드백 반영: 판정-발견 매핑 기준, 부분 위임 처리, surface/date 판별 규칙 명문화

## 0.1.0 — 2026-07-07

### Added
- 스킬 4종: `prompt-enhance`, `guideline-check`, `deep-interview`, `case-ingest`
- 공유 레퍼런스 `references/fable5-guidelines.md` (공식 문서 지침 20건 압축)
- 마켓플레이스 매니페스트 (`.claude-plugin/marketplace.json`)
