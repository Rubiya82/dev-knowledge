# Architecture

소프트웨어 구조, 모듈 경계, 의존성 관리 및 설계 원칙을 기록합니다.
특정 구현보다 판단의 배경과 트레이드오프를 남기는 데 집중합니다.

## 문서

- [Mechanical · Control · Process 3계층 분리](mechanical-control-process-layers.md): 자동화 장비의 계층 책임, 알람 경계 및 모션 포함 공급 동작 패턴
- [Process Step 상태 기계의 switch 유지보수성 개선](process-step-state-machine-maintainability.md): 큰 `doRunStep()`을 안전하게 분해해 정적 분석 복잡도와 변경 범위를 낮추는 기준
