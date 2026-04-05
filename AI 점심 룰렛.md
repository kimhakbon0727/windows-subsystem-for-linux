# 🍽️ AI 점심 룰렛 (Lunch Roulette)

> Google Gemini AI가 점심 메뉴를 추천하고, Canvas 룰렛으로 오늘의 메뉴를 결정해주는 웹 애플리케이션

![Java](https://img.shields.io/badge/Java-17-orange?style=flat-square)
![Spring MVC](https://img.shields.io/badge/Spring%20MVC-6.1.4-green?style=flat-square)
![Tomcat](https://img.shields.io/badge/Tomcat-10.1-yellow?style=flat-square)
![Gemini](https://img.shields.io/badge/Gemini-2.0%20Flash-blue?style=flat-square)

---

## 📋 목차

- [프로젝트 소개](#-프로젝트-소개)
- [시스템 환경](#-시스템-환경)
- [프로젝트 구조](#-프로젝트-구조)
- [주요 기능](#-주요-기능)
- [개발 과정 및 트러블슈팅](#-개발-과정-및-트러블슈팅)
- [배포 방법](#-배포-방법)
- [API 설정](#-api-설정)

---

## 🎯 프로젝트 소개

매일 반복되는 "오늘 뭐 먹지?" 고민을 AI가 해결해주는 프로젝트입니다.
Google Gemini API를 통해 다양한 점심 메뉴를 추천받고, Canvas 기반의 룰렛을 돌려 오늘의 메뉴를 결정합니다.

---

## 🖥️ 시스템 환경

| 항목 | 내용 |
|------|------|
| OS | Ubuntu (WSL2 on Windows) |
| WAS | Apache Tomcat 10.1 |
| 언어 | Java 17 |
| 프레임워크 | Spring MVC 6.1.4 |
| 빌드 | Maven |
| AI API | Google Gemini 2.0 Flash |
| 포트 | 공유기 8888 → 서버 8080 (포트포워딩) |

---

## 📁 프로젝트 구조

```
lunch-roulette/
├── pom.xml
└── src/main/
    ├── java/com/lunch/
    │   ├── controller/
    │   │   └── LunchController.java     # Spring MVC 컨트롤러
    │   ├── service/
    │   │   └── GeminiService.java       # Gemini API 연동 서비스
    │   └── model/
    │       └── LunchMenu.java           # 메뉴 모델
    ├── webapp/
    │   ├── WEB-INF/
    │   │   ├── web.xml                  # Tomcat 10 / Jakarta EE 설정
    │   │   ├── spring-mvc.xml           # Spring MVC 설정
    │   │   └── views/
    │   │       └── roulette.jsp         # 룰렛 UI 메인 화면
    │   └── index.jsp                    # 루트 리다이렉트
    └── resources/
        └── application.properties       # Gemini API 키 설정
```

---

## ✨ 주요 기능

- 🤖 **AI 메뉴 추천** - Google Gemini 2.0 Flash API를 통해 매번 다양한 점심 메뉴 8가지 추천
- 🎰 **Canvas 룰렛** - 추천받은 메뉴를 룰렛에 자동으로 채워 easeOut 애니메이션으로 스핀
- 🎯 **당첨 메뉴 표시** - 룰렛이 멈추면 당첨 메뉴와 설명, 가격대 표시
- 🔄 **폴백 처리** - API 실패 시 내장된 기본 메뉴 8개로 자동 대체
- 📋 **메뉴 칩 목록** - 추천받은 메뉴를 칩 형태로 미리 확인 가능

---

## 🔧 개발 과정 및 트러블슈팅

### 1️⃣ 프로젝트 초기 설계

Spring MVC 6.1.4 + Tomcat 10.1 환경에서 Gemini API를 연동하는 구조로 설계했습니다.

- **브라우저** → JSP/HTML/JS로 UI 렌더링
- **LunchController** → `/lunch/suggest` 엔드포인트로 Ajax 요청 수신
- **GeminiService** → Java 11+ `HttpClient`로 Gemini API 호출
- **LunchMenuParser** → 응답 JSON 파싱 후 메뉴 리스트 반환
- **roulette.jsp** → Canvas API로 룰렛 그리기 및 애니메이션

---

### 2️⃣ WAR 배포

```bash
# 빌드
cd /home/bon/lunch-roulette
mvn clean package -DskipTests

# Tomcat webapps에 배포
sudo cp target/lunch-roulette.war /var/lib/tomcat10/webapps/

# Tomcat 재시작
sudo systemctl restart tomcat10

# 배포 로그 확인
sudo tail -f /var/lib/tomcat10/logs/catalina.out
```

접속 URL: `http://서버IP:8888/lunch-roulette/lunch`

---

### 3️⃣ 🐛 트러블슈팅 #1 - `Unexpected token '<'` JSON 파싱 오류

**증상**
```
❌ 메뉴 추천 실패: Unexpected token '<', "<!doctype "... is not valid JSON
```

**원인**
`application.properties`에 Gemini API 키가 `YOUR_GEMINI_API_KEY_HERE` 그대로 설정되어 있어 API 호출이 실패하고, 에러 HTML 페이지가 JSON으로 돌아옴

**해결**

[Google AI Studio](https://aistudio.google.com/apikey)에서 API 키 발급 후 적용:

```bash
sudo nano /var/lib/tomcat10/webapps/lunch-roulette/WEB-INF/classes/application.properties
```

```properties
gemini.api.key=발급받은_API_키
gemini.api.url=https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent
```

```bash
sudo systemctl restart tomcat10
```

---

### 4️⃣ 🐛 트러블슈팅 #2 - `-parameters` 컴파일 플래그 누락

**증상**
```
Name for argument of type [int] not specified, and parameter name information
not available via reflection. Ensure that the compiler uses the '-parameters' flag.
```

**원인**
Maven 컴파일 시 `-parameters` 플래그가 없어 `@RequestParam` 파라미터 이름을 리플렉션으로 읽지 못함

**해결**

`pom.xml`의 `maven-compiler-plugin`에 `-parameters` 플래그 추가:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <source>17</source>
        <target>17</target>
        <encoding>UTF-8</encoding>
        <compilerArgs>
            <arg>-parameters</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

재빌드 & 재배포:
```bash
cd /home/bon/lunch-roulette
mvn clean package -DskipTests && sudo cp target/lunch-roulette.war /var/lib/tomcat10/webapps/ && sudo systemctl restart tomcat10
```

---

### 5️⃣ 🐛 트러블슈팅 #3 - 메뉴 개수 표시 안 됨

**증상**
```
✅ 개 메뉴 준비 완료! 룰렛을 돌려보세요 🎰
```
숫자가 빠진 채로 표시됨

**원인**
JSP에서 `menus.length`를 사용했으나 응답 구조상 `data.count`를 사용해야 함

**해결**
```bash
sed -i 's/`✅ \${menus.length}개 메뉴 준비 완료! 룰렛을 돌려보세요 🎰`/`✅ 메뉴 준비 완료! 룰렛을 돌려보세요 🎰`/' \
  /home/bon/lunch-roulette/src/main/webapp/WEB-INF/views/roulette.jsp
```

개수 표시 대신 깔끔하게 제거하는 방향으로 결정

---

### 6️⃣ 🐛 트러블슈팅 #4 - 메뉴 추천 누를 때마다 동일한 메뉴

**증상**
AI 메뉴 추천받기 버튼을 눌러도 매번 같은 메뉴가 나옴

**원인**
Gemini API `temperature` 값이 낮고, 프롬프트에 다양성을 유도하는 내용이 없어 캐싱처럼 동일한 응답 반환

**해결**

temperature 값을 최대인 2.0으로 올리고, 프롬프트에 현재 시각을 포함해 매번 다른 응답 유도:

```bash
# temperature 수정
sed -i 's/generationConfig.put("temperature", 1.0)/generationConfig.put("temperature", 2.0)/' \
  /home/bon/lunch-roulette/src/main/java/com/lunch/service/GeminiService.java
```

`GeminiService.java`의 `buildPrompt` 메서드에 현재 시각 추가:

```java
private String buildPrompt(int count) {
    String now = java.time.LocalDateTime.now().toString();
    return String.format("""
            현재 시각은 %s 입니다.
            오늘 점심 메뉴 %d가지를 추천해 주세요.
            이전과 겹치지 않게 매번 다양하고 새로운 메뉴를 추천해 주세요.
            한국 직장인이 즐겨먹는 메뉴 위주로, 다양한 종류(한식, 중식, 양식, 일식 등)를 섞어서 추천해 주세요.

            반드시 아래 JSON 배열 형식으로만 답변하세요.

            [
              {
                "name": "메뉴 이름",
                "description": "한 줄 설명 (15자 이내)",
                "price": "예상 가격대 (예: 8,000~10,000원)"
              }
            ]
            """, now, count);
}
```

---

### 7️⃣ 멀티 WAR 환경 관리

기존에 `mailsystem.war` (스케줄러 포함)와 `lunch-roulette.war` 두 개가 같은 Tomcat에 배포되어 있었습니다.

두 WAR는 컨텍스트 경로로 분리되어 포트 하나를 공유합니다:
- `http://서버:8888/mailsystem/`
- `http://서버:8888/lunch-roulette/`

용량 및 로그 관리를 위해 mailsystem은 별도로 관리하기로 결정:

```bash
# 백업 후 제거
sudo cp /var/lib/tomcat10/webapps/mailsystem.war ~/mailsystem_backup.war
sudo rm -f /var/lib/tomcat10/webapps/mailsystem.war
sudo rm -rf /var/lib/tomcat10/webapps/mailsystem/

# 복구 시
sudo cp ~/mailsystem_backup.war /var/lib/tomcat10/webapps/
```

---

## 🚀 배포 방법

### 사전 준비

```bash
# Java 17 확인
java -version

# Maven 확인 (없으면 설치)
mvn -version
sudo apt install maven -y
```

### API 키 설정

`src/main/resources/application.properties`:

```properties
gemini.api.key=여기에_Gemini_API_키_입력
gemini.api.url=https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent
```

### 빌드 & 배포

```bash
cd /home/bon/lunch-roulette
mvn clean package -DskipTests && \
sudo cp target/lunch-roulette.war /var/lib/tomcat10/webapps/ && \
sudo systemctl restart tomcat10
```

### 로그 확인

```bash
sudo tail -f /var/lib/tomcat10/logs/catalina.out
```

---

## 🔑 API 설정

Google Gemini API 키 발급:
1. [Google AI Studio](https://aistudio.google.com/apikey) 접속
2. **Create API Key** 클릭
3. 발급된 키를 `application.properties`에 입력

> ⚠️ API 키는 절대 GitHub에 커밋하지 마세요! `.gitignore`에 `application.properties`를 추가하세요.

```
# .gitignore
src/main/resources/application.properties
```

---

## 📝 느낀 점

- Tomcat 10.x는 Jakarta EE 네임스페이스를 사용하므로 `javax.*` → `jakarta.*` 전환 필요
- Maven 컴파일 시 `-parameters` 플래그는 Spring MVC `@RequestParam` 사용 시 필수
- Gemini API 응답 다양성을 높이려면 temperature 조절보다 **프롬프트에 시간 정보를 포함**하는 것이 더 효과적
- WAR 배포 환경에서 JSP 수정은 webapps 폴더 직접 수정으로 빠르게 반영 가능하나, 소스와 동기화를 위해 항상 재빌드 권장
