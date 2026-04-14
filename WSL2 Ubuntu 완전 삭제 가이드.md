# 🗑️ WSL2 Ubuntu 완전 삭제 가이드

> WSL2 Ubuntu 환경을 완전히 삭제하는 방법입니다.
> 삭제 전 백업을 권장합니다 → [README.md](./README.md) 참고

---

## ⚠️ 삭제 전 체크리스트

- [ ] 중요한 파일 백업 완료 (`wsl --export`)
- [ ] 삭제할 배포판 이름 확인 (`wsl --list --verbose`)
- [ ] 작업 중인 WSL 세션 모두 종료

---

## 🚀 삭제 방법

### 방법 1. 배포판만 삭제 (WSL 기능은 유지) ⭐

WSL 자체는 남기고 Ubuntu만 삭제합니다.

```powershell
# 배포판 등록 해제 및 삭제
wsl --unregister Ubuntu
```

> ⚠️ Ubuntu 관련 모든 파일이 영구 삭제됩니다. 복구 불가능합니다!

---

### 방법 2. WSL 완전 제거 (기능까지 삭제)

WSL 기능 자체를 Windows에서 완전히 제거합니다.

```powershell
# 1. 배포판 먼저 삭제
wsl --unregister Ubuntu

# 2. WSL 기능 비활성화
dism.exe /online /disable-feature /featurename:Microsoft-Windows-Subsystem-Linux /norestart
dism.exe /online /disable-feature /featurename:VirtualMachinePlatform /norestart
```

> 🔄 명령어 실행 후 PC를 재시작해야 완전히 적용됩니다.

---

## ✅ 삭제 확인

```powershell
wsl --list
```

아래 메시지가 뜨면 성공입니다.
```
Windows Subsystem for Linux에 설치된 배포판이 없습니다.
```

---

## 🗂️ 상황별 추천 삭제 방식

| 상황 | 추천 방법 |
|------|-----------|
| 🔄 Ubuntu만 재설치 예정 | 방법 1 (배포판만 삭제) |
| 🧹 WSL 흔적 없이 깨끗하게 | 방법 2 (완전 제거) |
| 💼 회사 PC 반납 | 방법 2 (완전 제거) |

---

## 🔗 관련 문서

- [README.md](./README.md) - WSL2 백업 가이드
- [RESTORE.md](./RESTORE.md) - WSL2 복원 가이드

---

> 🔄 마지막 업데이트: 2026년 4월 14일
