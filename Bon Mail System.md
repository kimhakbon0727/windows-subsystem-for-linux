# 📧 Bon Mail System

MariaDB 기반 메일 발송/수신 웹 애플리케이션  
Spring MVC + JSP + Bootstrap으로 구성된 전통적인 Java 웹 프로젝트

---

## 📌 목차

1. [시스템 환경](#-시스템-환경)
2. [프로젝트 구조](#-프로젝트-구조)
3. [DB 구조](#-db-구조)
4. [최초 설치 및 배포](#-최초-설치-및-배포-과정)
5. [주요 기능](#-주요-기능)
6. [주요 설정](#-주요-설정)
7. [사용자 매뉴얼](#-사용자-매뉴얼)
8. [트러블슈팅](#-트러블슈팅-상세-기록)
9. [재빌드 및 배포 명령어](#-재빌드-및-재배포-표준-명령어)
10. [로그 확인 명령어](#-로그-확인-명령어-모음)
11. [사용 라이브러리](#-사용-라이브러리)

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

---

## 🚀 최초 설치 및 배포 과정

### 1. 필수 패키지 설치

```bash
sudo apt install unzip -y
sudo apt install maven -y
```

### 2. 소스 압축 해제

```bash
unzip mailsystem-source.zip -d ~/
cd ~/mailproject
```

### 3. DB 초기화

```bash
sudo mysql -u root
```

```sql
CREATE DATABASE IF NOT EXISTS maildb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS 'bon'@'localhost' IDENTIFIED BY '48200';
GRANT ALL PRIVILEGES ON maildb.* TO 'bon'@'localhost';
FLUSH PRIVILEGES;
```

```bash
sudo mysql -u root maildb < ~/init.sql
```

### 4. Tomcat 경로 확인

```bash
ps -ef | grep tomcat
# -Dcatalina.base=/var/lib/tomcat10   ← WAR 배포 경로
# -Dcatalina.home=/usr/share/tomcat10 ← Tomcat 설치 경로
```

### 5. Tomcat 포트 확인

```bash
sudo cat /var/lib/tomcat10/conf/server.xml | grep -i "port\|connector"
# → port="8080" 확인 / 공유기에서 8888 → 8080 포트포워딩
```

### 6. 빌드 및 배포

```bash
cd ~/mailproject && \
mvn clean package && \
sudo rm -rf /var/lib/tomcat10/webapps/mailsystem && \
sudo rm -f /var/lib/tomcat10/webapps/mailsystem.war && \
sudo cp target/mailsystem.war /var/lib/tomcat10/webapps/ && \
sudo systemctl restart tomcat10
```

배포 확인:
```bash
ls /var/lib/tomcat10/webapps/
# → ROOT  mailsystem  mailsystem.war
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
| 모바일 지원 | Bootstrap 반응형 - 햄버거 메뉴(☰) |

---

## ⚙️ 주요 설정

### applicationContext.xml - DB 연결
```xml
<property name="jdbcUrl" value="jdbc:mariadb://localhost:3306/maildb?useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Seoul"/>
<property name="username" value="bon"/>
<property name="password" value="48200"/>
```

### SMTP 서버별 설정값

| 서비스 | 호스트 | 포트 | 방식 | STARTTLS |
|--------|--------|------|------|----------|
| Gmail | smtp.gmail.com | 587 | TLS | ✅ 체크 |
| Naver | smtp.naver.com | 465 | SSL | ❌ 체크 해제 |
| Daum | smtp.daum.net | 465 | SSL | ❌ 체크 해제 |
| Outlook | smtp.office365.com | 587 | TLS | ✅ 체크 |

> ⚠️ 포트 465(SSL) 사용 시 STARTTLS 체크하면 안 됩니다!

---

## 📖 사용자 매뉴얼

---

### 1️⃣ 접속 및 로그인

**접속 URL**
```
http://218.233.210.80:8888/mailsystem/login
```

**기본 계정 정보**

| 항목 | 값 |
|------|-----|
| 아이디 | bon |
| 비밀번호 | 48200 |

**로그인 방법**
1. 브라우저에서 위 URL 접속
2. 아이디 `bon`, 비밀번호 `48200` 입력
3. 로그인 버튼 클릭
4. 성공 시 받은 메일함으로 자동 이동

**참고 사항**
- 로그인 실패 시 `아이디 또는 비밀번호가 올바르지 않습니다` 메시지 표시
- 세션은 **30분** 후 자동 만료, 만료 시 로그인 페이지로 이동
- 모바일에서도 동일한 URL로 접속 가능

---

### 2️⃣ 받은 메일함

**화면 이동**
- 로그인 직후 자동 이동
- 상단 네비게이션 → **받은메일함** 클릭

**화면 구성**
- 상단: 전체 메일 수 / 안읽은 메일 수 표시
- 목록: 보낸사람 / 제목 / 받은시간 표시
- 안읽은 메일은 **굵은 글씨 + N 뱃지** 로 표시
- 읽은 메일은 회색으로 표시

**메일 읽기**
1. 목록에서 읽고 싶은 메일 클릭
2. 메일 상세 화면으로 이동
3. 자동으로 **읽음** 처리됨

**메일 상세 화면 기능**
- `받은메일함으로` 버튼 → 목록으로 돌아가기
- `답장` 버튼 → 메일 쓰기 화면으로 이동 (받는이/제목 자동 입력)
- `삭제` 버튼 → 확인 후 소프트 삭제 (완전 삭제 아님)

---

### 3️⃣ 메일 쓰기

**화면 이동**
- 상단 네비게이션 → **메일쓰기** 클릭

**입력 항목**

| 항목 | 설명 | 필수 |
|------|------|------|
| 받는이 (TO) | 수신자 이메일 주소 | ✅ |
| 참조 (CC) | `참조(CC)` 버튼 클릭 후 입력 | ❌ |
| 숨은참조 (BCC) | `숨은참조(BCC)` 버튼 클릭 후 입력 | ❌ |
| 제목 | 메일 제목 | ✅ |
| 내용 | 메일 본문 | ✅ |

**사용 방법**
1. **받는이** 에 수신자 이메일 입력
   - 여러 명 보낼 때: 쉼표(,)로 구분 → `a@gmail.com, b@naver.com`
2. 필요 시 `참조(CC)` / `숨은참조(BCC)` 버튼 클릭하여 입력
3. **제목** 입력
4. **내용** 입력
5. **HTML 모드** 토글 켜면 HTML 태그 사용 가능
6. `발송 큐 등록` 버튼 클릭

**발송 방식**
- 즉시 발송이 아닌 **발송 큐에 등록** 됨
- 배치 스케줄러가 **1분마다** 자동으로 실제 발송 처리
- 발송 결과는 **보낸 메일함** 에서 확인 가능

> ⚠️ 발송 전 반드시 **SMTP 설정** 이 완료되어 있어야 합니다.

---

### 4️⃣ 보낸 메일함

**화면 이동**
- 상단 네비게이션 → **보낸메일함** 클릭

**발송 상태 설명**

| 상태 | 색상 | 의미 |
|------|------|------|
| `PENDING` | 🟡 노란색 | 발송 대기 중 (1분 내 발송 예정) |
| `SENDING` | 🔵 파란색 | 현재 발송 중 |
| `SENT` | 🟢 초록색 | 발송 완료 |
| `FAILED` | 🔴 빨간색 | 발송 실패 (SMTP 설정 확인 필요) |

**확인 방법**
1. 메일 쓰기 후 보낸 메일함 이동
2. 상태가 `PENDING` → 1분 내 자동 발송
3. 발송 후 상태가 `SENT` 로 변경되면 성공
4. `FAILED` 인 경우 SMTP 설정 확인 후 재시도
5. `새로고침` 버튼으로 최신 상태 확인

> 재시도 횟수가 **3회** 초과하면 더 이상 발송 시도하지 않습니다.

---

### 5️⃣ SMTP 설정

**화면 이동**
- 상단 네비게이션 → **SMTP설정** 클릭

**설정 항목**

| 항목 | 설명 | 예시 |
|------|------|------|
| SMTP 호스트 | 메일 서버 주소 | smtp.naver.com |
| SMTP 포트 | 메일 서버 포트 | 465 |
| SMTP 계정 | 발송에 사용할 이메일 | your@naver.com |
| SMTP 비밀번호 | 애플리케이션 비밀번호 | xxxxxxxxxxxxxxxx |
| STARTTLS | 암호화 방식 선택 | SSL(465) 사용 시 체크 해제 |

---

#### 📮 Naver 설정 방법 (실제 테스트 완료)

**1단계 - 네이버 POP3/SMTP 허용 설정**
- **PC 브라우저**에서만 설정 가능 (모바일 앱 불가)
- `https://mail.naver.com` 접속
- 환경설정 → **POP3/IMAP 설정** 탭 클릭
- **POP3/SMTP 사용** → **사용함** 선택 후 **저장** 클릭

**2단계 - 애플리케이션 비밀번호 발급**
- POP3/SMTP 설정 화면 하단 **`설정하기`** 버튼 클릭
- 네이버 2단계 인증 설정
- 애플리케이션 비밀번호 생성
- 생성된 비밀번호 복사해두기

> ⚠️ 네이버는 일반 로그인 비밀번호로 SMTP 접속 불가, 반드시 **애플리케이션 비밀번호** 사용

**3단계 - SMTP 설정 페이지 입력**

| 항목 | 값 |
|------|-----|
| SMTP 호스트 | smtp.naver.com |
| SMTP 포트 | **465** |
| SMTP 계정 | 네이버아이디@naver.com |
| SMTP 비밀번호 | 애플리케이션 비밀번호 |
| STARTTLS | ❌ **체크 해제** |

> ⚠️ 네이버는 포트 **465 (SSL)** 사용 → STARTTLS 체크하면 발송 실패!

---

#### 📮 Gmail 설정 방법

**1단계 - Gmail 앱 비밀번호 발급**
1. Google 계정 → 보안
2. 2단계 인증 활성화
3. 앱 비밀번호 → 앱: `메일`, 기기: `Windows 컴퓨터`
4. 생성된 16자리 비밀번호 복사

**2단계 - SMTP 설정 페이지 입력**

| 항목 | 값 |
|------|-----|
| SMTP 호스트 | smtp.gmail.com |
| SMTP 포트 | **587** |
| SMTP 계정 | Gmail 주소 |
| SMTP 비밀번호 | 앱 비밀번호 (16자리) |
| STARTTLS | ✅ **체크** |

---

### 6️⃣ 로그아웃

**방법**
- 상단 네비게이션 → **로그아웃** 클릭 (노란색)
- 또는 PC에서 우측 상단 계정명 클릭 → 로그아웃

**로그아웃 후**
- 세션 완전 삭제
- 로그인 페이지로 이동
- 뒤로가기 버튼으로 돌아와도 세션 없어 재로그인 필요

---

### 7️⃣ 모바일 사용법

**접속**
- 모바일 브라우저에서 동일 URL 접속
```
http://218.233.210.80:8888/mailsystem/login
```

**메뉴 이동**
- 상단 오른쪽 **햄버거 버튼(☰)** 탭
- 펼쳐진 메뉴에서 원하는 항목 탭
  - 받은메일함
  - 메일쓰기
  - 보낸메일함
  - SMTP설정
  - 로그아웃

> ⚠️ 네이버 POP3/SMTP 설정은 **모바일 앱에서 불가**, 반드시 PC 브라우저에서 설정해야 합니다.

---

### 8️⃣ 자주 묻는 질문 (FAQ)

**Q. 메일을 보냈는데 상대방이 못 받았어요.**

A. 아래 순서로 확인하세요.
1. 보낸메일함에서 상태가 `SENT` 인지 확인
2. `FAILED` 이면 SMTP 설정 확인
3. `PENDING` 이면 1분 기다린 후 새로고침
4. 상대방 스팸함 확인 요청

---

**Q. 네이버로 발송 시 FAILED가 돼요.**

A. 아래를 순서대로 확인하세요.
1. 네이버 메일 → 환경설정 → POP3/IMAP 설정 → **POP3/SMTP 사용함** 체크 여부
2. SMTP 포트가 **465** 인지 확인 (587 아님!)
3. STARTTLS가 **체크 해제** 되어 있는지 확인
4. 비밀번호가 **애플리케이션 비밀번호** 인지 확인 (일반 비밀번호 불가)
5. POP3/SMTP 설정은 **PC 브라우저**에서만 설정 가능

---

**Q. SMTP 설정을 저장했는데 메일이 FAILED가 돼요.**

A. 아래를 확인하세요.
- Gmail: 앱 비밀번호 사용 여부 (일반 비밀번호 불가)
- Naver: 포트 465, STARTTLS 체크 해제, 애플리케이션 비밀번호 사용 여부
- 포트 587이면 STARTTLS 체크, 포트 465이면 STARTTLS 체크 해제

---

**Q. 로그인이 안 돼요.**

A. 아래를 확인하세요.
- 아이디: `bon` (소문자)
- 비밀번호: `48200`
- SSH 터널링이 연결되어 있는지 확인
- 서버 로그 확인:
```bash
sudo tail -f /var/lib/tomcat10/logs/catalina.out
```

---

**Q. 페이지가 500 오류가 나요.**

A. Tomcat 로그를 확인하세요.
```bash
sudo cat /var/lib/tomcat10/logs/localhost.$(date +%Y-%m-%d).log
```

---

**Q. 메일 발송이 1분이 지나도 안 돼요.**

A. 배치 스케줄러 로그를 확인하세요.
```bash
sudo tail -f /var/lib/tomcat10/logs/catalina.out | grep -i "배치"
```
`배치 메일 발송 시작` 로그가 안 찍히면 Tomcat 재시작이 필요합니다.
```bash
sudo systemctl restart tomcat10
```

---

## 🔧 트러블슈팅 상세 기록

---

### ❌ 문제 1. Tomcat 10 Jakarta 호환 문제

**증상 - catalina.out 로그**
```
[crit] Context [/mailsystem] startup failed due to previous errors
```

**증상 - localhost.날짜.log**
```
java.lang.NoClassDefFoundError: javax/servlet/ServletContextListener
Caused by: java.lang.ClassNotFoundException: javax.servlet.ServletContextListener
```

**로그 확인 명령어**
```bash
sudo cat /var/lib/tomcat10/logs/localhost.$(date +%Y-%m-%d).log
```

**원인**
- Tomcat 10부터 `javax.*` → `jakarta.*` 패키지로 전환됨
- 기존 코드는 Spring 5.x + javax.servlet 기반으로 작성되어 충돌 발생

**해결 - pom.xml 수정**

Spring 버전 변경:
```xml
<!-- 변경 전 -->
<spring.version>5.3.23</spring.version>
<!-- 변경 후 -->
<spring.version>6.1.4</spring.version>
```

Servlet/JSP/JSTL 변경:
```xml
<!-- 변경 전 -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>javax.servlet.jsp-api</artifactId>
    <version>2.3.3</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>

<!-- 변경 후 -->
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.0.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>jakarta.servlet.jsp</groupId>
    <artifactId>jakarta.servlet.jsp-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.glassfish.web</groupId>
    <artifactId>jakarta.servlet.jsp.jstl</artifactId>
    <version>3.0.1</version>
</dependency>
<dependency>
    <groupId>jakarta.servlet.jsp.jstl</groupId>
    <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
    <version>3.0.0</version>
</dependency>
```

JavaMail 변경:
```xml
<!-- 변경 전 -->
<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>javax.mail</artifactId>
    <version>1.6.2</version>
</dependency>
<!-- 변경 후 -->
<dependency>
    <groupId>org.eclipse.angus</groupId>
    <artifactId>angus-mail</artifactId>
    <version>2.0.2</version>
</dependency>
```

Java 소스 파일 일괄 치환:
```bash
find src -name "*.java" -exec sed -i 's/javax\.servlet/jakarta.servlet/g' {} \;
find src -name "*.java" -exec sed -i 's/javax\.mail/jakarta.mail/g' {} \;
sed -i 's/javax\.servlet/jakarta.servlet/g' src/main/webapp/WEB-INF/web.xml
```

---

### ❌ 문제 2. `-parameters` 컴파일러 플래그 누락

**증상**
```
HTTP 상태 500
java.lang.IllegalArgumentException: Name for argument of type [java.lang.String]
not specified, and parameter name information not available via reflection.
Ensure that the compiler uses the '-parameters' flag.
```

**원인**
- Spring 6.x에서 `@RequestParam` 파라미터 이름 인식을 위해 `-parameters` 컴파일 플래그 필요

**해결 - pom.xml maven-compiler-plugin 수정**
```xml
<!-- 변경 전 -->
<configuration>
    <source>17</source>
    <target>17</target>
    <encoding>UTF-8</encoding>
</configuration>

<!-- 변경 후 -->
<configuration>
    <source>17</source>
    <target>17</target>
    <encoding>UTF-8</encoding>
    <compilerArgs>
        <arg>-parameters</arg>
    </compilerArgs>
</configuration>
```

---

### ❌ 문제 3. DB 접속 권한 오류

**증상**
```
HTTP 상태 500
Access denied for user 'bon'@'localhost' (using password: YES)
```

**원인 확인 쿼리**
```sql
sudo mysql -u root
SELECT user, host FROM mysql.user WHERE user='bon';
-- 결과: bon | %  (localhost 권한 없음!)
```

**해결**
```sql
GRANT ALL PRIVILEGES ON maildb.* TO 'bon'@'localhost' IDENTIFIED BY '48200';
FLUSH PRIVILEGES;

-- 확인
SELECT user, host FROM mysql.user WHERE user='bon';
-- bon | %
-- bon | localhost  ← 추가됨
```

---

### ❌ 문제 4. Boolean → Number 캐스팅 오류 (배치 스케줄러)

**증상 - catalina.out 로그**
```
java.lang.ClassCastException: class java.lang.Boolean cannot be cast to
class java.lang.Number
    at com.bon.mail.scheduler.MailBatchScheduler.processPendingMails(MailBatchScheduler.java:47)
```

**원인**
- MariaDB JDBC 드라이버가 `TINYINT(1)` 컬럼을 `Boolean` 타입으로 자동 변환
- `((Number) smtpConfig.get("use_tls")).intValue()` 에서 캐스팅 오류 발생

**해결 - MailBatchScheduler.java 수정**
```bash
sed -i 's/boolean useTls  = ((Number) smtpConfig.get("use_tls")).intValue() == 1;/Object tlsVal = smtpConfig.get("use_tls");\n        boolean useTls;\n        if (tlsVal instanceof Boolean) {\n            useTls = (Boolean) tlsVal;\n        } else {\n            useTls = ((Number) tlsVal).intValue() == 1;\n        }/' \
~/mailproject/src/main/java/com/bon/mail/scheduler/MailBatchScheduler.java

# 수정 확인
grep -n "useTls\|use_tls" ~/mailproject/src/main/java/com/bon/mail/scheduler/MailBatchScheduler.java
```

**해결 - SmtpConfigService.java 수정**
```bash
sed -i 's/defaults.put("use_tls", 1);/defaults.put("use_tls", true);/' \
~/mailproject/src/main/java/com/bon/mail/service/SmtpConfigService.java

# 수정 확인
grep -n "use_tls" ~/mailproject/src/main/java/com/bon/mail/service/SmtpConfigService.java
```

**정상 작동 로그**
```
=== 배치 메일 발송 시작 ===
HikariPool-1 - Starting...
HikariPool-1 - Added connection org.mariadb.jdbc.Connection@4d3179ae
HikariPool-1 - Start completed.
=== 배치 메일 발송 완료 - 성공:0, 실패:0 ===
```

---

### ❌ 문제 5. SHA2 함수 JDBC 호환 문제 (로그인 계속 실패)

**증상**
- 로그인 화면에서 `아이디 또는 비밀번호가 올바르지 않습니다` 반복 출력
- DB 직접 쿼리는 정상인데 Java 앱에서만 null 반환

**원인 파악 - DB 직접 쿼리 테스트**
```sql
USE maildb;
SELECT * FROM users WHERE username='bon' AND password=SHA2('48200',256) AND is_active=1;
-- 결과: 정상 조회됨 (1건)
```

**원인 파악 - 해시값 비교**
```sql
SELECT SHA2('48200', 256);
-- 54af9a7431410f0c9d6b4a47c93579f44a7d1344ba2cd7b1e6a8db782d7ce4c2
```

```bash
python3 -c "import hashlib; print(hashlib.sha256('48200'.encode()).hexdigest())"
# 54af9a7431410f0c9d6b4a47c93579f44a7d1344ba2cd7b1e6a8db782d7ce4c2 (동일)
```

**원인 파악 - AuthController에 디버그 로그 추가 후 확인**
```bash
sudo tail -f /var/lib/tomcat10/logs/catalina.out | grep -i "로그인\|조회"
```

로그 결과:
```
로그인 시도: username=[bon] password=[48200]
조회 결과: null
```

**원인**
- JDBC에서 `SHA2()` 함수를 `?` 파라미터 바인딩과 함께 사용 시 결과 불일치

**해결 - Java MessageDigest로 직접 SHA-256 계산**
```java
private String sha256(String input) {
    try {
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        byte[] hash = digest.digest(input.getBytes(StandardCharsets.UTF_8));
        StringBuilder hexString = new StringBuilder();
        for (byte b : hash) {
            String hex = Integer.toHexString(0xff & b);
            if (hex.length() == 1) hexString.append('0');
            hexString.append(hex);
        }
        return hexString.toString();
    } catch (Exception e) {
        throw new RuntimeException("SHA-256 오류", e);
    }
}

public User findByUsernameAndPassword(String username, String password) {
    String hashedPassword = sha256(password);
    String sql = "SELECT * FROM users WHERE username=? AND password=? AND is_active=1";
    List<User> list = jdbcTemplate.query(sql, rowMapper, username, hashedPassword);
    return list.isEmpty() ? null : list.get(0);
}
```

---

### ❌ 문제 6. SMTP 설정 페이지 500 오류

**증상**
```
HTTP 상태 500
jakarta.el.ELException: Cannot convert [true] of type [class java.lang.Boolean] to [class java.lang.Long]
```

**원인**
- settings.jsp에서 `${config.use_tls == 1}` 비교 시 Boolean을 Long으로 변환 시도

**해결 - settings.jsp 수정**
```jsp
<!-- 변경 전 -->
${config.use_tls == 1 ? 'checked' : ''}

<!-- 변경 후 -->
${config.use_tls == true ? 'checked' : ''}
```

---

### ❌ 문제 7. 모바일에서 메뉴 클릭 시 아무 동작 없음

**증상**
- 모바일에서 받은메일함 / 메일쓰기 / 보낸메일함 / SMTP설정 클릭해도 반응 없음
- PC에서는 정상 작동

**원인**
- 좌측 사이드바 메뉴가 모바일 화면에서 navbar 뒤에 z-index로 가려져 클릭이 안 됨

**해결**
- 모든 JSP(inbox, compose, sent, settings, mailDetail)에서 사이드바 컬럼 제거
- header.jsp에 모든 메뉴를 navbar로 통합
- 모바일에서 햄버거 버튼(☰) 탭 → 메뉴 펼쳐져서 정상 작동

---

### ❌ 문제 8. 네이버 SMTP 발송 실패

**증상**
- 네이버 SMTP 설정 후 메일 상태가 `FAILED`

**원인**
- 네이버는 포트 **465 (SSL)** 을 사용하는데 포트 587 + STARTTLS 조합으로 설정하면 실패
- 네이버는 일반 로그인 비밀번호로 SMTP 접속 불가, **애플리케이션 비밀번호** 필요
- 네이버 POP3/SMTP 설정은 **모바일 앱에서 불가**, PC 브라우저에서만 설정 가능

**올바른 네이버 SMTP 설정값**

| 항목 | 값 |
|------|-----|
| SMTP 호스트 | smtp.naver.com |
| SMTP 포트 | **465** |
| SMTP 계정 | 네이버아이디@naver.com |
| SMTP 비밀번호 | **애플리케이션 비밀번호** |
| STARTTLS | ❌ **체크 해제** |

**네이버 애플리케이션 비밀번호 발급 절차**
1. PC 브라우저에서 `https://mail.naver.com` 접속
2. 환경설정 → POP3/IMAP 설정 탭
3. POP3/SMTP 사용 → **사용함** 선택 후 저장
4. 하단 **`설정하기`** 버튼 클릭
5. 2단계 인증 설정
6. 애플리케이션 비밀번호 생성 후 복사
7. SMTP 설정 페이지에서 비밀번호란에 입력

---

## 🔄 재빌드 및 재배포 표준 명령어

```bash
cd ~/mailproject && \
mvn clean package && \
sudo rm -rf /var/lib/tomcat10/webapps/mailsystem && \
sudo rm -f /var/lib/tomcat10/webapps/mailsystem.war && \
sudo cp target/mailsystem.war /var/lib/tomcat10/webapps/ && \
sudo systemctl restart tomcat10
```

---

## 📊 로그 확인 명령어 모음

```bash
# Tomcat 전체 로그 실시간 확인
sudo tail -f /var/lib/tomcat10/logs/catalina.out

# 오늘 날짜 앱 오류 로그 확인
sudo cat /var/lib/tomcat10/logs/localhost.$(date +%Y-%m-%d).log

# 로그인 관련 로그만 필터
sudo tail -f /var/lib/tomcat10/logs/catalina.out | grep -i "로그인\|조회"

# 배치 스케줄러 로그 확인
sudo tail -f /var/lib/tomcat10/logs/catalina.out | grep -i "배치"

# Tomcat 프로세스 및 경로 확인
ps -ef | grep tomcat

# Tomcat 포트 확인
sudo cat /var/lib/tomcat10/conf/server.xml | grep -i "port\|connector"

# 배포된 WAR 확인
ls /var/lib/tomcat10/webapps/

# DB 사용자 권한 확인
sudo mysql -u root -e "SELECT user, host FROM mysql.user WHERE user='bon';"

# 테이블 확인
sudo mysql -u root -e "SHOW TABLES;" maildb

# 사용자 데이터 확인
sudo mysql -u root -e "SELECT id, username, LENGTH(password), is_active FROM users;" maildb

# 로그인 쿼리 직접 테스트
sudo mysql -u root -e "USE maildb; SELECT * FROM users WHERE username='bon' AND password=SHA2('48200',256) AND is_active=1;"
```

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
