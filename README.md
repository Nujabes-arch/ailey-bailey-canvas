# Ailey & Bailey 프로젝트 동등성 번들

## 목적

이 번들은 현재 대화 중인 Ailey & Bailey GPT의 외부 동작을 ChatGPT 프로젝트에서 최대한 재현하고, 같은 입력에 대한 기능 동등성을 반복 측정하기 위한 구성이다.

중요한 한계:
- 커스텀 GPT의 비공개 시스템·개발자 지침을 직접 추출한 것이 아니다.
- 업로드된 공개·사용자 제공 프롬프트와 현재 GPT에서 관찰 가능한 기능을 바탕으로 만든 기능 동등성 사양이다.
- 앱, Skills, 모델 선택, Canvas 실행 표면, GitHub 권한은 프롬프트만으로 복제되지 않는다.
- 따라서 목표는 문자열 동일성이 아니라 사용자 체감 기능과 상태 전이의 동등성이다.

## 폴더

- `00_PROJECT_INSTRUCTIONS.md`: 프로젝트 설정의 Project instructions에 붙여 넣을 지침
- `knowledge_authoritative/`: 프로젝트에 업로드할 기능별 Knowledge
- `reference_only/`: 공개 배포본과 이전 버전. 행동 권위로 사용하지 않음
- `tests/EQUIVALENCE_TEST_PLAN.md`: 전체 테스트 절차와 테스트 케이스
- `tests/SCORING_RUBRIC.md`: 점수와 필수 통과 조건
- `tests/RUN_LOG.csv`: GPT 기준과 프로젝트 결과 기록
- `tests/fixtures/`: source-bounded, 저장 복원, 범위 제한 테스트 자료
- `docs/PROJECT_SETUP.md`: 프로젝트와 GitHub 연결 설정
- `docs/FEATURE_MATRIX.md`: 기능별 구현 위치

## 설치 순서

a. 새 비공개 프로젝트를 만든다.
b. `00_PROJECT_INSTRUCTIONS.md` 내용을 프로젝트 지침에 붙인다.
c. `knowledge_authoritative`의 5개 파일을 프로젝트 소스로 업로드한다.
d. `reference_only` 파일은 비교가 필요할 때만 업로드한다. 업로드한다면 지침의 reference-only 우선순위를 유지한다.
e. GitHub 앱을 연결하고 필요한 private repository를 명시적으로 허용한다.
f. 새 프로젝트 채팅에서 테스트 계획의 Phase 0부터 실행한다.
g. 같은 테스트를 현재 GPT의 새 채팅에서도 실행해 baseline을 만든다.
h. 결과를 RUN_LOG.csv에 기록한다.

## 권장 업로드

필수:
- 01_cc_canvas.md
- 02_save_load.md
- 03_image_mode.md
- 04_diagnostic.md
- 05_deep_learning_method.md

선택:
- 공개 배포 프롬프트는 reference-only로 보관
- 실제 학습 교재, 논문, 저장소 링크는 프로젝트 목적에 따라 추가

## 합격 기준

- Critical Gate 전부 통과
- 총점 90점 이상
- 기능별 하위 점수 80% 이상
- 세 번의 독립 실행에서 치명 실패 0회
- 저장→복원→재저장 상태 드리프트 0
- private GitHub 읽기 테스트 통과 또는 플랫폼 미지원으로 명확히 분류
