# 🚀 Apache Tomcat 설치 및 운영 가이드

이 문서는 Linux(Ubuntu/WSL2) 환경에서의 Tomcat 설치, 운영 명령어, 폴더 구조 및 실무 트러블슈팅 사례를 정리한 문서입니다.

---

## 1. Tomcat 설치 (Ubuntu/WSL2)

### 🛠️ 사전 준비 (JDK 및 네트워크 도구 설치)
```bash
# 1. 저장소 업데이트 및 universe 활성화
sudo add-apt-repository universe
sudo apt update

# 2. JDK 및 net-tools 설치 (네트워크 상태 확인용)
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

| 파일명 | 역할 및 주요 설정 내용 |
| :--- | :--- |
| **`server.xml`** | 서버 포트(8080), 엔진 설정, **Connector(접근 제어/IP 포워딩)** 설정. |
| **`web.xml`** | 모든 웹 앱의 공통 설정. 세션 시간, MIME 타입, **설정 파일 경로** 지정. |
| **`context.xml`** | 개별 프로젝트의 **DocBase(소스 경로)** 및 DB 커넥션 풀 설정. |

---

## 5. [실무] 특정 경로의 소스로 배포하기 (DocBase)
WAR 파일로 묶어서 `webapps`에 넣지 않고, 서버의 특정 폴더에 있는 소스를 바로 구동하고 싶을 때 설정합니다.

**파일 위치:** `/etc/tomcat10/server.xml` (또는 `context.xml`)
```xml
<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
    <Context path="" docBase="/home/user/my_project/src" reloadable="true" />
</Host>
```

---

## 6. [트러블슈팅] 외부 IP가 아닌 Proxy IP만 수집될 때
L4 스위치나 Proxy 서버를 거칠 때, `HttpServletRequest`에서 외부 실제 IP 대신 내부 Proxy IP만 찍히는 문제를 해결하는 설정입니다.

**파일 위치:** `/etc/tomcat10/server.xml`
```xml
<Valve className="org.apache.catalina.valves.RemoteIpValve"
       remoteIpHeader="x-forwarded-for"
       proxiesHeader="x-forwarded-by"
       protocolHeader="x-forwarded-proto" />
```
* **효과:** 위 설정을 추가하면 `request.getRemoteAddr()` 호출 시 Proxy IP가 아닌 **사용자의 실제 외부 IP**를 반환하게 되어 통계 테이블에 정확한 데이터가 쌓입니다.

---

## 7. 프로젝트 배포 프로세스
1. 소스 수정 후 서버의 지정된 경로(`docBase`)에 반영합니다.
2. `sudo systemctl restart tomcat10` 명령어로 반영 사항을 확인합니다.
3. `netstat -an | grep 8080`으로 서비스 상태를 최종 체크합니다.
