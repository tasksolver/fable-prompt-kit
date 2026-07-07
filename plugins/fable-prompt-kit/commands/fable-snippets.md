---
description: Fable 5 공식 권장 스니펫 11종에서 골라 CLAUDE.md에 설치
argument-hint: "[전부 | 번호들 | 키워드] (생략 시 목록 표시)"
---

Fable 5 공식 프롬프팅 문서의 권장 스니펫을 이 프로젝트의 CLAUDE.md에 설치하는 작업이다.

1. `${CLAUDE_PLUGIN_ROOT}/references/fable5-guidelines.md`를 읽어 아래 11종 스니펫의 **영어 원문**을
   확보하라 (번역 금지 — 원문이 검증된 표현이다):

   | # | 스니펫 | 용도 | 붙이는 곳 |
   |---|---|---|---|
   | 1 | 행동 경계 (state the boundaries) | 문제 서술≠수정 요청 구분 | CLAUDE.md |
   | 2 | 범위 제한 (no-tidying) | 과잉 리팩터링·기능 추가 억제 | CLAUDE.md |
   | 3 | 과도한 계획 방지 (act when ready) | 옵션 나열 대신 실행 | CLAUDE.md |
   | 4 | 일시정지 기준 (pause only when…) | 체크포인트 행동 | CLAUDE.md |
   | 5 | 진행 보고 접지 (ground progress claims) | 허위 보고 방지 | CLAUDE.md |
   | 6 | 서브에이전트 위임 (delegate independent…) | 병렬 활용 | CLAUDE.md |
   | 7 | 메모리 운영 규칙 (one lesson per file) | 메모리 시스템 | CLAUDE.md |
   | 8 | 간결성 (lead with the outcome) | 출력 형식 | CLAUDE.md |
   | 9 | 최종 요약 가독성 (re-grounding) | 긴 세션 보고 | CLAUDE.md |
   | 10 | 조기 중단 방지 (operating autonomously) | 자율 파이프라인 전용 | 하네스 리마인더 |
   | 11 | 메모리 부트스트랩 (reflect on sessions) | 1회 실행 | 사용자 메시지 |

2. 인자 처리 ($ARGUMENTS):
   - 비어 있으면: 위 표를 보여주고 "어떤 스니펫을 설치할까요? (번호, '전부', 또는 키워드)"를
     묻고 턴을 끝내라
   - "전부": 1–9번을 설치 (10·11은 CLAUDE.md 대상이 아니므로 안내만)
   - 번호/키워드: 해당 항목만

3. 설치 규칙:
   - 프로젝트 루트 `CLAUDE.md`에 `## Fable 5 작업 지침` 섹션을 만들거나 찾아, 그 아래에
     스니펫을 영어 원문 그대로 코드 블록 없이 본문으로 추가한다 (CLAUDE.md는 지시문
     자체가 프롬프트이므로 코드 블록으로 감싸면 안 된다)
   - **이미 같은 내용이 있으면 중복 설치하지 않는다** — 존재 사실만 보고
   - CLAUDE.md가 없으면 새로 만들되, 프로젝트 설명 한 줄을 함께 채워 넣으라고 안내
   - 10번(조기 중단 방지)·11번(부트스트랩)을 요청받으면 CLAUDE.md가 아니라 각각 하네스
     시스템 리마인더 / 1회성 사용자 메시지로 쓰라고 안내하고 원문만 출력한다

4. 완료 보고: 설치한 스니펫 목록 + CLAUDE.md의 해당 섹션 diff 요약. 관련 심화 내용은
   FableIt 가이드(/guide/agentic/)를 안내.
