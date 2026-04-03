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
| **포트 점유 확인** | `netstat -an` |

---

## 3. Tomcat 주요 폴더 경로 (Directory Structure)

| 폴더명 | 용도 | 실제 경로 (Ubuntu 기준) |
| :--- | :--- | :--- |
| **`CATALINA_HOME`** | 실행 바이너리 및 공통 라이브러리 | `/usr/share/tomcat10/` |
| **`CATALINA_BASE`** | 웹앱, 로그 등 사용자 데이터 | `/var/lib/tomcat10/` |
| **`conf`** | 설정 파일 | `/etc/tomcat10/` |
| **`webapps`** | 배포 경로 | `/var/lib/tomcat10/webapps/` |
| **`logs`** | 로그 저장 | `/var/log/tomcat10/` |

> 최초 설치 후 브라우저에서 서버에 접속하면 **기본 환영 페이지**가 표시됩니다.
> 이 페이지는 `/var/lib/tomcat10/webapps/ROOT/index.html` 파일에 의해 렌더링됩니다.
> 내 프로젝트를 루트(`/`)에 바로 띄우려면 이 `ROOT` 폴더를 교체하거나 `ROOT.war`로 배포하면 됩니다.

---

## 4. 핵심 설정 파일 상세 설명 (`conf/`)

| 파일명 | 역할 및 실무 활용 |
| :--- | :--- |
| **`server.xml`** | **서버 엔진 설정.** 포트 변경, **Connector(Proxy) 설정**, Valve 설정 등. |
| **`web.xml`** | **앱 공통 설정.** 세션 시간, 기본 응답 파일(`welcome-file-list`) 지정 등. |
| **`context.xml`** | **프로젝트 개별 설정.** **DocBase(소스 경로)** 연동 및 DB 자원 정의. |
| **`tomcat-users.xml`** | **관리자 계정 설정.** Manager, Host-Manager 접근 권한 부여. |

### 📌 기본 응답 파일 설정 (`web.xml`)

사용자가 경로만 입력했을 때(`http://localhost:8080/`) 서버가 찾을 파일의 우선순위를 지정합니다.
기본값은 아래와 같으며, 순서대로 탐색 후 첫 번째로 발견된 파일을 응답합니다.
```xml
<welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.jsp</welcome-file>
</welcome-file-list>
```

> `index`라는 이름은 역사적 관례입니다. 웹의 초기부터 "이 폴더의 목록(Index)을 보여주는 대표 파일"이라는 약속으로 굳어진 이름이며, 위 설정을 통해 다른 이름으로 자유롭게 변경할 수 있습니다.

---

## 5. WAR 파일 배포 원리 (Auto Deploy)

Tomcat은 `webapps` 디렉토리를 실시간으로 감시합니다. `server.xml`의 `<Host>` 태그에 아래 두 옵션이 기본 활성화되어 있기 때문입니다.

| 옵션 | 기본값 | 설명 |
| :--- | :--- | :--- |
| `unpackWARs` | `true` | `.war` 파일 감지 시 자동으로 압축 해제 |
| `autoDeploy` | `true` | 실행 중에도 파일 변화를 감지해 즉시 반영 |

### WAR 파일 이름에 따른 접속 경로 차이

| 파일명 | 배포 후 경로 | 접속 URL |
| :--- | :--- | :--- |
| `ROOT.war` | `webapps/ROOT/` | `http://localhost:8080/` |
| `myapp.war` | `webapps/myapp/` | `http://localhost:8080/myapp/` |

> **실무 팁:** 운영 서버에서는 보안을 위해 기본 `ROOT` 애플리케이션을 삭제하고, 본인 프로젝트를 `ROOT.war`로 배포하는 것이 일반적입니다.

---

## 6. [실무] 로그 파일 분석 (`logs/`)

- **`catalina.out`**: 톰캣 엔진 로그 및 표준 출력. (장애 시 최우선 확인)
- **`localhost_access_log`**: HTTP 요청 기록. (IP, 접속 URL 등 **통계 테이블 분석용**)

---

## 7. [실무] Proxy(Nginx/L4) 연동 시 필수 설정

웹 서버(Proxy) 뒷단에서 톰캣이 동작할 때, 리다이렉트 경로가 깨지거나 실제 IP가 누락되는 것을 방지하기 위한 설정입니다.

**위치:** `/etc/tomcat10/server.xml`

### ① Connector 설정 (포트 및 프록시 정보 명시)
```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           proxyName="www.yourdomain.com"
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

## 8. [실무] 특정 경로 소스 배포 (DocBase 설정)

WAR 배포 대신 특정 폴더의 소스를 직접 연결할 때 사용합니다.

**위치:** `/etc/tomcat10/server.xml`
```xml
<Host name="localhost" appBase="webapps" ...>
    <Context path="" docBase="/home/user/workspace/my_project" reloadable="true" />
</Host>
```

---

## 9. 최종 운영 체크리스트

1. 설정 변경 후 재시작: `sudo systemctl restart tomcat10`
2. 포트 리스닝 확인: `netstat -an | grep 8080`
3. 실시간 로그 감시: `tail -f /var/log/tomcat10/catalina.out`
