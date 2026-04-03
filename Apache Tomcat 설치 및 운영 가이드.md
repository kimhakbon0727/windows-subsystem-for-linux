# 🚀 Apache Tomcat 설치 및 운영 가이드

이 문서는 Linux(Ubuntu/WSL2) 환경에서의 Tomcat 설치 방법, 주요 명령어, 폴더 구조 및 핵심 설정 파일의 역할을 정리한 문서입니다.

---

## 1. Tomcat 설치 (Ubuntu/WSL2)

### 🛠️ 사전 준비 (JDK 및 네트워크 도구 설치)
Tomcat 구동을 위한 Java와 포트 확인을 위한 네트워크 도구를 먼저 설치합니다.
```bash
# 1. 패키지 업데이트 및 저장소 활성화
sudo add-apt-repository universe
sudo apt update

# 2. JDK 및 net-tools 설치 (netstat 사용을 위함)
sudo apt install default-jdk net-tools -y

# 3. 설치 확인
java -version
```

### 📥 Tomcat 설치 (Tomcat 10 기준)
최신 우분투 환경에서는 `tomcat10` 설치를 권장합니다.
```bash
# Tomcat 10 및 관리자 도구 설치
sudo apt install tomcat10 tomcat10-admin -y
```

---

## 2. 주요 운영 명령어 (Service Management)

| 기능 | 명령어 |
| :--- | :--- |
| **서비스 시작** | `sudo systemctl start tomcat10` |
| **서비스 중지** | `sudo systemctl stop tomcat10` |
| **서비스 재시작** | `sudo systemctl restart tomcat10` |
| **상태 실시간 확인** | `sudo systemctl status tomcat10` |
| **실시간 로그 모니터링** | `tail -f /var/log/tomcat10/catalina.out` |
| **8080 포트 점유 확인** | `netstat -an | grep 8080` |

---

## 3. Tomcat 주요 폴더 경로 (Directory Structure)



| 폴더명 | 용도 | 실제 경로 (Ubuntu 기준) |
| :--- | :--- | :--- |
| **`bin`** | **실행 파일** | `/usr/share/tomcat10/bin/` |
| **`conf`** | **설정 파일** | `/etc/tomcat10/` |
| **`lib`** | **라이브러리** | `/usr/share/tomcat10/lib/` |
| **`logs`** | **로그 저장** | `/var/log/tomcat10/` |
| **`webapps`** | **배포 경로** | `/var/lib/tomcat10/webapps/` |
| **`work`** | **작업 공간** | `/var/cache/tomcat10/` |

---

## 4. 핵심 설정 파일 상세 설명 (`conf/`)

톰캣의 동작을 제어하는 핵심 파일들의 역할입니다.

| 파일명 | 역할 및 주요 설정 내용 |
| :--- | :--- |
| **`server.xml`** | **가장 중요한 설정 파일.** 서버의 포트 번호(기본 8080), 커넥션 타임아웃, 엔진 및 호스트 설정, SSL(HTTPS) 인증서 적용 등을 수행합니다. |
| **`web.xml`** | **전역 웹 애플리케이션 설정.** 모든 웹 앱에 적용될 기본 서블릿 매핑, MIME 타입 정의, 세션 유지 시간(Session Timeout), 웰컴 파일 리스트 등을 설정합니다. |
| **`context.xml`** | **개별 웹 앱 공통 설정.** 데이터베이스 커넥션 풀(DBCP)과 같은 자원(Resource) 정의나 특정 웹 앱의 공통 속성을 정의할 때 사용합니다. |
| **`tomcat-users.xml`** | **사용자 권한 설정.** 톰캣 관리자 페이지(/manager, /host-manager)에 접속할 수 있는 ID/PW와 역할(Role)을 지정합니다. |
| **`logging.properties`** | **로그 출력 설정.** 톰캣 내부 로깅 시스템의 로그 레벨(DEBUG, INFO, ERROR 등)과 로그 파일 저장 형식을 지정합니다. |

---

## 5. 실무 핵심 설정 가이드

### ⚙️ 포트 번호 변경 (기본 8080 -> 80 등)
```bash
# 설정 파일 열기
sudo nano /etc/tomcat10/server.xml
```
* `<Connector port="8080" ... />` 부분을 찾아 숫자를 수정한 후 `sudo systemctl restart tomcat10` 명령으로 재시작합니다.

---

## 6. 프로젝트 배포 프로세스 (Deployment)

1. 개발 도구(Eclipse/IntelliJ)에서 프로젝트를 **`.war`** 파일로 빌드합니다.
2. 배포 경로인 `/var/lib/tomcat10/webapps/` 폴더로 해당 파일을 복사합니다.
   * 예: `sudo cp my-app.war /var/lib/tomcat10/webapps/`
3. 톰캣이 실행 중이면 자동으로 압축이 풀리며 배포가 완료됩니다.
4. 브라우저에서 `http://서버IP:포트/파일명`으로 접속을 확인합니다.
