---
tags:
  - MFC
  - C++
  - DLL
---

# MFC DLL ABI 호환성

## 개요

MFC DLL과 EXE 사이의 경계에서는 컴파일러, Runtime, MFC 설정 차이로 ABI(Application Binary Interface) 문제가 생길 수 있습니다. 이 문서는 특정 제품의 정답이 아니라 ABI 안정성을 높이기 위한 일반적인 설계 원칙을 정리합니다.

## 문제 / 증상

`CString`, `CWnd`, STL 컨테이너 같은 C++/MFC 객체를 DLL API에 직접 노출하면 양쪽 모듈의 빌드 옵션이나 라이브러리 버전이 다를 때 메모리 해제 실패, 예외 처리 불일치, 객체 레이아웃 차이와 같은 문제가 발생할 수 있습니다.

## 원인

DLL과 EXE가 `/MD` 또는 `/MT`로 서로 다르게 빌드되면 CRT 상태와 메모리 할당자가 달라질 수 있습니다. MFC Extension DLL은 호스트 MFC 상태 및 리소스를 공유하는 전제를 가지는 반면, Regular DLL은 사용 방식과 MFC 초기화 모델을 별도로 고려해야 합니다. 이러한 차이는 객체를 생성한 모듈과 해제한 모듈이 달라질 때 특히 중요합니다.

## 해결 방법

- DLL 경계는 가능한 한 C ABI와 단순 POD 데이터로 유지합니다.
- 문자열은 `const wchar_t*`와 길이처럼 명확한 형태로 전달하고, 할당한 모듈이 해제하는 규칙을 문서화합니다.
- 핸들 기반 API를 사용해 내부 MFC/C++ 객체를 숨기고, 생성·파괴 함수를 같은 DLL에 둡니다.
- 필요한 경우 DLL 유형(Extension/Regular), toolset, CRT, MFC 사용 옵션을 배포 단위에서 일관되게 맞춥니다.

## 코드 / 명령어

다음처럼 불투명 핸들과 기본 Windows 타입을 사용하면 C++ 객체를 API 표면에서 분리하는 데 도움이 됩니다.

```cpp
extern "C"
{
    TREELIST_API void* TreeList_Create(HWND hParent);
    TREELIST_API void TreeList_Destroy(void* handle);
    TREELIST_API BOOL TreeList_SetText(
        void* handle,
        int item,
        const wchar_t* text);
}
```

## 주의사항

이 방식이 모든 호환성 문제를 자동으로 없애지는 않습니다. Calling convention, 구조체 정렬, 오류 전달 방식, 문자 인코딩, 지원할 Windows 및 toolset 범위도 API 계약에 포함해야 합니다. MFC UI 객체는 생성 스레드와 수명 규칙도 별도로 지켜야 합니다.

## 결론

DLL 경계를 작고 명시적인 C 스타일 계약으로 제한하고, 생성·소유·파괴 책임을 한 모듈에 모으면 ABI 변경에 대한 민감도를 낮출 수 있습니다.
