# Baseline 캡처

이 디렉터리는 사람이 실제 현재 GPT와 프로젝트에서 실행해 얻은 응답과 상태 전이를 저장하기 위한 빈 템플릿만 제공한다. 저장소가 응답을 임의로 생성하거나 원본 GPT의 결과처럼 꾸미지 않는다.

## 사용 순서

1. 동일한 setup과 prompt를 현재 GPT와 프로젝트의 새 스레드에서 실행한다.
2. 원문을 각각의 템플릿 복사본에 그대로 붙이고 실행 시각·모델·도구 조건을 기록한다.
3. 민감 정보와 실제 사용자 저장 패킷은 커밋하지 않는다. 저장 계약 검증에는 합성 fixture만 사용한다.
4. 상태 전이는 `state_transition_template.json` 복사본에 기록한다.
5. 검토자가 근거를 확인한 뒤 `manual_approval.status`를 `approved` 또는 `rejected`로 바꾼다.

baseline이 없거나 `manual_approval.status`가 `pending`이면 해당 비교는 `not_evaluated`다. 자동 합격, 2점, PASS로 처리하지 않는다.

권장 실제 캡처 파일명은 `<test-id>_<surface>_<YYYYMMDD-HHMMSS>.md`다. 공개 저장소에는 비식별·공개 가능한 캡처만 추가한다.
