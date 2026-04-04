# 🗄️ WSL MariaDB 외부 접속 설정 가이드

WSL(Windows Subsystem for Linux)에서 MariaDB를 설치하고 외부에서 접속할 수 있도록 설정하는 방법입니다.

---

## 📡 네트워크 구조

```
외부 기기 → 공인 IP:3388 → 공유기 → Windows PC (192.168.35.20) → WSL (172.x.x.x:3306)
```

---

## ✅ 설정 순서

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

### 2️⃣ 보안 설정 (권장)

> ⚠️ 외부에서 접속 가능한 환경이라면 반드시 진행하세요!

```bash
sudo mysql_secure_installation
```

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

### 3️⃣ 외부 접속 허용 설정

MariaDB 설정 파일을 열어 `bind-address`를 수정합니다.

```bash
sudo vi /etc/mysql/mariadb.conf.d/50-server.cnf
```

아래 줄을 찾아서:
```
bind-address = 127.0.0.1
```

이렇게 변경:
```
bind-address = 0.0.0.0
```

저장 후 MariaDB 재시작:

```bash
sudo service mariadb restart
```

---

### 4️⃣ 외부 접속용 사용자 생성

```bash
sudo mysql -u root -p
```

```sql
CREATE USER '유저명'@'%' IDENTIFIED BY '비밀번호';
GRANT ALL PRIVILEGES ON *.* TO '유저명'@'%';
FLUSH PRIVILEGES;
EXIT;
```

---

### 5️⃣ Windows 포트포워딩 설정

**PowerShell을 관리자 권한으로 열고** 실행합니다.

```powershell
netsh interface portproxy add v4tov4 listenport=3388 listenaddress=0.0.0.0 connectport=3306 connectaddress=172.20.232.187
```

> `172.20.232.187` 부분을 본인의 WSL IP(`hostname -I` 로 확인)로 변경하세요.

---

### 6️⃣ Windows 방화벽 허용

```powershell
New-NetFirewallRule -DisplayName "MariaDB WSL" -Direction Inbound -LocalPort 3388 -Protocol TCP -Action Allow
```

---

### 7️⃣ 공유기 포트포워딩 설정 (H724G 기준)

공유기 관리 페이지(`http://192.168.35.1`) → 방화벽 → 포트 포워딩에서 아래 값을 추가합니다.

| 항목 | 값 |
|------|-----|
| 서비스 포트 | `3388` |
| 프로토콜 | `TCP` |
| 내부 IP 주소 | `192.168.35.20` |
| 포트 | `3306` |
| 설명 | `MariaDB WSL` |

> ✅ 적용 버튼 클릭 후 즉시 반영됩니다. (재부팅 불필요)

---

### 8️⃣ 외부에서 접속 테스트 (DBeaver)

DBeaver 실행 후 새 연결 추가:

| 항목 | 값 |
|------|-----|
| Server Host | `YOUR_PUBLIC_IP` |
| Port | `3388` |
| Username | 설정한 유저명 |
| Password | 설정한 비밀번호 |

> `YOUR_PUBLIC_IP`는 [https://whatismyip.com](https://whatismyip.com) 에서 확인하세요.

> ⚠️ **같은 와이파이에서 공인 IP로 접속하면 안 될 수 있습니다! (헤어핀 NAT 문제)**  
> 반드시 **외부 네트워크(LTE 등)** 에서 테스트하세요.

---

## 🔒 SSH 터널링으로 외부 접속 (권장)

> SK 브로드밴드 등 통신사에서 DB 포트를 차단하는 경우 SSH 터널링을 사용하세요.  
> DB 포트를 외부에 직접 열지 않아도 되어 **더 안전한 방법**입니다. ✅

### DBeaver SSH 터널링 설정

**1. SSH Tunnel 탭 설정:**

| 항목 | 값 |
|------|-----|
| Host | `YOUR_PUBLIC_IP` |
| Port | `2288` |
| Username | `bon` (SSH 유저명) |
| Password | SSH 비밀번호 |

**2. Main 탭 설정:**

| 항목 | 값 |
|------|-----|
| Host | `127.0.0.1` |
| Port | `3306` |
| Username | `bon` (DB 유저명) |
| Password | DB 비밀번호 |

> ✅ SSH 터널링은 암호화된 SSH 연결을 통해 DB에 접속하므로 보안상 더 안전합니다.

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

## 🔧 포트포워딩 관리

```powershell
# 설정 확인
netsh interface portproxy show all

# 설정 삭제
netsh interface portproxy delete v4tov4 listenport=3388 listenaddress=0.0.0.0
```

---

## ⚠️ 주의사항

- **WSL IP는 재부팅 시마다 변경될 수 있습니다.**  
  재부팅 후 접속이 안 된다면 `hostname -I` 로 IP를 다시 확인하고 포트포워딩을 재설정하세요.

- **공인 IP도 재부팅 시 변경될 수 있습니다.**  
  고정이 필요하면 DDNS 서비스(예: [DuckDNS](https://www.duckdns.org)) 사용을 권장합니다.

- **같은 네트워크에서 공인 IP로 접속 시 헤어핀 NAT 문제로 접속이 안 될 수 있습니다.**  
  내부에서는 내부 IP(`192.168.35.20:3388`)로 접속하세요.
