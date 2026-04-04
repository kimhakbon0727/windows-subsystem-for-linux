🐧 WSL (Windows Subsystem for Linux) 완벽 가이드

> Windows에서 Linux를 네이티브처럼 사용할 수 있는 WSL의 설치부터 활용까지 정리한 문서입니다.

---

## 📋 목차

- [WSL이란?](#-wsl이란)
- [사전 요구사항](#-사전-요구사항)
- [WSL 설치](#-wsl-설치)
- [Linux 배포판 설치](#-linux-배포판-설치)
- [기본 사용법](#-기본-사용법)
- [파일 시스템 접근](#-파일-시스템-접근)
- [네트워크 설정](#-네트워크-설정)
- [유용한 팁 & 명령어](#-유용한-팁--명령어)
- [WSL1 vs WSL2 비교](#-wsl1-vs-wsl2-비교)
- [문제 해결](#-문제-해결)

---

## 🤔 WSL이란?

**WSL(Windows Subsystem for Linux)** 은 Windows에서 Linux 환경을 직접 실행할 수 있게 해주는 Microsoft의 공식 기능입니다.

- ✅ 가상 머신 없이 Linux 커널을 Windows 위에서 실행
- ✅ Windows와 Linux 파일 시스템 간 자유로운 접근
- ✅ VS Code, Docker 등 개발 도구와 완벽 통합
- ✅ WSL2는 실제 Linux 커널 사용으로 높은 성능 제공

---

## ⚙️ 사전 요구사항

| 항목 | 요구사항 |
|------|----------|
| 🪟 Windows 버전 | Windows 10 (버전 2004 이상, 빌드 19041+) 또는 Windows 11 |
| 💾 저장 공간 | 최소 4GB 이상 여유 공간 |
| 🔧 BIOS 설정 | 가상화(Virtualization) 활성화 필요 |
| 🛡️ 권한 | 관리자 권한 필요 |

> 💡 **Windows 버전 확인**: `Win + R` → `winver` 입력

---

## 🚀 WSL 설치

### 방법 1: 한 줄 명령어로 설치 (권장)

**PowerShell을 관리자 권한으로 실행** 후 아래 명령어 입력:

```powershell
wsl --install
```

> 이 명령어 하나로 WSL2 활성화 + Ubuntu 자동 설치까지 완료됩니다.

설치 후 **재부팅** 필수!

---

### 방법 2: 단계별 수동 설치

#### 1️⃣ WSL 기능 활성화

```powershell
# WSL 기능 활성화
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# Virtual Machine Platform 활성화
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

#### 2️⃣ 시스템 재부팅

```powershell
Restart-Computer
```

#### 3️⃣ WSL2를 기본 버전으로 설정

```powershell
wsl --set-default-version 2
```

#### 4️⃣ Linux 커널 업데이트 패키지 설치

👉 [Microsoft 공식 WSL2 커널 업데이트 패키지 다운로드](https://aka.ms/wsl2kernel)

---

## 🐧 Linux 배포판 설치

### Microsoft Store에서 설치

1. **Microsoft Store** 열기
2. 원하는 배포판 검색 및 설치

| 배포판 | 특징 | 추천 대상 |
|--------|------|-----------|
| 🟠 Ubuntu 22.04 LTS | 가장 많이 사용, 문서 풍부 | 입문자, 일반 개발 |
| 🔵 Debian | 가볍고 안정적 | 서버 개발, 경량 사용 |
| 🟢 openSUSE | 기업용 환경 | 엔터프라이즈 개발 |
| ⚫ Kali Linux | 보안 도구 내장 | 보안/해킹 학습 |

### 명령어로 설치

```powershell
# 설치 가능한 배포판 목록 확인
wsl --list --online

# 특정 배포판 설치 (예: Ubuntu-22.04)
wsl --install -d Ubuntu-22.04
```

### 초기 설정

배포판 실행 후 사용자 이름과 비밀번호를 설정합니다:

```
Enter new UNIX username: your_username
New password: ********
Retype new password: ********
```

---

## 💻 기본 사용법

### WSL 실행 방법

```powershell
# 기본 배포판 실행
wsl

# 특정 배포판 실행
wsl -d Ubuntu-22.04

# 특정 명령어만 실행
wsl ls -la
```

### 자주 쓰는 WSL 관리 명령어

```powershell
# 설치된 배포판 목록 확인
wsl --list --verbose
wsl -l -v

# 실행 중인 WSL 종료
wsl --shutdown

# 특정 배포판 종료
wsl --terminate Ubuntu-22.04

# 기본 배포판 변경
wsl --set-default Ubuntu-22.04

# WSL 버전 변경 (1 ↔ 2)
wsl --set-version Ubuntu-22.04 2

# WSL 업데이트
wsl --update
```

### Linux 기본 패키지 업데이트

```bash
# 패키지 목록 업데이트 및 업그레이드
sudo apt update && sudo apt upgrade -y
```

---

## 📁 파일 시스템 접근

### Windows → Linux 파일 접근

```bash
# Windows C 드라이브는 /mnt/c/ 경로로 접근
ls /mnt/c/Users/YourName/Desktop

# Windows 파일 복사
cp /mnt/c/Users/YourName/file.txt ~/file.txt
```

### Linux → Windows 파일 탐색기 접근

```bash
# 현재 Linux 디렉토리를 Windows 탐색기에서 열기
explorer.exe .
```

또는 Windows 탐색기 주소창에 입력:
```
\\wsl$\Ubuntu-22.04\home\your_username
```

### 📌 경로 정리

| 위치 | 경로 |
|------|------|
| Windows C 드라이브 | `/mnt/c/` |
| Windows D 드라이브 | `/mnt/d/` |
| Linux 홈 디렉토리 | `~/` 또는 `/home/username/` |
| Linux 루트 | `/` |

> ⚠️ **성능 팁**: Linux 파일은 Linux 파일 시스템(`~/`)에, Windows 파일은 Windows 파일 시스템(`/mnt/c/`)에 저장하면 성능이 훨씬 좋습니다.

---

## 🌐 네트워크 설정

### Windows ↔ Linux 간 포트 접근

```bash
# Linux에서 실행한 서버는 Windows 브라우저에서 접근 가능
# 예: Linux에서 포트 3000으로 서버 실행 시
# Windows 브라우저에서 http://localhost:3000 접속
```

### Windows IP 주소 확인 (Linux에서)

```bash
# WSL에서 Windows 호스트 IP 확인
cat /etc/resolv.conf | grep nameserver
# 또는
ip route show | grep -i default | awk '{ print $3}'
```

---

## 🛠️ 유용한 팁 & 명령어

### 개발 환경 세팅 예시

```bash
# Node.js 설치 (nvm 사용 권장)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install --lts

# Python 설치
sudo apt install python3 python3-pip -y

# Git 설정
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# zsh + Oh My Zsh 설치 (선택)
sudo apt install zsh -y
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### VS Code 연동

```bash
# Linux 터미널에서 VS Code 실행
code .

# 특정 파일 열기
code filename.txt
```

> 💡 VS Code에서 **Remote - WSL** 확장 설치 시 더욱 편리하게 사용 가능

### Docker 연동

WSL2 + Docker Desktop 조합으로 강력한 개발 환경 구성:

1. [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/) 설치
2. Docker Desktop 설정 → **Use the WSL 2 based engine** 활성화
3. WSL Integration에서 사용할 배포판 선택

```bash
# Docker 정상 동작 확인
docker --version
docker run hello-world
```

### 🔑 SSH 키 생성 및 관리

```bash
# SSH 키 생성
ssh-keygen -t ed25519 -C "your@email.com"

# SSH 에이전트 시작
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

---

## ⚖️ WSL1 vs WSL2 비교

| 항목 | WSL1 | WSL2 |
|------|------|------|
| 🏗️ 아키텍처 | 번역 레이어 | 실제 Linux 커널 |
| ⚡ Linux 파일 I/O 속도 | 느림 | **빠름** |
| 📁 Windows 파일 접근 속도 | **빠름** | 느림 |
| 🐳 Docker 지원 | 제한적 | **완벽 지원** |
| 🔧 시스템 콜 호환성 | 부분적 | **완전 호환** |
| 💾 메모리 사용 | 적음 | 더 많음 |
| 🌐 네트워크 | Windows와 공유 | 별도 가상 네트워크 |

> ✅ **결론**: 대부분의 경우 **WSL2 사용을 강력 권장**

---

## 🔧 문제 해결

### ❌ `wsl --install` 실패 시

```powershell
# Windows 기능 수동 활성화
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

### ❌ WSL2 커널 에러 발생 시

```powershell
# WSL 업데이트
wsl --update

# 커널 재설치
wsl --shutdown
wsl --update
```

### ❌ 가상화 관련 오류 (`0x80370102`)

- BIOS/UEFI 진입 후 **Intel VT-x** 또는 **AMD-V** 활성화 필요
- Hyper-V 기능 활성화 확인

```powershell
# Hyper-V 활성화
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

### ❌ 배포판 재설치

```powershell
# 배포판 제거 (데이터 삭제 주의!)
wsl --unregister Ubuntu-22.04

# 재설치
wsl --install -d Ubuntu-22.04
```

### ❌ 메모리/CPU 사용량 제한 설정

`C:\Users\YourName\.wslconfig` 파일 생성:

```ini
[wsl2]
memory=4GB       # 최대 메모리 제한
processors=2     # CPU 코어 수 제한
swap=2GB         # 스왑 메모리
```

설정 적용:
```powershell
wsl --shutdown
```

---

## 📚 참고 링크

- 📖 [Microsoft WSL 공식 문서](https://docs.microsoft.com/ko-kr/windows/wsl/)
- 🐧 [Ubuntu WSL 공식 가이드](https://ubuntu.com/wsl)
- 🐳 [Docker Desktop WSL2 가이드](https://docs.docker.com/desktop/wsl/)
- 💻 [VS Code WSL 개발 가이드](https://code.visualstudio.com/docs/remote/wsl)

---

<div align="center">

**⭐ 도움이 되셨다면 Star를 눌러주세요!**

Made with ❤️ for developers

</div>
