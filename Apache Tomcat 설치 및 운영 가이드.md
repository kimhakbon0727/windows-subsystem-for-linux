# 🚀 Apache Tomcat 설치 및 운영 가이드

이 문서는 Linux(Ubuntu/WSL2) 환경에서의 Tomcat 설치 방법, 주요 명령어, 그리고 폴더 구조를 정리한 문서입니다.

---

## 1. Tomcat 설치 (Ubuntu/WSL2)

### 🛠️ 사전 준비 (JDK 설치)
Tomcat은 Java 기반으로 구동되므로 JDK 설치가 필수입니다.
```bash
# 패키지 업데이트
sudo apt update

# JDK 설치 (기본 버전)
sudo apt install default-jdk -y

# 설치 확인 (JDK 1.8 또는 11 이상 권장)
java -version
```

### 📥 Tomcat 설치
```bash
# Tomcat 9 버전 및 관리자 도구 설치
sudo apt install tomcat9 tomcat9-admin -y
```

---

## 2. 주요 운영 명령어 (Service Management)

실무에서 가장 많이 사용하는 서비스 제어 명령어입니다.

| 기능 | 명령어 |
| :--- | :--- |
| **서비스 시작** | `sudo systemctl start tomcat9` |
| **서비스 중지** | `sudo systemctl stop tomcat9` |
| **서비스 재시작** | `sudo systemctl restart tomcat9` |
| **상태 실시간 확인** | `sudo systemctl status tomcat9` |
| **실시간 로그 모니터링** | `tail -f /var/log/tomcat9/catalina.out` |
| **8080 포트 점유 확인** | `netstat -an | grep 8080` |

---

## 3. Tomcat 폴더 구조 (Directory Structure)

Tomcat의 주요 데이터와 설정이 담긴 경로별 역할입니다.

| 폴더명 | 용도 | 주요 내용 |
| :--- | :--- | :--- |
| **`bin`** | **실행 파일** | `startup.sh`, `shutdown.sh` 등 구동 스크립트 |
| **`conf`** | **설정 파일** | `server.xml`(포트), `web.xml`(서블릿 설정) |
| **`lib`** | **라이브러리** | Tomcat 및 웹 앱이 공통으로 사용하는 JAR 파일 |
| **`logs`** | **로그 저장** | 에러 및 접속 기록 (`catalina.out` 등) |
| **`webapps`** | **배포 경로** | 프로젝트 WAR 파일이 위치하며 자동 배포됨 |
| **`work`** | **작업 공간** | JSP가 자바 소스로 변환되어 컴파일된 클래스 저장 |

---

## 4. 실무 핵심 설정 가이드

### ⚙️ 포트 번호 변경 (기본 8080 -> 80 등)
```bash
# 설정 파일 열기
sudo nano /etc/tomcat9/server.xml
```
* 파일 내용 중 `<Connector port="8080" ... />` 부분의 숫자를 원하는 포트로 수정 후 저장합니다.

---

## 5. 프로젝트 배포 프로세스 (Deployment)

1. 개발 도구(Eclipse/IntelliJ)에서 프로젝트를 **`.war`** 파일로 빌드합니다.
2. 배포 경로인 `/var/lib/tomcat9/webapps/` 폴더로 해당 파일을 복사합니다.
3. 톰캣이 실행 중이면 자동으로 압축이 풀리며 배포가 완료됩니다.
4. 브라우저에서 `http://서버IP:포트/파일명`으로 접속을 확인합니다.
