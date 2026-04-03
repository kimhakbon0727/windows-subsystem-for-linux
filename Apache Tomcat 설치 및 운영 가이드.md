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

---

## 4. 핵심 설정 파일 상세 설명 및 실무 팁 (`conf/`)

### ① server.xml (서버 엔진 설정)
* **역할:** 서버 포트, 프로토콜, 엔진 설정을 담당합니다.
* **실무 팁 (IP 접근 제어):** 특정 IP만 접속을 허용(방화벽 역할)하다가 전체 허용으로 바꿀 때 사용합니다.
    * **특정 IP만 허용 시:** `<RemoteAddrValve allow="127\.0\.0\.1|192\.168\.0\.10" />`
    * **모두 허용 시:** 위 설정을 주석 처리(``)하거나 `allow=".*"`로 수정합니다.

### ② web.xml (웹 애플리케이션 설정)
* **역할:** 서블릿 매핑, 세션 시간, 파일 경로 등을 설정합니다.
* **실무 팁 (파일 경로 변경):** 외부 리소스나 설정 파일의 경로를 지정할 때 사용합니다.
    ```xml
    <context-param>
        <param-name>configLocation</param-name>
        <param-value>/etc/myproject/config.properties</param-value>
    </context-param>
    ```
    * 배포 환경(운영/개발)에 따라 위 `param-value` 값을 실제 서버 경로로 수정합니다.

### ③ context.xml (공통 자원 설정)
* **역할:** DB 커넥션 풀(DBCP) 등 프로젝트 전역에서 쓸 자원을 정의합니다.

### ④ tomcat-users.xml (권한 설정)
* **역할:** 매니저 페이지 접속용 ID/PW를 설정합니다.

---

## 5. 실무 핵심 설정 가이드

### ⚙️ 포트 번호 변경 (기본 8080 -> 80 등)
```bash
# 설정 파일 열기
sudo nano /etc/tomcat10/server.xml
```
* `<Connector port="8080" ... />` 부분을 수정 후 서비스를 재시작합니다.

### 🛡️ IP 접근 제한 해제 (RemoteAddrValve)
만약 특정 IP만 접속 가능하게 막혀 있다면, `context.xml`이나 `server.xml` 내부의 `Valve` 설정을 확인하세요.
* `allow` 속성에 들어있는 IP 패턴을 `.*`로 바꾸면 모든 IP에서 접속이 가능해집니다.

---

## 6. 프로젝트 배포 프로세스 (Deployment)

1. 프로젝트를 **`.war`** 파일로 빌드합니다.
2. `/var/lib/tomcat10/webapps/` 폴더로 복사합니다.
   * `sudo cp project.war /var/lib/tomcat10/webapps/`
3. 톰캣이 실행 중이면 자동으로 압축이 풀리며 배포됩니다.
4. 브라우저에서 `http://서버IP:포트/파일명`으로 접속을 확인합니다.
