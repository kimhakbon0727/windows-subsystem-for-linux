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
| **포트 점유 확인** | `netstat -an` |

---

## 3. Tomcat 주요 폴더 경로 (Directory Structure)

| 폴더명 | 용도 | 실제 경로 (Ubuntu 기준) |
| :--- | :--- | :--- |
| **`bin`** | **실행 파일** | `/usr/share/tomcat10/bin/` |
| **`conf`** | **설정 파일** | `/etc/tomcat10/` |
| **`webapps`** | **배포 경로** | `/var/lib/tomcat10/webapps/` |
| **`logs`** | **로그 저장** | `/var/log/tomcat10/` |

---

## 4. [상세] 핵심 설정 파일의 역할 (`conf/`)

| 파일명 | 역할 및 실무 활용 |
| :--- | :--- |
| **`server.xml`** | 포트 변경, SSL 설정, **Proxy IP 포워딩(RemoteIpValve)** 등 서버 엔진 설정. |
| **`web.xml`** | 전역 세션 시간, MIME 타입, **Spring 설정 파일 경로** 등 앱 공통 설정. |
| **`context.xml`** | **DocBase(특정 폴더 소스 연결)**, DB 커넥션 풀(DBCP) 자원 정의. |
| **`tomcat-users.xml`** | 매니저 페이지(/manager) 접속 계정 및 권한(Role) 설정. |

---

## 5. [상세] 로그 파일 분석하기 (`logs/`)


1. **`catalina.out`**
   * **설명:** 톰캣의 표준 출력 로그. 서버 시작/종료 메시지와 `System.out.println()` 로그가 모두 찍힘.
   * **예시:** `04-Apr-2026 10:00:00.123 INFO [main] Server startup in [1234] ms`

2. **`localhost_access_log.YYYY-MM-DD.txt`**
   * **설명:** HTTP 요청 로그. 접속 IP, 요청 시간, URL, 응답 코드 기록. **통계 테이블 분석용**으로 필수.
   * **예시:** `192.168.0.5 - - [04/Apr/2026:10:05:21] "GET /api/stats HTTP/1.1" 200 45`

3. **`catalina.YYYY-MM-DD.log`**
   * **설명:** 톰캣 엔진 자체에서 발생하는 이벤트 및 경고/오류 메시지 기록.

4. **`localhost.YYYY-MM-DD.log`**
   * **설명:** 특정 호스트(Localhost)에서 구동되는 웹 앱 내에서 발생하는 에러 메시지 중심 기록.

---

## 6. [실무] 특정 경로 소스로 배포 (DocBase 설정)
WAR 배포 대신 특정 경로의 소스 폴더를 직접 연결할 때 사용합니다.
**위치:** `/etc/tomcat10/server.xml`
```xml
<Host name="localhost" appBase="webapps" ...>
    <Context path="" docBase="/home/kim/workspace/my_project" reloadable="true" />
</Host>
```

---

## 7. [실무] 외부 실제 IP 수집 설정 (Proxy 환경)
L4나 Proxy를 거칠 때 실제 클라이언트 IP를 수집하여 통계 테이블에 반영하는 방법입니다.
**위치:** `/etc/tomcat10/server.xml`
```xml
<Valve className="org.apache.catalina.valves.RemoteIpValve"
       remoteIpHeader="x-forwarded-for"
       proxiesHeader="x-forwarded-by"
       protocolHeader="x-forwarded-proto" />
```

---

## 8. 최종 운영 체크리스트
1. `sudo systemctl restart tomcat10` (설정 변경 후 필수)
2. `netstat -an | grep 8080` (서비스 포트 오픈 확인)
3. `tail -f /var/log/tomcat10/catalina.out` (구동 시 에러 로그 확인)
