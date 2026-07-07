# fable-prompt-kit

Claude Fable 5 프롬프팅 툴킷 — Claude Code 플러그인(스킬 6종 + 에이전트 1종 + 커맨드 1종). 설치: `/plugin marketplace add tasksolver/fable-prompt-kit` → `/plugin install fable-prompt-kit@fableit`.

## 이 저장소의 위치 (먼저 읽을 것)

- 이 repo는 **공개 배포 미러**다. 사용자가 설치하는 진입점.
- 플러그인의 **정본(canonical) 개발은 비공개 FableIt 모노레포**(`plugins/fable-prompt-kit/`)에서 이뤄지고, 이 repo는 릴리스마다 그 하위트리를 게시한 결과다.
- 따라서 **스킬·지침 내용을 이 repo에서 직접 고치지 말 것** — 모노레포에서 고치고 재게시(아래 "릴리스")한다. 여기서만 고치면 다음 동기화에 덮여 사라진다.
- 예외로 **이 repo 전용 파일**(루트 `README.md`, 이 `CLAUDE.md`, `.gitignore`, `plugin.json`의 `repository` 필드)은 여기가 정본이다 — 동기화가 덮지 않는다.
- 외부 기여는 Issue/PR로 받아 모노레포에 반영 후 재게시.

## 구조

```
.claude-plugin/marketplace.json      # 마켓플레이스 매니페스트 (name: "fableit", source: ./plugins/fable-prompt-kit)
plugins/fable-prompt-kit/
├── .claude-plugin/plugin.json        # 플러그인 매니페스트 (version = semver, repository = 이 repo)
├── skills/<name>/SKILL.md            # 스킬 6종: prompt-enhance · guideline-check · deep-interview · case-ingest · tip-ingest · fable-migrate
├── agents/fable-verifier.md          # 검증 전용 서브에이전트
├── commands/fable-snippets.md        # CLAUDE.md 스니펫 설치 커맨드
├── references/fable5-guidelines.md   # 6 스킬이 공유하는 지침 원천 (단일 갱신점)
├── docs/trigger-eval.md              # 스킬 발동 트리거 평가 매트릭스 (30쿼리)
├── README.md · CHANGELOG.md
README.md · CHANGELOG.md · LICENSE    # 루트 (공개 랜딩)
```

설치 후 마켓플레이스 이름은 `fableit`이므로 설치는 `fable-prompt-kit@fableit`, 마켓플레이스 등록은 `tasksolver/fable-prompt-kit`(repo)다 — 둘은 다르다.

## 절대 규칙 (모노레포에서 편집 시 적용)

- **지침 원천 단일화**: `prompt-enhance`·`guideline-check`·`deep-interview`·`fable-migrate`는 반드시 `references/fable5-guidelines.md`를 읽고 동작한다. 지침을 각 SKILL.md에 중복 기술하지 말 것 — 갱신 지점 1곳.
- **가이드라인 ↔ 사이트 동기화**: `references/fable5-guidelines.md`는 FableIt 사이트의 `web/content/guide/*.mdx`와 **동일 원천**이다. 지침을 바꾸면 모노레포에서 둘을 **함께 커밋**한 뒤 재게시.
- **스키마 사본 주의**: `skills/case-ingest/references/case-schema.md`·`skills/tip-ingest/references/tip-schema.md`는 사이트 zod 스키마(`web/lib/schemas.ts`)의 **사본**이다. 스키마가 바뀌면 이 사본도 함께 갱신(모노레포에서).
- **버전 3곳 동시**: 릴리스마다 `plugin.json`의 `version` + 루트 README 배지 + `CHANGELOG.md`를 함께 올린다. 지침 내용 변경=minor, 버그·문구=patch.
- **실행 모델 분리**: 스킬은 대부분 "Fable에 보내기 전 준비 작업"이라 **아무 모델 세션**에서 실행 가능하다. `prompt-enhance`·`guideline-check`는 대상이 Fable 5가 아니면 Fable 전용 규칙(§3.1 파라미터, §1.4 de-prescribe)을 빼고 범용 규칙만 적용한다(`--for`/`--target`).

