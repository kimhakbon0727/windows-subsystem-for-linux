# 🔄 WSL2 Ubuntu 환경 복원 가이드

> 백업한 WSL2 Ubuntu 환경을 새로운 컴퓨터에 그대로 복원하는 방법입니다.
> 백업 방법은 [README.md](./README.md)를 참고하세요.

---

## 📋 복원 전 체크리스트

- [ ] 백업 파일(`ubuntu-backup.tar`) USB 또는 클라우드로 새 PC에 옮기기
- [ ] 새 PC Windows 버전 확인 (Windows 10 2004 이상 또는 Windows 11 필요)
- [ ] 저장 공간 확인 (tar 파일 용량의 2배 이상 여유 공간 필요)

---

## 🚀 복원 순서

### 1️⃣ WSL2 설치 (새 PC에 WSL이 없다면)

```powershell
wsl --install
```

> 설치 후 PC 재시작이 필요할 수 있습니다.

---

### 2️⃣ 백업 파일 복원

```powershell
# 복원할 폴더 먼저 생성
mkdir C:\WSL

# 백업 파일 import
wsl --import Ubuntu C:\WSL\Ubuntu C:\backup\ubuntu-backup.tar
```

---

### 3️⃣ 실행 확인

```powershell
# 배포판 목록 확인
wsl --list --verbose

# WSL 실행
wsl
```

---

### 4️⃣ 기본 사용자 설정 (root로 로그인되는 경우)

import 후 root로 로그인될 수 있습니다. 아래 명령어로 기본 유저를 설정하세요.

```powershell
ubuntu config --default-user bon
```

---

## ⚠️ 주의사항

| 항목 | 내용 |
|------|------|
| 📁 백업 파일 이동 | USB 또는 Google Drive 등으로 새 PC에 옮겨야 함 |
| 💽 용량 | tar 파일 용량 넉넉히 확인 |
| 🪟 WSL2 지원 | Windows 10 (2004 이상) 또는 Windows 11 필요 |
| 👤 기본 사용자 | import 후 root로 로그인될 수 있음 |

---

## 🗂️ 파일 구조

```
📦 백업 파일
 ┣ 📄 ubuntu-backup.tar   ← WSL 전체 환경
 ┗ 📄 README.md           ← 백업 가이드 (선택)
```

---

> 🔄 마지막 업데이트: 2026년 4월 14일
