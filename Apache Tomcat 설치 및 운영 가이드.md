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

## 5. [실무] 로그 파일의 종류와 분석 (`logs/`)



| 파일명 | 설명 및 용도 | 로그 예시 (Example) |
| :--- | :--- | :--- |
| **`catalina.out`** | **가장 중요한 로그.** 톰캣 엔진의 시작/종료 로그와 `System.out.println()`으로 찍은 모든 표준 출력이 기록됩니다. | `INFO: Server startup in [1,234] ms` |
| **`localhost_access_log`** | **접속 기록.** 누가(IP), 언제, 어떤 URL에 접속했는지 기록됩니다. 통계 분석 시 가장 많이 활용됩니다. | `127.0.0.1 - - [04/Apr/2026:10:00:01] "GET /index.do HTTP/1.1" 200` |
| **`catalina.YYYY-MM-DD.log`** | 톰캣 자체 시스템 로그입니다. 엔진 구동 중 발생하는 경고나 에러를 일자별로 저장합니다. | `SEVERE: Error starting static Resources` |
| **`localhost.log`** | 개별 웹 애플리케이션에서 발생하는 에러나 로그가 기록됩니다. | `org.apache.catalina.core.StandardContext.startInternal...` |
| **`manager.log`** | Tomcat Manager 앱(/manager) 관련 로그입니다. | `INFO: Manager: list for context /` |

---

## 6. [실무] 특정 경로의 소스로 배포하기 (DocBase)
WAR 파일 배포가 아닌, 특정 폴더의 소스를 바로 연결하고 싶을 때 사용합니다.

**파일 위치:** `/etc/tomcat10/server.xml`
```xml
<Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">
    <Context path="" docBase="/home/user/workspace/my_project" reloadable="true" />
</Host>
```

---

## 7. [실무] 외부 실제 IP 수집 설정 (RemoteIpValve)
Proxy나 L4 장비를 거칠 때 실제 사용자 IP를 수집하기 위한 필수 설정입니다.

**파일 위치:** `/etc/tomcat10/server.xml`
```xml
<Valve className="org.apache.catalina.valves.RemoteIpValve"
       remoteIpHeader="x-forwarded-for"
       proxiesHeader="x-forwarded-by"
       protocolHeader="x-forwarded-proto" />
```
* 적용 후 `HttpServletRequest.getRemoteAddr()`를 통해 실제 외부 IP가 수집되는지 확인합니다.

---

## 8. 운영 체크리스트
1. 소스 반영 후 `sudo systemctl restart tomcat10` 실행.
2. `tail -f /var/log/tomcat10/catalina.out`으로 에러 여부 실시간 모니터링.
3. `netstat -an | grep 8080`으로 서비스 포트 활성화 확인.
