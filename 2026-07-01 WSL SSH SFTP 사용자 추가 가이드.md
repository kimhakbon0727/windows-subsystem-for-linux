# WSL SSH / FTP 사용자 추가 가이드

SSH 서버가 설치되어 있고 외부 접속이 가능한 상태에서, 새 사용자를 추가하는 방법입니다.

---

## 사용자 생성

```bash
sudo adduser --gecos "" userk2
```

> `--gecos ""` 옵션은 이름, 전화번호 등 부가 정보 입력을 생략합니다.

비밀번호 입력 프롬프트가 나오면 원하는 비밀번호를 설정합니다.

---

## SSH 접속

```bash
ssh userk2@<서버_IP>
```

---

## FTP 접속

FTP는 SSH 키 인증을 지원하지 않으므로 **비밀번호 인증**을 사용합니다.  
위에서 설정한 비밀번호로 FTP 클라이언트에서 접속하면 됩니다.

| 항목 | 값 |
|---|---|
| 호스트 | `<서버_IP>` |
| 사용자명 | `userk2` |
| 비밀번호 | 설정한 비밀번호 |
| 포트 | `21` (FTP 기본값) |

---

## 비밀번호 변경 (필요 시)

```bash
sudo passwd userk2
```

---

## sudo 권한 부여 (필요 시)

```bash
sudo usermod -aG sudo userk2
```

---

## 사용자 목록 확인

```bash
cat /etc/passwd | grep userk
```
