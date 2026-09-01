# ESP32-P4 개발환경 구축 --- ESP-IDF v5.5.5 설치

## 1. 목적

ESP32-P4 기반 디스플레이 개발 보드에서 산업용 HMI를 개발하기 위한
Windows 개발환경을 구축한다.

향후 개발 환경은 다음 구성을 기준으로 한다.

-   Windows 11
-   Git
-   Visual Studio Code
-   ESP-IDF
-   C/C++
-   LVGL
-   Codex
-   ESP32-P4

> 이 문서는 **ESP-IDF Tools Installer를 이용한 ESP-IDF v5.5.5 설치
> 과정**을 기록한다.

------------------------------------------------------------------------

## 2. 권장 설치 경로

ESP-IDF 관련 파일은 다음 경로를 사용한다.

``` text
C:\Espressif
```

ESP-IDF Framework는 다음 위치에 설치한다.

``` text
C:\Espressif\frameworks\esp-idf-v5.5.5
```

설치 완료 후 예상되는 주요 디렉터리 구조는 다음과 같다.

``` text
C:\Espressif
├─ dist
├─ frameworks
│  └─ esp-idf-v5.5.5
├─ python_env
└─ tools
```

프로젝트 소스는 ESP-IDF 설치 폴더와 분리하여 관리하는 것을 권장한다.

예:

``` text
C:\Work\ESP32
```

------------------------------------------------------------------------

## 3. ESP-IDF Tools Installer 실행

ESP-IDF Tools Installer를 실행한다.

본 설치에서는 다음 버전을 사용하였다.

``` text
ESP-IDF Tools 2.4.1
```

------------------------------------------------------------------------

## 4. ESP-IDF 다운로드 방식 선택

`Download or use ESP-IDF` 화면에서 다음 항목을 선택한다.

``` text
Download ESP-IDF
```

기존 ESP-IDF 디렉터리를 사용하는 `Use an existing ESP-IDF directory`는
선택하지 않는다.

`Next`를 선택한다.

------------------------------------------------------------------------

## 5. ESP-IDF 버전 선택

`Version of ESP-IDF` 화면에서 다음 버전을 선택한다.

``` text
v5.5.5 (release version - zip archive download)
```

설치 디렉터리는 다음과 같이 지정한다.

``` text
C:\Espressif\frameworks\esp-idf-v5.5.5
```

### Zip release를 선택하는 이유

일반적인 개발환경 구축에서는 release branch를 직접 Git Clone하는 것보다
공식 release zip을 이용하는 편이 간단하다.

다음 항목은 선택하지 않는다.

``` text
release/v5.5 (release branch - git clone)
```

`Next`를 선택한다.

------------------------------------------------------------------------

## 6. ESP-IDF Tools 설치 위치

`Select Destination Location` 화면에서 ESP-IDF Tools의 설치 경로를
지정한다.

``` text
C:\Espressif
```

기본값이 위 경로라면 변경하지 않고 `Next`를 선택한다.

------------------------------------------------------------------------

## 7. 설치 Component 선택

`Select Components` 화면에서 상단 설치 유형을 다음과 같이 유지한다.

``` text
Full installation
```

기본적으로 필요한 항목을 설치한다.

주요 항목:

-   Frameworks
-   Development integrations
-   ESP-IDF Toolchain
-   Python 환경
-   CMake
-   Ninja
-   ESP-IDF 관련 Tools

이번 HMI 프로젝트에서 Rust 및 Java 개발환경은 필요하지 않으므로 별도로
활성화할 필요가 없다.

------------------------------------------------------------------------

## 8. 설치 설정 최종 확인

`Ready to Install` 화면에서 설정을 확인한다.

본 설치에서 확인된 설정은 다음과 같다.

``` text
Using Embedded Python 3.11.2

Using Embedded Git:
C:\Espressif\tools\idf-git\2.44.0\cmd\git.exe

ESP-IDF:
v5.5.5

ESP-IDF Framework:
C:\Espressif\frameworks\esp-idf-v5.5.5

IDF_TOOLS_PATH:
C:\Espressif
```

### ESP32-P4 지원 확인

설치 대상 Target 목록에 반드시 다음 항목이 포함되어 있는지 확인한다.

``` text
ESP32-P4
```

