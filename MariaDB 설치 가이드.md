# 🗄️ WSL MariaDB 설치 가이드

WSL(Windows Subsystem for Linux)에서 MariaDB를 설치하고 실행하는 방법입니다.

---

## ✅ 설치 순서

### 1️⃣ MariaDB 설치 및 실행

WSL 터미널에서 순서대로 실행합니다.

```bash
# 설치
sudo apt update
sudo apt install mariadb-server -y

# 실행
sudo service mariadb start

# 상태 확인
sudo service mariadb status
```

> `active (running)` 이 나오면 정상입니다. ✅

---

## 🔒 보안 설정 (선택사항이지만 권장)

> ⚠️ 외부에서 접속 가능한 환경이라면 **반드시** 진행하세요!

```bash
sudo mysql_secure_installation
```

실행 시 아래 질문들이 나옵니다:

| 질문 | 추천 답변 |
|------|-----------|
| Enter current password for root | 그냥 `Enter` |
| Switch to unix_socket authentication | `n` |
| Change the root password | `y` → 비밀번호 설정 |
| Remove anonymous users | `y` |
| Disallow root login remotely | `y` |
| Remove test database | `y` |
| Reload privilege tables | `y` |

---

## 🔧 기본 명령어

```bash
# MariaDB 시작
sudo service mariadb start

# MariaDB 중지
sudo service mariadb stop

# MariaDB 재시작
sudo service mariadb restart

# MariaDB 접속
sudo mysql -u root -p
```

---

## ⚠️ 주의사항

- **WSL 재부팅 시 MariaDB가 자동으로 시작되지 않을 수 있습니다.**  
  접속이 안 된다면 `sudo service mariadb start` 로 수동 실행하세요.

- 외부에서 접속하려면 추가적인 포트포워딩 및 MariaDB 사용자 권한 설정이 필요합니다.
