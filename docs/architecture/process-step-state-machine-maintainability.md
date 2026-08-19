---
tags:
  - Architecture
  - Automation
  - C++
---

# Process Step 상태 기계의 switch 유지보수성 개선

자동화 Process Layer에서 `doRunStep()`은 공정 상태를 분기하고 다음 Step으로 전이하는 디스패처다. Step 수가 늘어날수록 하나의 `switch`가 수백~수천 줄로 커지기 쉽고, 정적 분석의 메서드 길이·복잡도·응집도 지표를 악화시킨다. 이 문서는 설비 안전 동작을 바꾸지 않으면서 큰 `switch`를 단계적으로 분해하는 기준을 정리한다.

## 문제를 판단하는 신호

다음이 함께 나타나면 단순 서식 문제가 아니라 구조 개선 대상으로 본다.

- 하나의 `doRunStep()`이 수백 줄을 넘거나 `case`가 10개 이상이다.
- 각 `case`에 모션, IO, 진공, 센서 확인, 타 공정 핸드셰이크, Alarm 처리가 섞여 있다.
- 같은 실패 처리와 `setStep()` 패턴이 여러 `case`에 반복된다.
- 특정 공정의 변경이 큰 `switch` 전체를 다시 읽고 수정하게 만든다.
- `case` 하나가 여러 상태 전이를 내포해, 현재 상태와 다음 상태를 한눈에 파악하기 어렵다.

실제 사례를 일반화해 측정했을 때 Process `doRunStep()`은 약 300~2,000줄, 11~35개 `case`, 12~43회의 Step 전이를 포함했다. 점수 개선의 목표는 줄 수만 줄이는 것이 아니라, **각 상태의 책임과 변경 범위를 좁히는 것**이다.

## 원인

큰 `switch`는 보통 다음 책임이 한 메서드에 누적되어 생긴다.

| 책임 | Process에 남길 내용 | 분리할 대상 |
| --- | --- | --- |
| 상태 전이 | 현재 Step 선택, 다음 Step 결정, 운전 모드·재시도 정책 | - |
| 장치 동작 | 어떤 공정 명령을 언제 실행할지 | Control의 의미 있는 복합 명령 |
| 물리 세부 | 축 순서, 안전 위치, IO·진공·센서 조합 | Mechanical 또는 Control |
| 오류 처리 | 오류를 Alarm으로 승격하고 안전한 전이 결정 | 공통 실패 처리 보조 함수 |
| 공정 간 동기화 | Ready/Done/Unlock 계약 확인 | 명명된 동기화 보조 함수 또는 ITI API |

계층 책임 자체는 [Mechanical · Control · Process 3계층 분리](mechanical-control-process-layers.md)의 기준을 따른다. Process가 물리 동작을 직접 구현할수록 `case`가 길어지고, 장비 변경이 공정 상태 기계 전체에 전파된다.

## 권장 구조

`doRunStep()`은 상태 선택만 담당하고, 각 상태의 본문은 짧은 private handler로 옮긴다.

```cpp
void MProcess::doRunStep()
{
	switch (m_estepCurrent)
	{
	case STEP_INIT:
	{
		runInitStep();
		break;
	}

	case STEP_WAIT_PART:
	{
		runWaitPartStep();
		break;
	}

	case STEP_PROCESS_PART:
	{
		runProcessPartStep();
		break;
	}

	default:
	{
		handleInvalidStep();
		break;
	}
	} // End of Switch
}
```

각 handler는 한 Step의 정책만 표현한다. 하드웨어 세부 순서를 새 handler로 그대로 복사하지 말고, 이미 존재하거나 새로 정의한 Control 명령으로 위임한다.

```cpp
void MProcess::runProcessPartStep()
{
	const int iResult = m_pControl->ProcessPart();
	if (iResult != ERR_SUCCESS)
	{
		handleStepFailure(iResult, STEP_ERROR);
		return;
	}

	setStep(STEP_WAIT_PROCESS_COMPLETE);
}
```

## 안전한 분해 순서