예:

``` text
Targets:
ESP32
ESP32-C2
ESP32-C3
ESP32-C5
ESP32-C6
ESP32-C61
ESP32-S2
ESP32-S3
ESP32-P4
```

ESP32-P4가 포함되어 있으면 `Install`을 선택하여 설치를 진행한다.

------------------------------------------------------------------------

## 9. 설치 후 ESP-IDF 동작 확인

설치 완료 후 Windows 시작 메뉴에서 ESP-IDF용 PowerShell 또는 Command
Prompt를 실행한다.

예:

``` text
ESP-IDF PowerShell
```

일반 PowerShell에서는 ESP-IDF 환경이 활성화되지 않아 `idf.py`를 바로
찾지 못할 수 있다.

ESP-IDF PowerShell에서 다음 명령을 실행한다.

``` powershell
idf.py --version
```

정상적인 경우 다음과 같이 표시된다.

``` text
ESP-IDF v5.5.5
```

------------------------------------------------------------------------

## 10. ESP32-P4 Target 확인

다음 명령을 실행한다.

``` powershell
idf.py --list-targets
```

출력 목록에 다음 Target이 존재하는지 확인한다.

``` text
esp32p4
```

`esp32p4`가 존재하면 ESP32-P4용 ESP-IDF Toolchain을 사용할 준비가 된
것이다.

------------------------------------------------------------------------

## 11. 설치 문제 --- `idf.py`를 찾지 못하는 경우

일반 PowerShell에서 다음 명령을 실행했을 때:

``` powershell
idf.py --version
```

다음과 같은 오류가 발생할 수 있다.

``` text
'idf.py' 용어가 cmdlet, 함수, 스크립트 파일 또는
실행할 수 있는 프로그램 이름으로 인식되지 않습니다.
```

### 원인 1 --- 일반 PowerShell 사용

ESP-IDF 환경이 활성화되지 않은 일반 PowerShell에서는 `idf.py`를 찾지
못할 수 있다.

Windows 시작 메뉴에서 ESP-IDF PowerShell을 실행한 뒤 다시 확인한다.

### 원인 2 --- ESP-IDF Framework가 설치되지 않음

다음 명령으로 설치 디렉터리를 확인한다.

``` powershell
dir C:\Espressif
```

정상적인 설치라면 다음 디렉터리가 존재해야 한다.

``` text
C:\Espressif\frameworks
```

그리고:

``` powershell
dir C:\Espressif\frameworks
```

에서 다음과 같은 ESP-IDF Framework 디렉터리가 확인되어야 한다.

``` text
esp-idf-v5.5.5
```

만약 `C:\Espressif` 아래에 `dist`, `tools`만 존재하고 `frameworks`가
없다면 ESP-IDF Framework 설치가 완료되지 않은 상태일 수 있다.

이 경우 ESP-IDF Tools Installer를 다시 실행하고:

``` text
Download ESP-IDF
```

를 선택하여 Framework를 설치한다.

------------------------------------------------------------------------

## 12. 설치 완료 체크리스트

다음 항목을 모두 확인한다.

-   [ ] `C:\Espressif\frameworks\esp-idf-v5.5.5` 존재
-   [ ] ESP-IDF PowerShell 실행 가능
-   [ ] `idf.py --version` 정상 실행
-   [ ] ESP-IDF 버전이 `v5.5.5`
-   [ ] `idf.py --list-targets` 정상 실행
-   [ ] Target 목록에 `esp32p4` 존재

------------------------------------------------------------------------

## 13. 다음 단계

개발환경 설치 확인 후 다음 순서로 진행한다.

``` text
ESP-IDF 설치
    ↓
ESP32-P4 Target 확인
    ↓
Hello World 프로젝트 생성
    ↓
ESP32-P4 Target 설정
    ↓
Build
    ↓
개발보드 USB 연결
    ↓
Flash
    ↓
Serial Monitor
    ↓
제조사 LCD Demo
    ↓
Touch 확인
    ↓
LVGL HMI
```

첫 ESP32-P4 프로젝트에서는 다음 항목을 확인한다.

``` powershell
idf.py set-target esp32p4
idf.py build
```

이후 실제 개발 보드를 USB로 연결하고 Flash/Monitor 단계로 진행한다.
