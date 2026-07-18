# 프로젝트 설정과 실행 환경

## 프로젝트

프로젝트는 채팅, 파일, 프로젝트 지침, 프로젝트 메모리를 한 공간에 둔다. 현재 GPT에서 만든 채팅은 프로젝트로 직접 이동할 수 없으므로 동등성 기준 응답은 새 테스트 채팅에서 다시 생성해 기록한다.

## GitHub

a. ChatGPT 설정에서 Apps 또는 Plugins 영역을 연다.
b. GitHub를 연결한다.
c. GitHub 인증 화면에서 읽을 private repository를 허용한다.
d. 저장소가 새로 생성됐거나 private이면 표시까지 지연될 수 있다.
e. 프로젝트 채팅에서 GitHub 앱을 도구로 선택하거나 GitHub를 명시해 조회한다.
f. 일반 GitHub 앱은 읽기·검색·인용용이다. 코드 push와 PR 생성은 별도 Codex 또는 쓰기 권한 워크플로가 필요하다.

## 프로젝트 파일

운영 프로젝트에는 다음만 올린다.

- `00_PROJECT_INSTRUCTIONS.md`의 내용
- `MANIFEST.json`에 선언된 8개 `authoritative_files`
- 사용자의 실제 학습 자료

다음은 QA 프로젝트와 저장소에만 두며 운영 프로젝트 소스로 올리지 않는다.

- `reference_only/`
- `tests/`와 `tests/fixtures/`
- `baselines/`
- 공개 구버전 프롬프트

기능 규격과 학습 자료를 같은 파일에 섞지 않는다. 운영 설치 전 README의 로컬 정적 점검 명령으로 JSON/JSONL 구문, 파일별 권위 선언, reference_only 경계를 확인한다.

## private 전환

a. GitHub 저장소 visibility는 GitHub 설정에서 별도로 변경한다.
b. ChatGPT 프로젝트도 비공개로 만들고 GitHub 앱에서 필요한 repository만 허용한다.
c. 프롬프트와 Knowledge는 앱 권한을 생성하지 않는다.
d. 토큰, private 파일 원문, 실제 사용자 저장 패킷을 이 번들 또는 fixture에 넣지 않는다.
e. 접근 실패는 권한 없음, 연결 없음, 도구 미지원, 파일 부재로 구분해 기록한다.

## 실행 조건 고정

GPT와 프로젝트 비교 시 다음을 같게 맞춘다.
- 같은 모델 계열과 thinking 설정
- 같은 웹 검색 사용 여부
- 같은 이미지 모드
- 같은 업로드 파일
- 같은 GitHub 권한
- 새 채팅 여부
- 동일한 사용자 입력과 입력 순서
- 같은 언어

## 반복

각 critical test는 세 번 실행한다.
형식이 한 번만 맞고 반복 실행에서 흔들리면 동등하지 않다.
