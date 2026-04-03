# 🚀 Apache Tomcat 설치 및 운영 가이드

이 문서는 Linux(Ubuntu/WSL2) 환경에서의 Tomcat 설치, 운영 명령어, 폴더 구조 및 실무 트러블슈팅 사례를 정리한 문서입니다.

---

## 1. Tomcat 설치 (Ubuntu/WSL2)

### 🛠️ 사전 준비 (JDK 및 네트워크 도구 설치)
```bash
# 1. 저장소 업데이트 및 universe 활성화
sudo add-apt-repository universe
sudo apt update

# 2. JDK 및 net-tools 설치
sudo apt install default-jdk net-tools -y
```

### 📥 Tomcat 설치 (Tomcat 10 기준)
```bash
sudo apt install tomcat10 tomcat10-admin -y
```

---

## 2. 주요 운영 명령어 (Service Management)

| 기능 | 명령어 |
| :--- | :--- |
| **서비스 시작** | `sudo systemctl start tomcat10` |
| **서비스 중지** | `sudo systemctl stop tomcat10` |
| **서비스 재시작** | `sudo systemctl restart tomcat10` |
| **상태 확인** | `sudo systemctl status tomcat10` |
| **로그 모니터링** | `tail -f /var/log/tomcat10/catalina.out` |
| **포트 점유 확인** | `netstat -an | grep 8080` |

---

## 3. Tomcat 주요 폴더 경로 (Directory Structure)

| 폴더명 | 용도 | 실제 경로 (Ubuntu 기준) |
| :--- | :--- | :--- |
| **`bin`** | **실행 파일** | `/usr/share/tomcat10/bin/` |
| **`conf`** | **설정 파일** | `/etc/tomcat10/` |
| **`webapps`** | **배포 경로** | `/var/lib/tomcat10/webapps/` |
| **`logs`** | **로그 저장** | `/var/log/tomcat10/` |

---

## 4. 핵심 설정 파일 상세 설명 (`conf/`)

| 파일명 | 역할 및 실무 활용 |
| :--- | :--- |
| **`server.xml`** | **서버 엔진 설정.** 포트 변경, **Connector(Proxy) 설정**, Valve 설정 등. |
| **`web.xml`** | **앱 공통 설정.** 세션 시간, **설정 파일 경로(param-value)** 등 지정. |
| **`context.xml`** | **프로젝트 개별 설정.** **DocBase(소스 경로)** 연동 및 DB 자원 정의. |

---

## 5. [실무] 로그 파일 분석 (`logs/`)

* **`catalina.out`**: 톰캣 엔진 로그 및 표준 출력. (장애 시 최우선 확인)
* **`localhost_access_log`**: HTTP 요청 기록. (IP, 접속 URL 등 **통계 테이블 분석용**)

---

## 6. [실무] Proxy(Nginx/L4) 연동 시 필수 설정

웹 서버(Proxy) 뒷단에서 톰캣이 동작할 때, 리다이렉트 경로가 깨지거나 실제 IP가 누락되는 것을 방지하기 위한 설정입니다.

**위치:** `/etc/tomcat10/server.xml`

### ① Connector 설정 (포트 및 프록시 정보 명시)
```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           proxyName="[www.yourdomain.com](https://www.yourdomain.com)"
           proxyPort="80"
           redirectPort="443" />
```

### ② RemoteIpValve 설정 (실제 사용자 IP 수집)
```xml
<Valve className="org.apache.catalina.valves.RemoteIpValve"
       remoteIpHeader="x-forwarded-for"
       proxiesHeader="x-forwarded-by"
       protocolHeader="x-forwarded-proto" />
```

---

## 7. [실무] 특정 경로 소스 배포 (DocBase 설정)
WAR 배포 대신 특정 폴더의 소스를 직접 연결할 때 사용합니다.

**위치:** `/etc/tomcat10/server.xml`
```xml
<Host name="localhost" appBase="webapps" ...>
    <Context path="" docBase="/home/user/workspace/my_project" reloadable="true" />
</Host>
```

---

## 8. 최종 운영 체크리스트
1. 설정 변경 후: `sudo systemctl restart tomcat10`
2. 서비스 확인: `netstat -an | grep 8080`
3. 실시간 감시: `tail -f /var/log/tomcat10/catalina.out`
