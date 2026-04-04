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
# MariaDB root 접속
sudo mysql -u root
```

```sql
-- DB 및 계정 생성
CREATE DATABASE IF NOT EXISTS maildb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS 'bon'@'localhost' IDENTIFIED BY '48200';
GRANT ALL PRIVILEGES ON maildb.* TO 'bon'@'localhost';
FLUSH PRIVILEGES;
```

```bash
# 테이블 생성
sudo mysql -u root maildb < ~/init.sql
```

### 4. Tomcat 경로 확인

```bash
# ps 명령어로 실제 경로 확인
ps -ef | grep tomcat
```

출력 결과에서 확인:
```
-Dcatalina.base=/var/lib/tomcat10   ← WAR 배포 경로
-Dcatalina.home=/usr/share/tomcat10 ← Tomcat 설치 경로
```

### 5. Tomcat 포트 확인

```bash
sudo cat /var/lib/tomcat10/conf/server.xml | grep -i "port\|connector"
# → port="8080" 확인
# 공유기에서 8888 → 8080 포트포워딩 설정
```

### 6. 빌드 및 배포 (표준 명령어)

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
# → ROOT  mailsystem  mailsystem.war 가 있어야 함
```

### 7. 접속 확인

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
| 모바일 지원 | Bootstrap 반응형 - 햄버거 메뉴(☰) |

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
- 기존 코드는 Spring 5.x + javax.servlet 기반 작성으로 충돌 발생

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

**증상 - 브라우저 화면**
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

**증상 - 브라우저**
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

**로그 확인 명령어**
```bash
sudo tail -f /var/lib/tomcat10/logs/catalina.out
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

**해결 - UserDao.java: Java MessageDigest로 직접 SHA-256 계산**
```bash
cat > ~/mailproject/src/main/java/com/bon/mail/dao/UserDao.java << 'EOF'
package com.bon.mail.dao;

import com.bon.mail.model.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;

import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.util.List;

@Repository
public class UserDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    private final RowMapper<User> rowMapper = (rs, rowNum) -> {
        User u = new User();
        u.setId(rs.getLong("id"));
        u.setUsername(rs.getString("username"));
        u.setPassword(rs.getString("password"));
        u.setEmail(rs.getString("email"));
        u.setDisplayName(rs.getString("display_name"));
        u.setActive(rs.getInt("is_active") == 1);
        java.sql.Timestamp ca = rs.getTimestamp("created_at");
        if (ca != null) u.setCreatedAt(ca.toLocalDateTime());
        return u;
    };

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

    public User findById(Long id) {
        List<User> list = jdbcTemplate.query("SELECT * FROM users WHERE id=?", rowMapper, id);
        return list.isEmpty() ? null : list.get(0);
    }

    public int insert(User user) {
        String sql = "INSERT INTO users (username, password, email, display_name) VALUES (?, ?, ?, ?)";
        return jdbcTemplate.update(sql, user.getUsername(), sha256(user.getPassword()), user.getEmail(), user.getDisplayName());
    }
}
EOF
```

---

### ❌ 문제 6. SMTP 설정 페이지 500 오류 (Boolean → Long 변환 오류)

**증상 - 브라우저**
```
HTTP 상태 500
jakarta.el.ELException: Cannot convert [true] of type [class java.lang.Boolean] to [class java.lang.Long]
```

**원인**
- settings.jsp에서 `${config.use_tls == 1}` 비교 시 Boolean을 Long으로 변환 시도

**해결 - settings.jsp 수정**
```bash
# use_tls 비교 조건 변경
sed -i "s/\${config.use_tls == 1 ? 'checked' : ''}/\${config.use_tls == true ? 'checked' : ''}/g" \
~/mailproject/src/main/webapp/WEB-INF/views/settings.jsp
```

또는 nano로 직접 수정:
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

**해결 - 사이드바 완전 제거, 상단 navbar로 통합**

모든 JSP(inbox, compose, sent, settings, mailDetail)에서 사이드바 컬럼 제거:
```bash
# 각 JSP 파일을 사이드바 없는 버전으로 교체
cat > ~/mailproject/src/main/webapp/WEB-INF/views/inbox.jsp << 'EOF'
# ... 사이드바 없는 버전
EOF

cat > ~/mailproject/src/main/webapp/WEB-INF/views/compose.jsp << 'EOF'
# ... 사이드바 없는 버전
EOF

# sent, settings, mailDetail 동일하게 수정
```

header.jsp에 모든 메뉴를 navbar로 통합:
```jsp
<ul class="navbar-nav me-auto">
    <li class="nav-item">
        <a class="nav-link" href=".../mail/inbox">받은메일함</a>
    </li>
    <li class="nav-item">
        <a class="nav-link" href=".../mail/compose">메일쓰기</a>
    </li>
    <li class="nav-item">
        <a class="nav-link" href=".../mail/sent">보낸메일함</a>
    </li>
    <li class="nav-item">
        <a class="nav-link" href=".../mail/settings">SMTP설정</a>
    </li>
</ul>
```

모바일에서 햄버거 버튼(☰) 탭 → 메뉴 펼쳐져서 정상 작동

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