1. **현재 동작을 고정한다.** Step enum, 전이표, 각 실패 경로와 Alarm 코드를 기록한다. 정상·오류·재기동 시퀀스를 기준 동작으로 삼는다.
2. **동작을 바꾸지 않고 handler만 추출한다.** `case` 본문을 private 메서드로 이동하고 `doRunStep()`의 상태 선택 구조와 `setStep()` 호출 순서를 유지한다.
3. **반복 실패 처리를 공통화한다.** 로그, Alarm 승격, 안전한 오류 Step 전이가 동일할 때만 `handleStepFailure()`로 모은다. Alarm 코드와 실제 정지/복구 정책은 그대로 유지한다.
4. **장치 세부를 Control/Mechanical로 이동한다.** 모션·IO·진공·센서의 세부 순서는 Process handler에서 제거하고, 성공/실패 계약이 명확한 의미 단위 API로 감싼다.
5. **논리적인 상태 묶음으로 재구성한다.** Init, 자재 대기, Pick, 검사, 조립, 배출, Recovery처럼 공정 경계가 명확할 때에만 상위 dispatcher 또는 하위 상태 기계로 나눈다.

한 번에 enum 이름, 전이 정책, 장치 API, 동작 순서를 모두 바꾸지 않는다. 첫 변경은 추출 리팩터링으로 제한해야 설비 동작 회귀를 추적할 수 있다.

## handler 설계 규칙

- `runXxxStep()`은 현재 상태 하나를 처리하고, 성공·대기·실패 중 어떤 결과로 끝나는지 명확히 한다.
- Step 전이는 한 handler의 끝 또는 명확한 분기 지점에 둔다. 보이지 않는 부수 효과로 현재 Step을 변경하지 않는다.
- 대기 상태에서는 반복 호출되어도 출력이 중복 실행되지 않도록 완료 조건과 발행 조건을 분리한다.
- 공정 간 핸드셰이크는 `isPeerReady()`, `notifyPeerDone()`처럼 의도를 드러내는 API로 표현한다.
- `default`는 무시하지 말고 비정상 Step 로그, Alarm, 안전한 Recovery 또는 Stop 정책으로 처리한다.
- 기존 MFC/C++ 코드베이스의 명명 규칙과 Unicode 문자열 규칙을 유지한다.

## 피해야 할 방식

- 큰 `switch`를 여러 파일로 기계적으로 잘라 상태 전이의 소유자가 불분명해지는 방식
- 모든 `case`를 함수로 옮긴 뒤에도 각 함수가 축 이동·IO·Alarm을 직접 모두 처리하는 방식
- 공통 handler에서 서로 다른 Alarm 코드나 복구 Step을 하나로 뭉개는 방식
- 안전 확인을 "중복"으로 판단해 삭제하는 방식
- 정적 분석 점수만 맞추기 위해 의미 없는 wrapper를 대량 추가하는 방식

특히 안전 PLC, 하드웨어 인터록, Emergency Stop에 따른 즉시 정지 책임은 리팩터링 중에도 보존해야 한다. 코드 구조 개선은 물리 안전 기능을 대체하지 않는다.

## 검증 체크리스트

- [ ] 모든 기존 Step이 정확히 하나의 handler 또는 명시적 전이표에 대응한다.
- [ ] 정상, 타임아웃, 장치 오류, 재시작 경로의 다음 Step과 Alarm 코드가 변경 전과 같다.
- [ ] 모션·IO 호출의 순서와 완료 대기 조건이 변경되지 않았다.
- [ ] 반복 호출되는 대기 Step에서 출력이 중복 실행되지 않는다.
- [ ] 비정상 Step과 Control 오류가 안전한 정지 또는 Recovery 경로로 간다.
- [ ] `doRunStep()`은 상태 선택과 전이 정책을 읽을 수 있는 크기로 줄었고, handler는 단일 Step 책임을 가진다.
- [ ] 기존 자동 운전, 수동 운전, Dry-run/Test, 복구 시나리오를 설비 안전 절차에 따라 검증했다.

## 결과

이 구조는 `switch`의 메서드 길이와 분기 복잡도를 낮추는 동시에, 상태 전이·장치 동작·오류 정책의 변경 위치를 분리한다. 정적 분석 지표는 이를 따른 결과이며, 리팩터링의 완료 기준은 기존 안전 동작과 공정 시퀀스가 보존되는 것이다.
