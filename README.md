# 개발 지식 저장소

개발 과정에서 얻은 기술 지식을 Markdown으로 장기간 축적하는 개인 Knowledge Base입니다. 원본 문서는 Git으로 관리하며, MkDocs Material 사이트로 생성해 GitHub Pages에 배포합니다.

## 주요 분류

- C++, MFC
- Windows (BitLocker, VHDX, Troubleshooting)
- Automation (EtherCAT, Motion, Vision)
- Architecture
- Troubleshooting

## 로컬 실행 방법

Windows PowerShell에서 다음을 실행합니다.

```powershell
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
mkdocs serve
```

명령이 실행되면 브라우저에서 <http://127.0.0.1:8000>을 열어 로컬 문서를 확인합니다. 배포 전 빌드는 `mkdocs build --strict`로 검증합니다.

## 문서 추가 방법

알맞은 카테고리에 Markdown 파일을 추가합니다. 예: `docs/windows/troubleshooting/task-manager-cpu-zero.md`

`mkdocs.yml`의 `nav`에 새 문서를 등록하면 메뉴에 표시됩니다. 기존 문서와 중복되지 않는지 먼저 확인하고, 관련 문서에는 상대 Markdown 링크를 추가합니다. 문서 형식과 공개 저장소 보안 규칙은 [AGENTS.md](AGENTS.md)를 따릅니다.

## GitHub Pages

`main` 브랜치에 push하면 GitHub Actions가 사이트를 빌드하고 Pages에 배포합니다. 최초 1회 GitHub 저장소 **Settings → Pages**에서 Source가 **GitHub Actions**인지 확인하세요. 조직 저장소라면 Actions 및 Pages 배포 권한 정책도 허용되어 있어야 합니다.
