# Authority Manifest

- schema_version: `2.0`
- status: authoritative
- authority_scope: 운영 규칙의 권위 순서와 기능별 소유권 색인
- integration_rule: 이 파일은 권위 순서와 책임 경계만 색인하며 라우팅, 출력, 상태의 세부 동작을 재정의하지 않는다.

## 권위 순서

1. ChatGPT 플랫폼의 시스템·안전·도구 제한
2. 사용자의 현재 명시 요청
3. 활성 독점 모드의 출력 계약
4. 라우팅으로 활성화된 기능별 authoritative 모듈의 세부 계약
5. `00_PROJECT_INSTRUCTIONS.md`의 통합 라우팅과 공통 기본값
6. 일반 참고 문서
7. `reference_only/` 자료

이 순서에서 manifest는 별도 실행 단계가 아니다. 라우팅은 현재 요청에 맞는 모드와 기능을 선택하고, 활성화 이후에는 해당 독점 출력 계약과 기능 모듈이 세부 실행을 소유한다. 프로젝트 지침은 모듈에 없는 공통 기본값만 보충하며, 상위 규칙을 하위 파일이 덮어쓸 수 없다. 같은 단계의 기능 규칙이 충돌하면 더 좁은 기능 범위의 authoritative 모듈을 적용하고, 해석이 여전히 둘 이상이면 실행하지 말고 제한을 명시한다.

## 기능별 소유권 색인

| 기능 | 최종 권위 파일 |
|---|---|
| 권위 순서·소유권 색인 | `knowledge_authoritative/00_authority_manifest.md` |
| `.cc` HTML 출력 계약 | `knowledge_authoritative/01_cc_canvas.md` |
| 저장·복원 스키마와 무진행 복원 | `knowledge_authoritative/02_save_load.md` |
| 이미지 모드 | `knowledge_authoritative/03_image_mode.md` |
| 역량 진단 | `knowledge_authoritative/04_diagnostic.md` |
| 심층 학습 방법 | `knowledge_authoritative/05_deep_learning_method.md` |
| 프로젝트/스레드 상태 격리 | `knowledge_authoritative/06_thread_state_isolation.md` |
| 도구·소스·인용 정책 | `knowledge_authoritative/07_tool_and_source_policy.md` |

`00_PROJECT_INSTRUCTIONS.md`는 항상 필요한 트리거, 금지 사항, 통합 라우팅과 모드 간 출력 우선순위를 제공한다. 기능별 긴 스키마와 셸 계약은 라우팅으로 해당 기능이 활성화된 뒤 위 모듈이 최종 권위다. 기능 모듈은 다른 모드보다 자신을 우선시키거나 통합 우선순위를 다시 정의하지 않는다. Knowledge는 플랫폼 권한을 만들거나 시스템·안전 제한을 바꾸지 않는다.

## 책임 경계

`00_PROJECT_INSTRUCTIONS.md`는 공통 역할, 요청 라우팅, 모드 활성화와 모드 간 독점 출력 우선순위를 담당한다. 기능 모듈은 활성화된 기능의 세부 출력과 상태 동작을 담당하며 다른 모드의 전역 우선순위를 다시 정의하지 않는다. 행동 권위로 지정되지 않은 참고 자료는 기능 계약을 덮어쓰지 않는다.
