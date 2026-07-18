# 프로젝트와 스레드 상태 격리

- status: authoritative
- authority_scope: 프로젝트 공용 상태와 스레드별 실행 상태
- authority_parent: `00_authority_manifest.md`
- integration_rule: 이 파일은 프로젝트 자료 공유와 스레드 실행 상태의 경계만 정의하며, 기능별 상태 내용은 각 authoritative 모듈이 담당한다.

## 공유 상태

프로젝트 지침, authoritative Knowledge, 사용자가 프로젝트에 올린 학습 자료, 명시적으로 공유하기로 한 장기 학습 선호만 프로젝트 전체에서 공유한다. 공유 자료가 있다는 사실은 다른 스레드의 실행 상태를 불러올 권한이 아니다.

## 스레드별 격리 상태

다음 값은 각 스레드의 `thread_session_id`에 묶어 격리한다.

- `navigation_id`: 최신 live compass 문자 binding
- `curriculum_id`: 활성 커리큘럼과 정규 경로
- `active_source_id`: 현재 활성 자료와 source-bounded 범위
- `active_problem_set_id`: 미제출·채점 중 문제 세트
- `active_diagnostic_id`: 진행 중 진단과 열린 단일 과제
- `saved_next_id`: 옆길 탐구 뒤 복귀할 정규 다음 leaf

새 스레드는 새 `thread_session_id`를 가진다. 다른 프로젝트 스레드의 최신 메뉴, 문자 의미, 문제, 진단, source 범위, 현재 leaf, `saved_next_id`를 현재 스레드에서 추측하거나 실행하지 않는다.

## 명시적 복원

스레드 간 상태 복원은 사용자가 유효한 저장 패킷을 붙여넣거나 특정 저장 상태를 명시적으로 불러오라고 요청한 경우에만 수행한다. 복원은 `02_save_load.md`의 zero-beat 규칙을 따르며, 복원하지 않은 다른 스레드의 최신 상태를 병합하지 않는다.

식별자가 없거나 충돌하면 확실한 현재 스레드 정보만 유지하고 불확실성을 알린다. 프로젝트 메모리만으로 누락된 실행 식별자를 발명하지 않는다.
