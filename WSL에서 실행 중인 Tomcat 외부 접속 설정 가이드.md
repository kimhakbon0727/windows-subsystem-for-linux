# 🌐 WSL에서 실행 중인 Tomcat 외부 접속 설정 가이드

WSL(Windows Subsystem for Linux)에서 실행 중인 Tomcat 서버를 외부 기기에서 접속할 수 있도록 설정하는 방법입니다.

## 📡 네트워크 구조

```
외부 기기 → 윈도우 PC (공유기 IP) → 포트 프록시 → WSL (x.x.x.x)
```

---

## ✅ 설정 순서

### 1️⃣ WSL 내부 IP 확인

WSL 터미널에서 아래 명령어를 입력합니다.

```bash
hostname -I
```

> ⚠️ 소문자 `-i`가 아닌 **대문자 `-I`** 를 사용하세요!
> - 소문자 `-i` → `127.0.1.1` (루프백 주소, 사용 불가 ❌)
> - 대문자 `-I` → `x.x.x.x` (실제 WSL IP ✅)

예시 결과:
```
xxx.xx.xxx.xxx
```

---

### 2️⃣ Windows 포트 포워딩 설정

**PowerShell을 관리자 권한으로 열고** 아래 명령어를 실행합니다.

```powershell
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=xxxx connectaddress=xxx.xx.xxx.xxx
```

> `xxx.xx.xxx.xxx` 부분을 1단계에서 확인한 본인의 WSL IP로 변경하세요.

---

### 3️⃣ Windows 방화벽 허용

외부에서 윈도우로 들어오는 8080 포트를 허용합니다.

```powershell
New-NetFirewallRule -DisplayName "Tomcat WSL" -Direction Inbound -LocalPort xxxx -Protocol TCP -Action Allow
```

---

### 4️⃣ 윈도우 실제 IP 확인

```powershell
ipconfig
```

출력 결과에서 **IPv4 주소** (`xxx.xxx.x.x`)를 확인합니다.

---

### 5️⃣ 외부 접속 테스트

같은 와이파이에 연결된 기기의 브라우저에서 아래 주소로 접속합니다.

```
http://xxx.xxx.xx.xx:xxxx
```

> `xxx.xxx.xx.xx` 부분을 4단계에서 확인한 본인의 윈도우 IP로 변경하세요.

---

## 🔧 설정 관리

| 명령어 | 설명 |
|--------|------|
| `netsh interface portproxy show all` | 현재 포트 포워딩 설정 확인 |
| `netsh interface portproxy delete v4tov4 listenport=xxxx listenaddress=0.0.0.0` | 포트 포워딩 설정 삭제 |
| `netsh interface portproxy reset` | 모든 포트 포워딩 설정 초기화 |

---

## ⚠️ 주의사항

- **WSL IP는 재부팅 시마다 변경될 수 있습니다.**  
  재부팅 후 접속이 안 된다면 1단계부터 다시 IP를 확인하고 포트 포워딩을 재설정하세요.

- Tomcat이 `localhost`에서만 리스닝하도록 설정된 경우 외부 접속이 안 될 수 있습니다.  
  `/etc/tomcat10/server.xml`의 `Connector` 부분에 `address="0.0.0.0"` 설정 여부를 확인하세요.
