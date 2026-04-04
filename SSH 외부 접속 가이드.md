# 🔐 WSL SSH/SFTP 외부 접속 설정 가이드

WSL(Windows Subsystem for Linux)에서 실행 중인 SSH 서버를 외부에서 접속할 수 있도록 설정하는 방법입니다.

---

## 📡 네트워크 구조

```
외부 기기 → 공인 IP:2288 → 공유기 → Windows PC (192.168.35.20:2288) → WSL (172.20.232.187:22)
```

---

## ✅ 설정 순서

### 1️⃣ WSL에 SSH 서버 설치 및 실행

WSL 터미널에서 순서대로 실행합니다.

```bash
# 설치
sudo apt update
sudo apt install openssh-server -y

# 실행
sudo service ssh start

# 상태 확인
sudo service ssh status
```

> `active (running)` 이 나오면 정상입니다. ✅

---

### 2️⃣ Windows 포트포워딩 설정

**PowerShell을 관리자 권한으로 열고** 아래 명령어를 실행합니다.

```powershell
netsh interface portproxy add v4tov4 listenport=2288 listenaddress=0.0.0.0 connectport=22 connectaddress=172.20.232.187
```

> `172.20.232.187` 부분을 본인의 WSL IP(`hostname -I` 로 확인)로 변경하세요.

---

### 3️⃣ Windows 방화벽 허용

```powershell
New-NetFirewallRule -DisplayName "SSH WSL" -Direction Inbound -LocalPort 2288 -Protocol TCP -Action Allow
```

---

### 4️⃣ 공유기 포트포워딩 설정 (H724G 기준)

공유기 관리 페이지(`http://192.168.35.1`) → 방화벽 → 포트 포워딩에서 아래 값을 추가합니다.

| 항목 | 값 |
|------|-----|
| 서비스 포트 | `2288` |
| 프로토콜 | `TCP` |
| 내부 IP 주소 | `192.168.35.20` |
| 포트 | `22` |
| 설명 | `SSH WSL` |

> ✅ 적용 버튼 클릭 후 즉시 반영됩니다. (재부팅 불필요)

---

### 5️⃣ 외부에서 접속 테스트

#### SSH 접속
```bash
ssh -p 2288 유저명@YOUR_PUBLIC_IP
```

#### SFTP 접속
```bash
sftp -P 2288 유저명@YOUR_PUBLIC_IP
```

> `유저명`은 WSL에서 `whoami` 명령어로 확인할 수 있습니다.  
> `YOUR_PUBLIC_IP`는 [https://whatismyip.com](https://whatismyip.com) 에서 확인하세요.

---

## 🔧 포트포워딩 관리

```powershell
# 설정 확인
netsh interface portproxy show all

# 설정 삭제
netsh interface portproxy delete v4tov4 listenport=2288 listenaddress=0.0.0.0
```

---

## ⚠️ 주의사항

- **WSL IP는 재부팅 시마다 변경될 수 있습니다.**  
  재부팅 후 접속이 안 된다면 `hostname -I` 로 IP를 다시 확인하고 포트포워딩을 재설정하세요.

- **공인 IP도 재부팅 시 변경될 수 있습니다.**  
  고정이 필요하면 DDNS 서비스(예: [DuckDNS](https://www.duckdns.org)) 사용을 권장합니다.

- **포트 선택 시 이미 사용 중인 포트는 피하세요.**  
  현재 사용 중인 SSH 관련 포트: `22`, `2200`, `2210`, `2211`, `2222`, `22222`

- 보안을 위해 사용하지 않을 때는 포트포워딩 규칙을 삭제하는 것을 권장합니다.
