# Codex 작업 지침

## 저장소 목적과 한계

이 저장소는 현재 관찰 가능한 Ailey & Bailey Teacher for Study GPT의 사용자 체감 기능, 출력 계약, 상태 전이를 ChatGPT 프로젝트에서 재현하고 반복 검증한다. 비공개 GPT의 원본 시스템 프롬프트를 복원·추출했거나 원본과 완전히 같다고 주장하지 않는다.

## 권위 순서

충돌은 다음 순서로 해결한다.

1. ChatGPT 플랫폼의 시스템·안전·도구 제한
2. 사용자의 현재 명시 요청
3. 활성 독점 모드의 출력 계약
4. 기능별 authoritative 모듈 (`knowledge_authoritative/`)
5. 통합 프로젝트 지침 (`00_PROJECT_INSTRUCTIONS.md`)
6. 일반 참고 문서
7. `reference_only/` 자료

`knowledge_authoritative/00_authority_manifest.md`는 권위 순서와 기능별 소유권을 색인한다. `reference_only/`는 연구·비교 자료이며 운영 지침이나 프로젝트 Knowledge로 사용하지 않는다.

## 수정 전에 읽을 파일

1. `README.md`
2. `MANIFEST.json`
3. `knowledge_authoritative/00_authority_manifest.md`
4. `00_PROJECT_INSTRUCTIONS.md`
5. 수정 기능에 해당하는 authoritative 모듈
6. `tests/EQUIVALENCE_TEST_PLAN.md`와 `tests/cases/equivalence_cases.jsonl`

## 작업 원칙

- 먼저 `git status --short`로 사용자 변경을 확인한다.
- 기능 수정은 작고 국소적으로 하며 관계없는 코드나 문서를 재구성하지 않는다.
- 기존 동작과 하위 호환성을 보존한다. 계약을 바꿀 때는 schema/bundle 버전과 변경 기록을 함께 갱신한다.
- 삭제나 대규모 이동 전에는 Manifest, README, docs, tests의 참조 경로를 확인한다.
- 같은 규칙을 여러 파일에 다시 정의하지 말고 최종 권위 파일을 참조한다.
- 실제로 실행하지 않은 도구나 검증을 실행했다고 보고하지 않는다.
- 비밀, 토큰, 개인 정보, private 저장소 내용, 실제 사용자 저장 패킷을 커밋하지 않는다.

## 문서와 파일명

- 운영 지침과 Knowledge는 Markdown, Manifest와 상태 계약은 JSON, 기계 테스트는 JSONL을 사용한다.
- authoritative 파일은 `NN_lower_snake_case.md`, 검증기는 `validate_<contract>.py`, fixture는 `<validator>_valid.*`와 `<validator>_invalid.*` 형식을 따른다.
- 문서는 UTF-8, JSON/JSONL은 유효한 UTF-8 JSON으로 유지한다.
- normative 문장은 `MUST`, `MUST NOT`, `SHOULD` 또는 명확한 한국어 의무 표현으로 모호하지 않게 쓴다.

## 검증 명령

이번 번들은 일회성 정적 감사를 우선하며 별도 Python 검증기를 운영 계약으로 두지 않는다. 다음 명령으로 구조와 문구를 직접 확인한다.

```powershell
Get-Content MANIFEST.json -Raw | ConvertFrom-Json | Out-Null
Get-Content tests/cases/equivalence_cases.jsonl | ForEach-Object { $_ | ConvertFrom-Json | Out-Null }
rg -n "status: authoritative|authority_scope:|authority_parent:|integration_rule:" knowledge_authoritative
rg -n "reference_only|최종 권위|출력 우선순위" README.md AGENTS.md 00_PROJECT_INSTRUCTIONS.md docs knowledge_authoritative
git diff --check
```

수동 동등성 테스트는 `tests/EQUIVALENCE_TEST_PLAN.md`, Critical Gate는 `tests/CRITICAL_CHECKLIST.md`를 따른다.

## 완료 보고

최종 보고에는 구현 요약, 주요 설계 결정, 변경·추가·삭제 파일, 실제 실행한 명령과 결과, 미해결 플랫폼 의존성, 수동 확인 항목, PR 제목과 설명 초안을 포함한다. 커밋이나 push는 사용자가 명시적으로 요청한 경우에만 수행한다.
