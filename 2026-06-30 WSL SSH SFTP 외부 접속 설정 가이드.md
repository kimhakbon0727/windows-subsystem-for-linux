# 🔐 WSL SSH/SFTP 외부 접속 설정 가이드 (H724G 공유기 기준)

WSL(Ubuntu)에서 실행 중인 SSH 서버를 **외부(LTE 등)에서도 WinSCP/SFTP로 접속**할 수 있도록 설정한 과정 정리입니다.

---

## 📡 네트워크 구조

```
외부 기기 (LTE) → 공인 IP:2222 → 공유기(H724G) → Windows PC(192.168.35.193:2222)
                                                  → portproxy
                                                  → WSL(192.168.75.46:22)
```

---

## ✅ 설정 순서

### 1️⃣ WSL에 SSH 서버 설치 및 실행

```bash
sudo apt update
sudo apt install openssh-server -y
sudo service ssh start
sudo systemctl status ssh.service
```

`active (running)` 확인.

---

### 2️⃣ ⚠️ ssh.socket 문제 해결 (가장 중요한 트러블슈팅)

기본적으로 Ubuntu는 systemd **socket activation** 방식으로 sshd를 띄우는데, 이 상태에서는 외부(Windows portproxy 등)에서 들어오는 연결이 다음과 같은 에러로 거부됨:

```
kex_exchange_identification: read: Connection reset
```

**해결: socket activation을 끄고 서비스가 직접 0.0.0.0:22를 리슨하게 변경**

```bash
sudo systemctl stop ssh.socket
sudo systemctl disable ssh.socket
sudo systemctl mask ssh.socket
sudo systemctl restart ssh.service
sudo systemctl status ssh.service
```

✅ `status` 결과에 `TriggeredBy: ● ssh.socket` 줄이 **사라지면** 정상.

> 🔁 WSL을 재부팅하면 `ssh.service`가 다시 꺼져있는 경우가 있음 → 그때마다 `sudo systemctl start ssh.service`로 재시작 필요.

---

### 3️⃣ WSL 내부 IP 확인

```bash
ip addr show eth0 | grep inet
```

예: `192.168.75.46` (⚠️ WSL 재부팅 시마다 바뀔 수 있음)

---

### 4️⃣ Windows → WSL 포트포워딩 (portproxy)

PowerShell **관리자 권한**:

```powershell
netsh interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=22 connectaddress=192.168.75.46
```

확인:
```powershell
netsh interface portproxy show v4tov4
```

---

### 5️⃣ Windows 방화벽 허용

```powershell
New-NetFirewallRule -DisplayName "SSH WSL 2222" -Direction Inbound -LocalPort 2222 -Protocol TCP -Action Allow
```

---

### 6️⃣ 공유기 포트포워딩 (이미 등록되어 있으면 생략 가능)

공유기 관리 페이지(`http://192.168.35.1`) → **방화벽 → 포트 포워딩**

| 서비스 포트 | 프로토콜 | 내부 IP | 포트 | 설명 |
|---|---|---|---|---|
| 2222 | TCP | 192.168.35.193 (Windows PC IP) | 2222 | SFTP 서버 |

> 💡 이미 같은 내용으로 등록되어 있다면 **공유기 설정은 건드릴 필요 없음** — Windows portproxy만 맞춰주면 기존 규칙을 그대로 활용 가능.

---

### 7️⃣ 로컬 테스트 (Windows PowerShell)

```powershell
ssh -p 2222 userk@localhost
```

`Connection reset` 없이 host key 확인 → 비밀번호 입력 → 로그인 성공하면 OK.

---

### 8️⃣ 외부 접속 테스트

PC 와이파이 IP 말고, **공인 IP**로 접속해야 함:

```bash
curl ifconfig.me   # 공인 IP 확인 (WSL에서)
```

```bash
ssh -p 2222 userk@<공인IP>
sftp -p 2222 userk@<공인IP>
```

#### WinSCP 접속 설정

| 항목 | 값 |
|---|---|
| 프로토콜 | SFTP |
| 호스트 이름 | 공인 IP |
| 포트 번호 | 2222 |
| 사용자 이름 | WSL 사용자명 (`whoami`로 확인) |
| 비밀번호 | WSL 로그인 비밀번호 |

---

## 🚨 트러블슈팅 — 절대 하지 말 것

진행 중 **`netsh interface ip add address` 로 Wi-Fi 어댑터에 보조(고정) IP를 수동 추가**하는 시도를 했다가, 어댑터가 DHCP 모드에서 벗어나며 **원격 PC 자체가 네트워크에서 통째로 떨어져 나가는 사고**가 발생했음 (Chrome 원격 데스크톱 접속 불가, "인터넷 연결 안 됨, 보안" 상태).

### 원인
```powershell
netsh interface ip add address "Wi-Fi" 192.168.35.20 255.255.255.0
```
→ 어댑터가 고정 IP로 전환되며 라우터로부터 정상 IP(192.168.35.x)를 못 받아오고, `169.254.x.x` (APIPA, 게이트웨이 없음) 임시 주소만 잡힘.

### 복구 방법
PC에 **직접 물리적으로 접근**해서:

```powershell
netsh interface ip set address "Wi-Fi" dhcp
netsh interface ip set dns "Wi-Fi" dhcp
ipconfig /release
ipconfig /renew
```

✅ DHCP가 다시 "예"로 바뀌고, 정상 IP(192.168.35.193)와 게이트웨이(192.168.35.1)가 잡히면 복구 완료.

### 교훈
- 원격으로 접속해서 작업할 땐 **네트워크 어댑터(IP 추가/변경) 관련 명령어는 절대 실행하지 말 것** — 연결이 끊기면 원격으로 되돌릴 방법이 없음.
- 기존 라우터 포워딩 규칙에 맞춰 PC IP를 바꾸려 하지 말고, **반대로 portproxy를 그 PC의 실제 IP에 맞게 등록**하는 방향으로 풀어야 안전함.

---

## 🔒 보안 권장사항

외부에 SSH 포트를 열어두면 전 세계에서 무차별 대입(brute force) 공격 시도가 들어올 수 있음.

- ✅ 비밀번호는 12자 이상, 영문+숫자+특수문자 조합으로 충분히 복잡하게
- ✅ 가능하면 SSH 키 인증으로 전환하고 비밀번호 로그인 비활성화
- ✅ 사용하지 않을 땐 포트포워딩 규칙 비활성화 (라우터 또는 `netsh interface portproxy delete`)
- ✅ `fail2ban` 설치해서 반복 실패 IP 자동 차단 고려

---

## 🛠️ 설정 관리 명령어 모음

```powershell
# portproxy 확인
netsh interface portproxy show v4tov4

# portproxy 삭제
netsh interface portproxy delete v4tov4 listenport=2222 listenaddress=0.0.0.0

# 방화벽 규칙 확인
Get-NetFirewallRule -DisplayName "SSH WSL 2222"
```

```bash
# WSL ssh 서비스 상태
sudo systemctl status ssh.service

# WSL ssh 서비스 시작
sudo systemctl start ssh.service

# WSL IP 확인
ip addr show eth0 | grep inet
```

---

## ✅ 최종 결과

- `ssh -p 2222 userk@<공인IP>` / WinSCP SFTP 접속 성공 확인 완료 🎉
- 공유기 설정은 변경 없이 기존 `2222 → 192.168.35.193:2222` 규칙 그대로 재활용
