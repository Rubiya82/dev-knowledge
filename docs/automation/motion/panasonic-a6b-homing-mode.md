---
tags:
  - Automation
  - Motion
  - Panasonic
  - MINAS
  - A6B
  - Servo
  - EtherCAT
  - CiA 402
  - Homing
---

# Panasonic MINAS A6B Homing Mode (hm) 사용 방법

## 개요

MINAS A6B의 `hm`(Homing mode)은 EtherCAT 드라이브가 Homing method, 속도, 가속도를 바탕으로 원점 복귀 위치를 탐색하는 위치 제어 모드다. `607Ch:00h Home offset`을 현재 위치 기준으로 다시 계산하는 PANATERM `Set Home` 기능과 다르다.

이 문서는 Panasonic 공식 A6B EtherCAT 사양의 객체와 상태 비트를 기준으로 정리한다. 실제 센서 배선, 회전 방향, 안전회로 및 Homing method는 장비별로 검증해야 한다.

!!! warning "축이 실제로 움직입니다"
    hm 시작은 기계적 원점 탐색 동작이다. 이동 범위, Home/Limit 입력, 충돌 위험, 비상정지 및 인터록을 확인하고 저속으로 시험한다. 검증 전에는 운전 중인 설비에서 실행하지 않는다.

## 핵심 객체

| 객체 | 역할 |
| --- | --- |
| `6060h:00h` | Modes of operation. `6`을 쓰면 hm mode를 요청한다. |
| `6061h:00h` | Modes of operation display. 실제 hm mode 전환을 확인한다. |
| `6040h:00h bit 4` | Start homing. `0 → 1` 상승 에지로 Homing을 시작한다. |
| `6041h:00h bit 12` | Homing attained. 성공 완료 상태 확인에 사용한다. |
| `6041h:00h bit 13` | Homing error. Homing 오류 상태 확인에 사용한다. |
| `6098h:00h` | Homing method. 센서·방향·index pulse 사용 방식을 지정한다. |
| `6099h:01h`, `6099h:02h` | Homing speeds. 탐색 및 index-pulse 탐색 속도다. |
| `609Ah:00h` | Homing acceleration/deceleration이다. |
| `607Ch:00h` | Homing 완료 후 감지 위치에 적용되는 Home offset이다. |

