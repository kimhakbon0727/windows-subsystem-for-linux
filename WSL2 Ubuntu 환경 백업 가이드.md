# 🖥️ WSL2 Ubuntu 환경 백업 가이드

> 내 WSL2 Ubuntu 개발 환경을 백업하고 복원하는 방법을 정리한 문서입니다.

---

## 📋 현재 환경

| 항목 | 내용 |
|------|------|
| 배포판 | Ubuntu |
| WSL 버전 | WSL2 |
| 상태 | ✅ 정상 |

---

## 🔍 환경 확인 방법

백업 전 현재 설정을 먼저 확인합니다.

### PowerShell에서 배포판 확인
```powershell
wsl --list --verbose
```

### WSL 내부 환경 한번에 확인
```bash
echo "=== SHELL ===" && echo $SHELL && \
echo "=== DOTFILES ===" && ls -la ~ | grep "^\." && \
echo "=== WSL.CONF ===" && cat /etc/wsl.conf 2>/dev/null || echo "없음" && \
echo "=== KEY TOOLS ===" && which git node python3 python docker zsh fish 2>/dev/null
```

---

## 💾 백업 방법

### 방법 1. 전체 배포판 내보내기 (권장 ⭐)

가장 완전한 백업 방법입니다. 모든 설정과 파일이 포함됩니다.

```powershell
# PowerShell에서 실행
wsl --export Ubuntu C:\backup\ubuntu-backup.tar
```

> 📁 파일 크기가 수 GB가 될 수 있습니다.

---

### 방법 2. 설정 파일(dotfiles)만 백업

가볍게 핵심 설정만 백업합니다.

```bash
mkdir ~/backup

# 🐚 Shell 설정
cp ~/.bashrc ~/backup/
cp ~/.zshrc ~/backup/          # zsh 사용 시
cp ~/.profile ~/backup/
cp ~/.bash_profile ~/backup/

# 🔑 SSH 키
cp -r ~/.ssh ~/backup/

# 🔧 Git 설정
cp ~/.gitconfig ~/backup/

# 📝 기타 설정
cp ~/.vimrc ~/backup/
cp ~/.tmux.conf ~/backup/
```

---

### 방법 3. 설치된 패키지 목록 백업

```bash
dpkg --get-selections > ~/backup/packages.list
```

---

### 방법 4. WSL 설정 파일 백업

```bash
# WSL 내부 설정
cat /etc/wsl.conf

# Windows 측 설정 (PowerShell에서 확인)
cat $env:USERPROFILE\.wslconfig
```

---

## ♻️ 복원 방법

### 전체 배포판 복원

```powershell
# PowerShell에서 실행
wsl --import Ubuntu C:\WSL\Ubuntu C:\backup\ubuntu-backup.tar
```

### 패키지 목록으로 복원

```bash
sudo dpkg --set-selections < ~/backup/packages.list
sudo apt-get dselect-upgrade
```

---

## 📁 백업 파일 목록 체크리스트

백업 후 아래 항목들이 포함되었는지 확인하세요.

- [ ] `~/.bashrc` 또는 `~/.zshrc`
- [ ] `~/.profile`
- [ ] `~/.gitconfig`
- [ ] `~/.ssh/` (SSH 키)
- [ ] `/etc/wsl.conf`
- [ ] `C:\Users\사용자명\.wslconfig`
- [ ] `packages.list` (설치된 패키지 목록)

---

## 🗂️ 상황별 추천 백업 방식

| 상황 | 추천 방법 |
|------|-----------|
| 💻 새 PC로 완전 이전 | `wsl --export` |
| ⚙️ 설정만 옮기기 | dotfiles 백업 |
| 🔄 환경 재현 | 패키지 목록 + dotfiles |
| 🆘 빠른 스냅샷 | `wsl --export` |

---

## 📝 메모

```
# 여기에 본인의 환경 특이사항이나 설치한 툴 목록을 기록하세요
# 예: nvm, pyenv, docker, etc.
```

---

> 🔄 마지막 업데이트: 2026년 4월
