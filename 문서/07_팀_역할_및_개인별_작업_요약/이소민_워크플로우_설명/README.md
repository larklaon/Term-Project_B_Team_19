# Make 시나리오 구조 분석

**시용모듈:** Integration RSS, HTTP, OpenAI (ChatGPT, Sora, Whisper), Notion

**실행 주기:** 매일 오전 9:00 자동 실행

---

## 전체 플로우

```jsx
[RSS] AI타임스 (최대 20개)
	↓
keyword filter: "로봇" 키워드 Title / Description / Summary 중 하나 포함 시 통과
	↓
[HTTP] 기사 원문 URL → HTTP GET → 본문 HTML 수집 (Error: Retry 2회)
	↓
[OpenAI] GPT-5.4 → 한국어 3줄 요약 생성 (Error: Retry 2회)
	↓
[Notion] DB 저장 (제목 + 발행일 + 요약 + 원문링크)
```

---

## 단계별 역할

### 1️⃣ RSS — 뉴스 수집

| 항목        | 내용                                           |
| ----------- | ---------------------------------------------- |
| 소스        | `https://www.aitimes.com/rss/allArticle.xml`   |
| 최대 수집량 | 20개 / 1회 실행                                |
| 출력 데이터 | Title, Description, Summary, URL, Date created |

AI타임스 RSS 피드에서 최신 기사를 최대 20개 가져온다.

---

### 2️⃣ keyword filter — 키워드 필터링

조건은 OR 방식으로, 아래 중 하나라도 해당되면 다음 단계로 통과한다.

| 필드        | 조건     | 값   |
| ----------- | -------- | ---- |
| Title       | Contains | 로봇 |
| Description | Contains | 로봇 |
| Summary     | Contains | 로봇 |

---

### 3️⃣ HTTP — 기사 본문 크롤링

| 항목      | 내용                                      |
| --------- | ----------------------------------------- |
| 방식      | GET                                       |
| URL       | `2. RSS fields: link` (RSS 기사 원문 URL) |
| 출력      | 기사 본문 데이터 (`4. Data`)              |
| 에러 처리 | Retry                                     |

필터를 통과한 기사의 URL로 직접 HTTP 요청을 보내 본문 HTML을 가져온다.

---

### 4️⃣ OpenAI (모듈 5) — AI 요약 생성

| 항목              | 내용                         |
| ----------------- | ---------------------------- |
| 모델              | `gpt-5.4 (system)`           |
| Prompt Type       | Text prompt                  |
| Max Output Tokens | 500                          |
| 입력              | `4. Data` (HTTP 크롤링 본문) |
| 출력              | 요약 텍스트 (`5. Result`)    |
| 에러 처리         | Retry                        |

**프롬프트:**

> 아래 뉴스 기사 내용을 핵심 내용 중심으로 3줄 이내로 한국어로 요약해줘. 각 줄은 "- "로 시작해줘.
> 기사 내용: `{4. Data}`
