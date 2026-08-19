---
tags:
  - Home
---

# 개발 지식 저장소

## Automation · C++ · MFC · Motion · EtherCAT · Windows

설비 자동화와 Windows 소프트웨어 개발 과정에서 반복해서 사용하는 기술, 설정 절차, 트러블슈팅 및 설계 원칙을 빠르게 검색하고 재사용하기 위한 개인 Knowledge Base입니다.

[:material-robot-industrial: Automation](automation/index.md){ .md-button .md-button--primary }
[:material-code-braces: C++ 문서](cpp/index.md){ .md-button }
[:material-wrench: Troubleshooting](troubleshooting/index.md){ .md-button }

!!! tip "빠르게 찾기"
    상단 검색 아이콘 또는 <kbd>/</kbd> 키로 검색을 열 수 있습니다. 예: `Panasonic A6B`, `EtherCAT`, `MFC DLL`, `BitLocker`.

## 주요 기술 영역

<div class="grid cards" markdown>

-   :material-robot-industrial:{ .lg .middle } **Automation**

    ---

    EtherCAT, Motion, Vision 및 자동화 시스템 개발 지식

    [:octicons-arrow-right-24: 문서 보기](automation/index.md)

-   :material-code-braces:{ .lg .middle } **C++ / MFC**

    ---

    DLL ABI, UI, Threading 등 Windows 네이티브 개발 사례

    [:octicons-arrow-right-24: 문서 보기](cpp/index.md)

-   :material-axis-arrow:{ .lg .middle } **Motion / Servo**

    ---

    좌표계, 상태 전이, 알람 처리 및 모션 제어 절차

    [:octicons-arrow-right-24: 문서 보기](automation/motion/index.md)

-   :material-lan:{ .lg .middle } **EtherCAT**

    ---

    EtherCAT 통신, SOEM 및 CiA 402 제어 프로파일

    [:octicons-arrow-right-24: 문서 보기](automation/ethercat/index.md)

-   :material-microsoft-windows:{ .lg .middle } **Windows / System**

    ---

    BitLocker, VHDX 및 Windows 문제 진단

    [:octicons-arrow-right-24: 문서 보기](windows/index.md)

-   :material-sitemap:{ .lg .middle } **Architecture**

    ---

    모듈 경계, 의존성 및 설계 판단의 배경

    [:octicons-arrow-right-24: 문서 보기](architecture/index.md)

</div>

## 추천 문서

<div class="grid cards" markdown>

-   **Panasonic MINAS A6B Absolute Home Offset 설정**

    Absolute System에서 `607Ch:00h Home offset`을 확인·변경·EEPROM 저장하는 현장 절차와 공식 사양.

    [:octicons-arrow-right-24: 문서 열기](automation/motion/panasonic-a6b-absolute-home-offset.md)

-   **MFC DLL ABI 호환성**

    DLL 경계에서 `CString`, STL 및 Runtime 구성을 안전하게 다루기 위한 설계 원칙.

    [:octicons-arrow-right-24: 문서 열기](cpp/mfc/dll/abi-compatibility.md)

-   **SOEM**

    EtherCAT Master 구현 시 네트워크 인터페이스, 상태 전이, 오류 복구를 점검하는 출발점.

    [:octicons-arrow-right-24: 문서 열기](automation/ethercat/soem.md)

-   **CiA 402**

    드라이브 상태 전이, 제어·상태 워드와 PDO 매핑을 확인하기 위한 요약.

    [:octicons-arrow-right-24: 문서 열기](automation/ethercat/cia402.md)

</div>

## 빠른 바로가기

<div class="grid cards" markdown>

-   :material-alert-circle-outline:{ .lg .middle } **Troubleshooting**

    증상, 재현 조건, 원인 및 검증 결과 중심의 문제 해결 기록.

    [:octicons-arrow-right-24: 열기](troubleshooting/index.md)

-   :octicons-mark-github-16:{ .lg .middle } **GitHub Repository**

    Markdown 원본과 변경 이력을 확인합니다.

    [:octicons-arrow-up-right-24: 저장소 열기](https://github.com/Rubiya82/dev-knowledge){ target="_blank" rel="noopener" }

</div>

## 이 저장소 사용 방법

1. 먼저 검색으로 기존 문서를 찾습니다.
2. 카테고리 문서에서 관련 링크와 세부 문서로 이동합니다.
3. 새 지식은 중복 여부를 확인한 뒤 적절한 카테고리에 추가하고, 근거·테스트 환경·주의사항을 함께 기록합니다.

공개 가능한 일반 기술 사례만 다루며, 회사·고객·프로젝트·계정 등 민감정보는 기록하지 않습니다.