## 스킬 편집 규약

- SKILL.md 프론트매터: `name` + `description`. **description이 스킬 발동을 좌우**한다 — 트리거 조건과 한/영 키워드를 병기하고, 상호 핸드오프 경계를 명시(사례↔팁 = `case-ingest`↔`tip-ingest`, 진단↔재작성 = `guideline-check`↔`prompt-enhance`, 단일 프롬프트↔프로젝트 스캔 = ↔`fable-migrate`). 네거티브 가드도(오발동 방지).
- description을 바꾸면 `docs/trigger-eval.md`의 30쿼리로 재판정해 발동/미발동 100% 유지 확인 후 커밋.
- 본문 구조: 목적 → 입력 처리 → 절차 → 출력 형식 → 엣지 케이스. 지시문은 한국어, 사용자 입력 언어를 따라 응답.

## 테스트 (설치 후 실호출)

```text
/plugin marketplace add tasksolver/fable-prompt-kit
/plugin install fable-prompt-kit@fableit
```

각 스킬 대표 시나리오 1회 이상:
- `prompt-enhance`: 모호한 프롬프트 1개 → 출력 형식(강화 프롬프트/변경 요약/남은 빈틈) 준수
- `guideline-check`: 잘된 프롬프트(억지 지적 없는지) + 나쁜 프롬프트 각 1
- `deep-interview`: 한 줄 요구 → 인터뷰 → 결정 요약표
- `case-ingest` / `tip-ingest`: URL → 스키마 유효 산출물 (case는 JSON, tip은 MDX)
- `fable-migrate`: 레거시 CLAUDE.md 스캔 → 심각도별 리포트
- 트리거 오발동/미발동은 `docs/trigger-eval.md` 쿼리로 점검

## 릴리스 (모노레포 → 이 미러 재게시)

모노레포에서 플러그인 하위트리를 이 repo 로컬 클론으로 복사한 뒤 커밋·푸시한다. 이 repo 전용 파일(루트 README·CLAUDE.md·plugin.json repository)은 덮지 않는다:

```bash
MONO=/path/to/fableit             # 비공개 모노레포
PUB=/path/to/fable-prompt-kit     # 이 repo 로컬 클론
cd "$MONO"
rsync -a --delete plugins/fable-prompt-kit/ "$PUB/plugins/fable-prompt-kit/"
cp .claude-plugin/marketplace.json "$PUB/.claude-plugin/marketplace.json"
cp LICENSE "$PUB/LICENSE"
cp plugins/fable-prompt-kit/CHANGELOG.md "$PUB/CHANGELOG.md"
# 이 repo 전용 값 복구 (rsync가 plugin.json을 모노레포 값으로 덮으므로):
sed -i '' 's|github.com/tasksolver/fableit"|github.com/tasksolver/fable-prompt-kit"|' \
  "$PUB/plugins/fable-prompt-kit/.claude-plugin/plugin.json"
# 루트 README 배지 버전은 필요 시 수동 갱신 (README는 rsync 대상 아님)
cd "$PUB" && git add -A && git commit -m "sync vX.Y.Z from monorepo" && git push
```

**gotcha**: 모노레포의 `plugin.json`은 `repository`가 `tasksolver/fableit`을 가리키므로 매 동기화마다 위 `sed` 한 줄로 이 repo 값으로 되돌린다. 모노레포 쪽 `plugin.json`의 `repository`를 아예 `tasksolver/fable-prompt-kit`으로 맞춰두면 이 보정이 필요 없어진다(권장).

## 참고

- Anthropic과 무관한 비공식 리소스. Claude·Fable은 Anthropic PBC의 상표.
- 짝이 되는 레퍼런스 사이트: https://fableit.pages.dev (지침 원천 자료를 한국어로 정리)
