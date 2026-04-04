# 📧 Bon Mail System

MariaDB 기반 메일 발송/수신 웹 애플리케이션  
Spring MVC + JSP + Bootstrap으로 구성된 전통적인 Java 웹 프로젝트

---

## 🖥️ 시스템 환경

| 항목 | 내용 |
|------|------|
| OS | Ubuntu (WSL2 on Windows) |
| WAS | Apache Tomcat 10.1 |
| DB | MariaDB |
| 언어 | Java 17 |
| 프레임워크 | Spring MVC 6.1.4 |
| 빌드 | Maven |
| 포트 | 공유기 8888 → 서버 8080 (포트포워딩) |

---

## 🗂️ 프로젝트 구조

```
mailproject/
├── pom.xml
├── db/
│   └── init.sql                          # DB 초기화 SQL
└── src/
    └── main/
        ├── java/com/bon/mail/
        │   ├── controller/
        │   │   ├── AuthController.java   # 로그인/로그아웃
        │   │   └── MailController.java   # 메일 발송/수신
        │   ├── dao/
        │   │   ├── UserDao.java          # 사용자 DB 접근
        │   │   ├── MailQueueDao.java     # 발송 큐 DB 접근
        │   │   └── MailReceivedDao.java  # 수신 메일 DB 접근
        │   ├── model/
        │   │   ├── User.java
        │   │   ├── MailQueue.java
        │   │   └── MailReceived.java
        │   ├── service/
        │   │   ├── MailService.java      # 메일 비즈니스 로직
        │   │   └── SmtpConfigService.java# SMTP 설정 관리
        │   ├── scheduler/
        │   │   └── MailBatchScheduler.java # 1분마다 배치 발송
        │   └── util/
        │       └── SessionFilter.java    # 로그인 세션 체크
        ├── resources/
        │   └── logback.xml              # 로그 설정
        └── webapp/
            ├── index.jsp
            └── WEB-INF/
                ├── web.xml
                ├── applicationContext.xml
                ├── dispatcher-servlet.xml
                └── views/
                    ├── login.jsp
                    ├── header.jsp
                    ├── inbox.jsp
                    ├── mailDetail.jsp
                    ├── compose.jsp
                    ├── sent.jsp
                    └── settings.jsp
```

---

## 🗄️ DB 구조

### SSH 터널링 설정
```
외부 서버: 218.233.210.80:2288 (SSH)
로컬 포트: localhost:3306 (MariaDB)
```

SSH 터널링 명령어:
```bash
ssh -L 3306:localhost:3306 유저명@218.233.210.80 -p 2288 -N &
```

### 테이블 구성

| 테이블 | 설명 |
|--------|------|
| `users` | 사용자 계정 |
| `mail_queue` | 발송 대기 큐 (PENDING/SENDING/SENT/FAILED) |
| `mail_received` | 수신 메일함 |
| `smtp_config` | SMTP 서버 설정 |

### DB 초기화
```bash
sudo mysql -u root < db/init.sql
```

### MariaDB 계정 설정
```sql
CREATE DATABASE IF NOT EXISTS maildb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS 'bon'@'localhost' IDENTIFIED BY '48200';
GRANT ALL PRIVILEGES ON maildb.* TO 'bon'@'localhost';
FLUSH PRIVILEGES;
```

---

## ⚙️ 주요 설정

### applicationContext.xml - DB 연결
```xml
<property name="jdbcUrl" value="jdbc:mariadb://localhost:3306/maildb?useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Seoul"/>
<property name="username" value="bon"/>
<property name="password" value="48200"/>
```

### SMTP 설정
로그인 후 **SMTP 설정** 메뉴에서 웹에서 직접 설정 가능

| 서비스 | 호스트 | 포트 | TLS |
|--------|--------|------|-----|
| Gmail | smtp.gmail.com | 587 | ✅ |
| Naver | smtp.naver.com | 587 | ✅ |
| Daum | smtp.daum.net | 465 | SSL |
| Outlook | smtp.office365.com | 587 | ✅ |

> Gmail 사용 시 **앱 비밀번호** 발급 필요 (2단계 인증 후)

---

## 🚀 빌드 및 배포

### 1. 의존성 설치
```bash
sudo apt install maven unzip -y
```

### 2. 빌드
```bash
cd ~/mailproject
mvn clean package
```

### 3. Tomcat 배포
```bash
sudo rm -rf /var/lib/tomcat10/webapps/mailsystem
sudo rm -f /var/lib/tomcat10/webapps/mailsystem.war
sudo cp target/mailsystem.war /var/lib/tomcat10/webapps/
sudo systemctl restart tomcat10
```

### 4. 로그 확인
```bash
sudo tail -f /var/lib/tomcat10/logs/catalina.out
```

---

## 🌐 접속 정보

```
URL: http://218.233.210.80:8888/mailsystem/login
ID : bon
PW : 48200
```

---

## 📋 주요 기능

| 기능 | 설명 |
|------|------|
| 로그인/로그아웃 | 세션 기반 인증 |
| 받은 메일함 | 수신 메일 목록 / 읽음 처리 / 삭제 |
| 메일 쓰기 | TO / CC / BCC / HTML 모드 지원 |
| 보낸 메일함 | 발송 상태 확인 (PENDING/SENT/FAILED) |
| 배치 발송 | 1분마다 자동 발송 스케줄러 |
| SMTP 설정 | DB 저장 방식으로 웹에서 동적 설정 |

---

## 🔧 트러블슈팅 기록

### 1. Tomcat 10 Jakarta 호환 문제
- **증상**: `javax/servlet/ServletContextListener` ClassNotFound
- **원인**: Tomcat 10부터 `javax.*` → `jakarta.*` 로 변경됨
- **해결**: pom.xml 의존성 전체를 jakarta 버전으로 교체, Spring 6.x 로 업그레이드

### 2. `-parameters` 컴파일러 플래그 누락
- **증상**: `Name for argument of type [String] not specified`
- **해결**: pom.xml maven-compiler-plugin에 `<arg>-parameters</arg>` 추가

### 3. DB 접속 권한 오류
- **증상**: `Access denied for user 'bon'@'localhost'`
- **원인**: bon 계정이 `%` 호스트로만 등록되어 localhost 접근 불가
- **해결**: `GRANT ALL PRIVILEGES ON maildb.* TO 'bon'@'localhost'` 실행

### 4. Boolean → Number 캐스팅 오류
- **증상**: `ClassCastException: Boolean cannot be cast to Number`
- **원인**: MariaDB JDBC가 `TINYINT(1)`을 Boolean으로 반환
- **해결**: instanceof 체크 후 분기 처리

### 5. SHA2 함수 JDBC 호환 문제
- **증상**: DB 직접 쿼리는 성공하는데 Java에서 null 반환
- **원인**: JDBC에서 `SHA2()` 함수 결과가 다르게 처리됨
- **해결**: Java 코드에서 `MessageDigest`로 직접 SHA-256 해시 계산

---

## 📦 사용 라이브러리

| 라이브러리 | 버전 |
|-----------|------|
| Spring MVC | 6.1.4 |
| MariaDB JDBC | 3.1.4 |
| HikariCP | 5.0.1 |
| Angus Mail (Jakarta) | 2.0.2 |
| Jakarta Servlet API | 6.0.0 |
| JSTL (Jakarta) | 3.0.1 |
| Jackson | 2.14.2 |
| Logback | 1.2.11 |
