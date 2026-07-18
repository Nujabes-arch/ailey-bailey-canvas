# Authority Manifest

- schema_version: `2.0`
- status: authoritative
- authority_scope: 이 저장소의 운영 규칙 권위와 충돌 해결
- integration_rule: 이 파일은 권위 순서와 기능별 소유권만 선언하며 각 기능의 세부 동작을 재정의하지 않는다.

## 충돌 해결 순서

1. ChatGPT 플랫폼의 시스템·안전·도구 제한
2. 사용자의 현재 명시 요청
3. 이 권위 manifest의 기능별 소유권 선언
4. `00_PROJECT_INSTRUCTIONS.md`의 통합 라우팅과 독점 출력 우선순위
5. 라우팅으로 활성화된 기능별 authoritative 모듈의 세부 계약
6. 일반 참고 문서
7. `reference_only/` 자료

상위 규칙을 하위 파일이 덮어쓸 수 없다. 같은 단계의 기능 규칙이 충돌하면 더 좁은 기능 범위의 authoritative 모듈을 적용하고, 해석이 여전히 둘 이상이면 실행하지 말고 제한을 명시한다.

## 기능별 최종 권위

| 기능 | 최종 권위 파일 |
|---|---|
| 권위·충돌 해결 | `knowledge_authoritative/00_authority_manifest.md` |
| `.cc` HTML 출력 계약 | `knowledge_authoritative/01_cc_canvas.md` |
| 저장·복원 스키마와 무진행 복원 | `knowledge_authoritative/02_save_load.md` |
| 이미지 모드 | `knowledge_authoritative/03_image_mode.md` |
| 역량 진단 | `knowledge_authoritative/04_diagnostic.md` |
| 심층 학습 방법 | `knowledge_authoritative/05_deep_learning_method.md` |
| 프로젝트/스레드 상태 격리 | `knowledge_authoritative/06_thread_state_isolation.md` |
| 도구·소스·인용 정책 | `knowledge_authoritative/07_tool_and_source_policy.md` |

`00_PROJECT_INSTRUCTIONS.md`는 항상 필요한 트리거, 금지 사항, 통합 라우팅과 모드 간 출력 우선순위를 제공한다. 기능별 긴 스키마와 셸 계약은 라우팅으로 해당 기능이 활성화된 뒤 위 모듈이 최종 권위다. 기능 모듈은 다른 모드보다 자신을 우선시키거나 통합 우선순위를 다시 정의하지 않는다. Knowledge는 플랫폼 권한을 만들거나 시스템·안전 제한을 바꾸지 않는다.

## 구버전과 비교 자료

- `reference_only/Ailey & Bailey X대화형_260323.md`
- `reference_only/Ailey & Bailey_Gemini_260514.md`

위 파일은 연구·비교 전용이며 운영 프로젝트에 업로드하거나 행동 권위로 지정하지 않는다.

## 버전 변경 기록 규칙

- 규칙 의미나 스키마가 바뀌면 `schema_version`을 갱신한다.
- 운영 번들 구성이 바뀌면 `MANIFEST.json`의 `bundle_version`을 갱신한다.
- 변경 이유, 하위 호환성 영향, migration 필요 여부를 PR 설명에 기록한다.
- 기존 authoritative 계약을 조용히 바꾸지 않는다.
