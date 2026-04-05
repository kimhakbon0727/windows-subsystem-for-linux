# 🍺 AI 술안주 룰렛 (Snack Roulette)

> Google Gemini AI가 주종에 맞는 술안주를 추천하고, Canvas 룰렛으로 오늘의 안주를 결정해주는 웹 애플리케이션

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

---

## 🎯 프로젝트 소개

점심 룰렛에 이어 오늘 마실 술에 어울리는 안주를 AI가 골라주는 프로젝트입니다.
주종(맥주/소주/위스키/와인/막걸리)을 선택하면 Google Gemini API가 어울리는 안주 8가지를 추천하고, Canvas 룰렛을 돌려 오늘의 안주를 결정합니다.

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
snack-roulette/
├── pom.xml
└── src/main/
    ├── java/com/snack/
    │   ├── controller/
    │   │   └── SnackController.java     # Spring MVC 컨트롤러
    │   ├── service/
    │   │   └── GeminiService.java       # Gemini API 연동 서비스
    │   └── model/
    │       └── SnackMenu.java           # 안주 모델
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

- 🍺 **주종 탭 선택** - 맥주 / 소주 / 위스키 / 와인 / 막걸리 중 선택
- 🤖 **AI 안주 추천** - 선택한 주종에 맞는 안주 8가지를 Gemini가 추천
- 💡 **추천 이유 표시** - 당첨 시 해당 주종과 어울리는 이유 함께 표시
- 🎰 **Canvas 룰렛** - easeOut 애니메이션으로 부드럽게 스핀
- 🎨 **어두운 테마** - 술집 분위기의 다크 UI
- 🔄 **폴백 처리** - API 실패 시 내장된 기본 안주 8개로 자동 대체

---

## 🔧 개발 과정 및 트러블슈팅

### 1️⃣ 프로젝트 설계

기존 lunch-roulette 구조를 그대로 활용하되 주종 선택 기능을 추가했습니다.

**lunch-roulette와 달라진 점**

| 항목 | lunch-roulette | snack-roulette |
|------|---------------|----------------|
| 패키지 | `com.lunch` | `com.snack` |
| 모델 | `LunchMenu` (name, description, price) | `SnackMenu` (name, description, reason) |
| API 엔드포인트 | `/lunch/suggest` | `/snack/suggest?drinkType=맥주` |
| 추가 기능 | 없음 | 주종 탭 선택 UI |
| 테마 | 밝은 보라 그라데이션 | 어두운 네이비 테마 |

---

### 2️⃣ 주종별 안주 추천 프롬프트 설계

주종에 따라 다른 안주를 추천받기 위해 프롬프트에 주종을 동적으로 삽입했습니다.

```java
private String buildPrompt(String drinkType, int count) {
    String now = java.time.LocalDateTime.now().toString();
    return String.format("""
            현재 시각은 %s 입니다. 이 시각을 시드로 완전히 새로운 안주를 골라주세요.
            %s에 어울리는 안주 %d가지를 추천해 주세요.
            절대로 이전에 추천한 안주와 겹치면 안됩니다.
            한식, 튀김류, 구이류, 전류, 해산물, 과자류 등 다양하게 섞어서 추천해 주세요.
            """, now, drinkType, count, drinkType);
}
```

- 현재 시각을 시드로 포함해 매번 다른 응답 유도
- `temperature: 2.0` 으로 최대 다양성 설정

---

### 3️⃣ 주종 탭 UI 구현

JSP에서 주종 탭을 클릭하면 `selectedDrink` 변수를 바꾸고 룰렛을 초기화합니다.

```javascript
function selectDrink(el) {
    document.querySelectorAll('.drink-tab').forEach(t => t.classList.remove('active'));
    el.classList.add('active');
    selectedDrink = el.dataset.drink;

    // 주종 바뀌면 룰렛 초기화
    snacks = [];
    drawPlaceholder();
    document.getElementById('btnSpin').disabled = true;
    document.getElementById('result').classList.remove('show');
}
```

---

### 4️⃣ WAR 배포

```bash
cd /home/bon/snack-roulette
mvn clean package -DskipTests
sudo cp target/snack-roulette.war /var/lib/tomcat10/webapps/
sudo systemctl restart tomcat10
```

접속 URL: `http://서버IP:8888/snack-roulette/snack`

---

### 5️⃣ 🐛 트러블슈팅 #1 - 주종 바꿔도 같은 메뉴 출력

**증상**

주종 탭을 맥주 → 소주 → 위스키로 바꿔도 항상 동일한 안주 목록이 나옴

**원인 파악 과정**

curl로 API 직접 호출:
```bash
curl -v "http://localhost:8080/snack-roulette/snack/suggest?drinkType=%EB%A7%A5%EC%A3%BC&count=8"
```

응답에서 폴백 메뉴(치킨, 감자튀김 등 고정 목록)가 나오는 것을 확인 → Gemini API 호출 실패 중

로그 확인:
```bash
sudo tail -100 /var/lib/tomcat10/logs/catalina.out | grep -E "ERROR|Gemini|API"
```

```
INFO  GeminiService -- Gemini API 응답 코드: 429
ERROR GeminiService -- Gemini API 오류: { "description": "Learn more about Gemini API quotas" }
```

**원인**

`429 Too Many Requests` - API 요청 한도 초과
무료 등급의 분당 요청 수 제한에 걸림
또한 API 키가 공개 채팅에 노출되어 외부에서 사용됐을 가능성 있음

**해결**

1. 기존 노출된 API 키 즉시 삭제
2. [Google AI Studio](https://aistudio.google.com/apikey)에서 새 키 발급
3. `application.properties` 교체 후 재배포

```bash
sed -i 's/기존키/새키/' \
  /home/bon/snack-roulette/src/main/resources/application.properties
cd /home/bon/snack-roulette && mvn clean package -DskipTests && \
sudo cp target/snack-roulette.war /var/lib/tomcat10/webapps/ && \
sudo systemctl restart tomcat10
```

> ⚠️ **교훈**: API 키는 절대 공개 채팅이나 GitHub에 노출하지 말 것! `.gitignore`에 `application.properties` 추가 필수

---

## 🚀 배포 방법

### API 키 설정

```bash
sed -i 's/YOUR_GEMINI_API_KEY_HERE/발급받은_키/' \
  /home/bon/snack-roulette/src/main/resources/application.properties
```

### 빌드 & 배포

```bash
cd /home/bon/snack-roulette
mvn clean package -DskipTests && \
sudo cp target/snack-roulette.war /var/lib/tomcat10/webapps/ && \
sudo systemctl restart tomcat10
```

### 로그 확인

```bash
sudo tail -f /var/lib/tomcat10/logs/catalina.out
```

---

## 🔑 API 키 보안 주의사항

```gitignore
# .gitignore 에 반드시 추가
src/main/resources/application.properties
```

> API 키가 외부에 노출되면 타인이 사용해 429 한도 초과 오류가 발생할 수 있습니다.
> 노출된 키는 즉시 [Google AI Studio](https://aistudio.google.com/apikey)에서 삭제하세요.

---

## 📝 느낀 점

- 기존 프로젝트 구조를 재활용하면 개발 속도가 크게 향상됨
- 한글 파라미터는 URL 인코딩(`encodeURIComponent`) 필수
- Gemini 무료 등급은 분당 요청 수 제한이 있으므로 연속 호출 주의
- API 키 보안은 개발 초기부터 철저히 관리해야 함