Panasonic A6B는 `6060h=6`의 hm mode를 지원하며, `6098h`에는 지원되는 방법만 설정해야 한다. 지원하지 않는 method로 Homing을 시작하면 `6041h bit 13` Homing error가 발생한다. [SX-DSV03242 R15.0, §6-5-2 및 §6-6-5, pp. 89, 142–148](https://mediap.industry.panasonic.eu/assets/custom-upload/Factory%20%26%20Automation/Industrial%20Motors/Manuals/mn_minas_a6b_ethercat_communication_specification_pid_en.pdf)

## PANATERM을 이용한 사전 설정과 확인

PANATERM Ver.6.0은 A6B를 USB로 지원한다. Object Editor에서는 현재 객체를 읽고 설정값을 확인·편집할 수 있다. 통신 연결 상태에서 PANATERM 매뉴얼은 `ESM Condition: INIT`일 때 driver object의 편집·전송이 가능하다고 설명한다.

1. USB로 A6B와 PC를 연결하고 PANATERM에서 대상 드라이버와 통신한다.
2. **기타 → 객체 편집기(Object Editor)** 를 열고 `ESM Condition`을 확인한다.
3. `6098h`, `6099h`, `609Ah`, `607Ch`의 현재값과 현장 설계값을 확인한다.
4. 변경이 필요하면 `INIT`에서 객체값을 변경하고, 유지가 필요하면 **EEPROM**으로 백업한다.
5. PANATERM은 설정·모니터링 용도로 사용하고, EtherCAT 운전 중 hm 시작 명령은 master의 `6060h`/`6040h` 제어 흐름으로 수행한다.

PANATERM Object Editor 및 ESM 편집 조건은 [PANATERM Ver.6.0 Operation Manual Rev. 3.15, pp. 175–176](https://mediap.industry.panasonic.eu/assets/download-files/import/mn_minas_a6_panaterm_operation_pidsx_en.pdf)를 따른다.

## hm 실행 절차

### 1. Homing method와 입력 조건을 결정한다

`6098h`의 method는 실제 입력과 일치해야 한다. 예를 들어 method 1/2는 negative/positive limit switch와 index pulse를, method 3–6은 Home switch와 index pulse를 사용한다. 일부 method는 index pulse 대신 limit 또는 Home switch 변화 위치를 사용한다. A6B가 실제로 지원하는 목록은 `60E3h`에서 읽을 수 있다.

HOME, POT, NOT 입력이 method의 요구와 다르면 Homing error가 발생할 수 있다. method 중 하나를 임의로 기본값으로 선택하지 말고, 센서 배선과 기계 방향에 맞춰 사양의 동작 예를 확인한다.

### 2. Homing 속도·가속도와 Home offset을 설정한다

- `6099h:01h`, `6099h:02h`: Homing 구간의 속도를 설정한다.
- `609Ah:00h`: Homing 중 가속·감속을 설정한다.
- `607Ch:00h`: Homing 완료 뒤 감지된 기준 위치에 부여할 좌표값을 설정한다.

Homing method가 시작되면 관련 method·속도·가속도·감속도 값이 저장되어 그 동작에 사용된다. Homing 진행 중에는 `6098h`를 변경할 수 없으므로, 변경하려면 먼저 축을 정지하고 hm 동작을 끝낸다.

### 3. hm mode로 전환한다

EtherCAT master에서 `6060h:00h = 6`을 설정하고 `6061h:00h`가 `6`으로 표시되는지 확인한다. A6B 사양상 operation mode의 기본값은 `0`이므로 control power ON 뒤에도 필요한 mode를 명시적으로 설정한다.

### 4. Controlword로 Homing을 시작한다

축이 안전하게 Enable 가능한 PDS 상태인지 확인한 뒤, master가 `6040h bit 4`를 `0`에서 `1`로 전환한다. 이 상승 에지가 start homing이다. 이미 Homing 중일 때 bit 4를 다시 상승시켜도 새 Homing 요청은 무시된다.

### 5. 완료 또는 오류를 확인한다

`6041h`의 Homing 상태 비트를 확인한다.

| bit 13: Homing error | bit 12: Homing attained | 의미 |
| --- | --- | --- |
| `0` | `0` | Homing 진행 중 또는 아직 완료 전 |
| `0` | `1` | Homing 성공 완료 |
| `1` | `0` | 오류가 감지됐지만 동작 중 |
| `1` | `1` | 오류가 감지돼 정지 |

성공 후 `6064h Position actual value`와 `607Ch`를 읽어 목표 좌표가 맞는지 확인한다. Homing을 하면 위치 정보가 reset되므로, 이전 좌표계에서 취득한 touch-probe 등의 데이터는 다시 취득해야 한다.

## Absolute System에서의 주의점

A6B Absolute mode에서는 전원 ON 때 `Homing attained`가 1일 수 있지만, hm 동작을 시작하면 해당 비트가 0이 되고 Homing 결과에 따라 다시 상태가 정해진다. Absolute encoder의 multi-turn clear는 별도 기능이며, hm mode에서 clear가 완료되면 bit 12가 다시 1이 된다. Home offset 설정, multi-turn clear, battery 교체 절차를 하나의 작업으로 취급하지 않는다.

## 공식 자료

- Panasonic Industry, *Technical Reference – EtherCAT Communication Specification – For MINAS A6B series*, **SX-DSV03242 R15.0**, 2025-01-31, §6-5-2, §6-6-5 (pp. 89, 137, 142–175).
- Panasonic Industry, *PANATERM Ver.6.0 Operation Manual*, **Rev. 3.15**, Object Editor 및 ESM Condition (pp. 175–176).
