# Ailey & Bailey 프로젝트 동등성 번들

## 목적

이 번들은 현재 대화 중인 Ailey & Bailey GPT의 외부 동작을 ChatGPT 프로젝트에서 최대한 재현하고, 같은 입력에 대한 기능 동등성을 반복 측정하기 위한 구성이다.

중요한 한계:
- 커스텀 GPT의 비공개 시스템·개발자 지침을 직접 추출한 것이 아니다.
- 업로드된 공개·사용자 제공 프롬프트와 현재 GPT에서 관찰 가능한 기능을 바탕으로 만든 기능 동등성 사양이다.
- 앱, Skills, 모델 선택, Canvas 실행 표면, GitHub 권한은 프롬프트만으로 복제되지 않는다.
- 따라서 목표는 문자열 동일성이 아니라 사용자 체감 기능과 상태 전이의 동등성이다.

## 라이선스와 출처

이 번들은 다음 두 공개 upstream을 실제 파일 근거로 검토해 변형한 비영리 동등성·QA 자료를 포함한다.

- `lemos999/ailey-bailey-canvas`: Ailey·Bailey·Codex·Canvas 페르소나와 프롬프트 모듈 구조, 학습 흐름과 비교 파일
- `lemos999/Singulari-Tea-Codex-Canvas`: Stable 프롬프트와 `.cc` 외부 bundle의 실행 자산

두 저장소 모두 `fewweekslater (Ray You)`가 CC BY-NC-SA 4.0으로 공개했으며, 검토한 커밋·파일·blob 근거와 로컬 연결은 [`docs/REFERENCE_PROVENANCE.md`](docs/REFERENCE_PROVENANCE.md)에 분리 기록한다. 변형 자료는 저작자 표시, 변경 사실 표시, 비영리 사용, 동일조건 공유가 필요하다.

- 라이선스: [`LICENSE`](LICENSE)
- 저작자·변형 고지: [`NOTICE.md`](NOTICE.md)
- 검토 커밋·원본 경로·체크섬: [`docs/REFERENCE_PROVENANCE.md`](docs/REFERENCE_PROVENANCE.md)

이 저장소는 비공개 GPT 원본 프롬프트를 확보·복원했다고 주장하지 않는다.

## 폴더

- `00_PROJECT_INSTRUCTIONS.md`: 프로젝트 설정의 Project instructions에 붙여 넣을 지침
- `knowledge_authoritative/`: 운영 프로젝트에 업로드할 기능별 최종 권위 Knowledge
- `reference_only/`: 공개 배포본과 이전 버전. QA 비교 전용이며 운영에 업로드하지 않음
- `tests/EQUIVALENCE_TEST_PLAN.md`: 전체 테스트 절차와 테스트 케이스
- `tests/cases/equivalence_cases.jsonl`: 기존 77개와 V2 추가 사례의 기계 판독 카탈로그
- `tests/SCORING_RUBRIC.md`: 점수와 필수 통과 조건
- `tests/RUN_LOG.csv`: GPT 기준과 프로젝트 결과 기록
- `tests/fixtures/`: source-bounded, 저장 복원, 범위 제한 테스트 자료
- `baselines/`: 실제 응답 캡처용 빈 템플릿과 수동 승인 계약
- `docs/PROJECT_SETUP.md`: 프로젝트와 GitHub 연결 설정
- `docs/FEATURE_MATRIX.md`: 기능별 구현 위치
- `docs/V2_STATIC_AUDIT.md`: V2 일회성 정적 감사 명령과 결과

## 설치 순서

a. 새 비공개 프로젝트를 만든다.
b. `00_PROJECT_INSTRUCTIONS.md` 내용을 프로젝트 지침에 붙인다.
c. `MANIFEST.json`의 `authoritative_files` 8개를 프로젝트 Knowledge로 업로드한다.
d. 사용자의 실제 학습 자료를 별도로 업로드한다.
e. `reference_only`, `tests`, `fixtures`, `baselines`, 공개 구버전 프롬프트는 QA 프로젝트에만 둔다.
f. 필요한 경우 GitHub 앱을 연결하고 private repository를 명시적으로 허용한다.
g. 새 QA 프로젝트와 현재 GPT의 새 채팅에서 같은 테스트를 실행해 실제 baseline을 만든다.
h. 결과를 RUN_LOG.csv와 baseline 템플릿에 기록하고 사람이 승인한다.

## 권장 업로드

필수:
- 00_authority_manifest.md
- 01_cc_canvas.md
- 02_save_load.md
- 03_image_mode.md
- 04_diagnostic.md
- 05_deep_learning_method.md
- 06_thread_state_isolation.md
- 07_tool_and_source_policy.md

학습 목적에 따라 실제 교재, 논문, 저장소 링크를 추가할 수 있다. 공개 배포 프롬프트와 `reference_only`는 운영 프로젝트에 추가하지 않는다.

## 로컬 정적 점검

이번 버전은 재사용할 Python 검증기보다 Markdown 권위 선언과 계약의 직접 감사를 우선한다. 네트워크 없이 다음을 실행한다.

```powershell
Get-Content MANIFEST.json -Raw | ConvertFrom-Json | Out-Null
Get-Content tests/cases/equivalence_cases.jsonl | ForEach-Object { $_ | ConvertFrom-Json | Out-Null }
rg -n "status: authoritative|authority_scope:|authority_parent:|integration_rule:" knowledge_authoritative
rg -n "reference_only|최종 권위|출력 우선순위" README.md AGENTS.md 00_PROJECT_INSTRUCTIONS.md docs knowledge_authoritative
git diff --check
```

위 명령은 JSON/JSONL 구문과 Markdown 선언을 확인한다. 기능 동등성은 `tests/EQUIVALENCE_TEST_PLAN.md`를 실제 GPT와 프로젝트에서 실행하고 baseline을 사람이 승인해야 판정할 수 있다.

## private 전환과 권한

저장소나 학습 프로젝트를 private로 전환해도 프롬프트가 GitHub·웹·이미지·Canvas 권한을 만들지는 않는다. ChatGPT 설정에서 앱을 연결하고 필요한 저장소만 명시적으로 허용한다. private 자료, 토큰, 사용자 저장 패킷을 이 public 번들에 복사하지 않는다. 접근 실패는 권한 없음, 연결 없음, 도구 미지원, 파일 부재로 구분해 기록한다.

실제 GPT 응답이 없는 테스트는 자동 합격하지 않는다. `baselines/README.md`에 따라 `not_evaluated`로 두고 사람이 근거를 승인한다.

## 합격 기준

- Critical Gate 전부 통과
- 총점 90점 이상
- 기능별 하위 점수 80% 이상
- 세 번의 독립 실행에서 치명 실패 0회
- 저장→복원→재저장 상태 드리프트 0
- private GitHub 읽기 테스트 통과 또는 플랫폼 미지원으로 명확히 분류
