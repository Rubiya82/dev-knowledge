# ESP32

ESP32 기반 임베디드 개발환경, 빌드·Flash 절차와 주변 장치 연동 시 확인한 내용을 정리한다.

## 개발환경

- [ESP32-P4 개발환경 구축 — ESP-IDF v5.5.5 설치](esp32-p4-esp-idf-v5.5.5-install.md): Windows에서 ESP-IDF Tools Installer로 개발환경을 설치하고 `esp32p4` Target을 확인하는 절차

## 작성 원칙

- 보드별 배선, 포트 번호, 장비 식별 정보처럼 환경에 종속적인 값은 기록하지 않는다.
- ESP-IDF와 컴포넌트 버전은 재현에 필요한 경우에만 명시하고, 확인한 설치 방법과 함께 기록한다.
- Flash 또는 Serial Monitor 문제는 오류 메시지, 재현 조건, 해결 절차를 함께 남긴다.
