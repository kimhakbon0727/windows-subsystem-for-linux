# 🚀 Apache Tomcat 설치 및 운영 가이드

이 문서는 Linux(Ubuntu/WSL2) 환경에서의 Tomcat 설치 방법, 주요 명령어, 그리고 폴더 구조를 정리한 문서입니다.

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
| **8080 포트 점유 확인** | netstat -an | grep 8080 |

---

## 3. Tomcat 주요 폴더 경로 (Directory Structure)

Ubuntu 패키지 관리자(`apt`)로 설치했을 때의 주요 경로입니다.

| 폴더명 | 용도 | 실제 경로 (Ubuntu 기준) |
| :--- | :--- | :--- |
| **`bin`** | **실행 파일** | `/usr/share/tomcat10/bin/` |
| **`conf`** | **설정 파일** | `/etc/tomcat10/` (`server.xml` 등) |
| **`lib`** | **라이브러리** | `/usr/share/tomcat10/lib/` |
| **`logs`** | **로그 저장** | `/var/log/tomcat10/` (`catalina.out` 등) |
| **`webapps`** | **배포 경로** | `/var/lib/tomcat10/webapps/` |
| **`work`** | **작업 공간** | `/var/cache/tomcat10/` (컴파일된 파일 저장) |

---

## 4. 실무 핵심 설정 가이드

### ⚙️ 포트 번호 변경 (기본 8080 -> 80 등)
```bash
# 설정 파일 열기
sudo nano /etc/tomcat10/server.xml
```
* 파일 내용 중 `<Connector port="8080" ... />` 부분의 숫자를 원하는 포트로 수정 후 서비스를 재시작(`restart`)합니다.

---

## 5. 프로젝트 배포 프로세스 (Deployment)

1. 개발 도구(Eclipse/IntelliJ)에서 프로젝트를 **`.war`** 파일로 빌드합니다.
2. 배포 경로인 `/var/lib/tomcat10/webapps/` 폴더로 해당 파일을 복사합니다.
   * 예: `sudo cp my-app.war /var/lib/tomcat10/webapps/`
3. 톰캣이 실행 중이면 자동으로 압축이 풀리며 배포가 완료됩니다.
4. 브라우저에서 `http://서버IP:포트/파일명`으로 접속을 확인합니다.
