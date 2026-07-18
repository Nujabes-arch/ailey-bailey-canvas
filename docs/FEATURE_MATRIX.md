# 기능 매트릭스

| 기능 | 구현 위치 | 핵심 검증 |
|---|---|---|
| 권위와 충돌 해결 | 00_authority_manifest.md | 단일 권위 순서, 기능별 최종 파일 |
| Ailey 기본 페르소나 | 프로젝트 지침 | 따뜻한 반말, 비유, 오류 인정 |
| Bailey 최소 개입 | 프로젝트 지침 | 반복 문제 오류 전에는 등장 금지 |
| 기본 compass | 프로젝트 지침 | 문자 binding과 KST timestamp |
| 강의·분석 라우팅 | 프로젝트 지침 | 목적별 shell과 밀도 |
| .ff | 프로젝트 지침 | 장문 5~7 프레임워크 |
| .cx | 프로젝트 지침 | 백과사전형 한 번의 심층 탐구 |
| .cc | 01_cc_canvas.md | HTML artifact + compass만 |
| 커리큘럼 | 프로젝트 지침 | 첫 leaf, saved-next 보존 |
| 문제·채점·복습 | 프로젝트 지침 | 답 비공개, 오답 1개씩 |
| 진단 | 04_diagnostic.md | 단일 자유응답, 진단 중 무교육 |
| source_bounded | 프로젝트 지침 | 자료 밖 보충 금지 |
| 이미지 모드 | 03_image_mode.md | off 기본, on이면 이미지 선행 |
| 저장·복원 | 02_save_load.md | zero-beat와 상태 드리프트 방지 |
| 심층 연구 방법 | 05_deep_learning_method.md | 경쟁 모델·증거·병목 |
| 웹 최신성 | 프로젝트 지침 + 도구 | 실제 검색과 인용 |
| 파일 분석 | 프로젝트 지침 + 프로젝트 파일 | 자료 근거와 범위 유지 |
| private GitHub | 프로젝트 앱 권한 | 허용 repo 읽기·검색·인용 |
| 프로젝트 다중 스레드 | 06_thread_state_isolation.md | 실행 식별자 격리, 명시적 복원만 허용 |
| 도구·소스·인용 | 07_tool_and_source_policy.md | source_bounded, 실제 도구 사용 정직성 |

## 운영과 QA 경계

운영 프로젝트는 프로젝트 지침, 8개 authoritative Knowledge, 실제 학습 자료만 사용한다. `reference_only`, tests, fixtures, baselines, 공개 구버전 프롬프트는 QA 전용이다. 정적 검사는 README의 직접 감사 명령을 사용하며 실제 GPT 동등성 판정은 baseline과 수동 승인이 있어야 한다.
