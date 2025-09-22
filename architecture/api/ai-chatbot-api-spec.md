# AI 챗봇 API 명세서

-----

## 문서 정보

- **문서명**: AI 챗봇 API 명세서
- **버전**: v2.0.0
- **작성일**: 2025.09.11
- **작성자**: 김경민
- **최종 수정일**: 2025.09.22

-----

## 1. 개요

* **API 버전**: v2.0
* **기본 URL**: `localhost:9000/api`
* **인증**: JWT 기반 Bearer Token (`Authorization: Bearer {token}`) 또는 HTTP-Only 쿠키
* **설명**: Google Gemini API를 활용한 AI 챗봇 기능에 대한 명세서입니다.
* **특징**:
    - 로그인 없이도 기본 챗봇 기능 이용 가능
    - 로그인 시 개인화된 응답 및 대화 기록 저장
    - 실시간 모임 추천 기능 제공

-----

## 2. 챗봇 메시지 전송

### 2.1. 기본 메시지 전송 (로그인 선택적)

* **API 엔드포인트**: `POST /chatbot/message`
* **설명**: 사용자 질문에 대한 AI 챗봇의 응답을 요청합니다. 로그인 여부에 따라 다른 수준의 서비스를 제공합니다.
* **요청 (Request)**
    * **헤더**:
        * `Content-Type: application/json`
        * `Authorization: Bearer {JWT}` (선택적 - 로그인한 사용자만)
    * **바디 (Body)**:
      ```json
      {
        "message": "양재점 코스트코에서 견과류 소분 모임 있나요?"
      }
      ```
    * **필드 설명**:
        * `message` (`string`, 필수): 사용자의 질문 메시지 (최대 1000자)

* **응답 (Response)**
    * **성공 (200 OK) - 로그인 사용자**:
      ```json
      {
        "status": "success",
        "data": {
          "response": "안녕하세요 홍길동님! 현재 양재점에서 견과류 소분 모임이 2개 진행 중입니다.",
          "suggestedMeetings": [
            {
              "meetingId": 123,
              "title": "코스트코 견과류 대용량 소분 모임",
              "martName": "코스트코 양재점",
              "meetingDate": "2025-09-25T14:00:00",
              "suggestionReason": "'견과류' 관련 요청에 적합한 모임입니다.",
              "currentParticipants": 3,
              "maxParticipants": 5,
              "status": "RECRUITING",
              "martAddress": "서울특별시 서초구 양재대로 159"
            }
          ],
          "intentType": "MEETING_SEARCH",
          "conversationId": 456
        },
        "message": "챗봇 응답이 성공적으로 생성되었습니다."
      }
      ```
    * **성공 (200 OK) - 게스트 사용자**:
      ```json
      {
        "status": "success",
        "data": {
          "response": "안녕하세요! 현재 양재점에서 견과류 소분 모임이 진행 중입니다. 더 정확한 정보를 원하시면 아래 카카오로 시작하기 버튼을 이용해주세요!",
          "suggestedMeetings": [
            {
              "meetingId": 123,
              "title": "코스트코 견과류 대용량 소분 모임",
              "martName": "코스트코 양재점",
              "meetingDate": "2025-09-25T14:00:00",
              "suggestionReason": "'견과류' 관련 요청에 적합한 모임입니다.",
              "currentParticipants": 3,
              "maxParticipants": 5,
              "status": "RECRUITING",
              "martAddress": "서울특별시 서초구 양재대로 159"
            }
          ],
          "intentType": "MEETING_SEARCH",
          "conversationId": null
        },
        "message": "챗봇 응답이 성공적으로 생성되었습니다."
      }
      ```
    * **응답 필드 설명**:
        * `response` (`string`): AI 챗봇의 답변
        * `suggestedMeetings` (`array`): 추천 모임 목록 (최대 3개)
            * `meetingId` (`number`): 모임 고유 식별자
            * `title` (`string`): 모임 제목
            * `martName` (`string`): 마트명
            * `meetingDate` (`string`): 모임 일시 (ISO 8601)
            * `suggestionReason` (`string`): 추천 이유
            * `currentParticipants` (`number`): 현재 참여 인원수
            * `maxParticipants` (`number`): 최대 참여 가능 인원수
            * `status` (`string`): 모임 상태 (`RECRUITING`, `FULL`, `ONGOING`, `COMPLETED`, `CANCELLED`)
            * `martAddress` (`string`): 마트 주소
        * `intentType` (`string`): 질문 의도 분류 (`MEETING_SEARCH`, `SERVICE_GUIDE`, `GENERAL`)
        * `conversationId` (`number|null`): 대화 기록 ID (로그인 사용자만)

### 2.2. 게스트 전용 엔드포인트

* **API 엔드포인트**: `POST /chatbot/guest`
* **설명**: 비로그인 사용자 전용 챗봇 엔드포인트입니다. 디버깅이 강화된 버전입니다.
* **요청 (Request)**:
    * **헤더**: `Content-Type: application/json`
    * **바디**: 위와 동일
* **응답**: 위의 게스트 사용자 응답과 동일

* **오류 (Error)**:
    * **400 Bad Request**:
      ```json
      {
        "status": "error",
        "message": "잘못된 요청입니다. 메시지를 확인해주세요.",
        "code": "INVALID_INPUT"
      }
      ```
    * **500 Internal Server Error**:
      ```json
      {
        "status": "error", 
        "message": "AI 챗봇 API 호출 중 오류가 발생했습니다.",
        "code": "AI_API_ERROR"
      }
      ```

-----

## 3. 챗봇 대화 기록 조회

* **API 엔드포인트**: `GET /chatbot/history`
* **설명**: 로그인한 사용자의 챗봇 대화 기록을 조회합니다. 최근 10개 대화를 최신순으로 반환합니다.
* **요청 (Request)**
    * **헤더**:
        * `Authorization: Bearer {JWT}` (필수)
* **응답 (Response)**
    * **성공 (200 OK)**:
      ```json
      {
        "status": "success",
        "data": [
          {
            "conversationId": 456,
            "userMessage": "양재점 코스트코에서 견과류 소분 모임 있나요?",
            "botResponse": "안녕하세요 홍길동님! 현재 양재점에서 견과류 소분 모임이 2개 진행 중입니다.",
            "intentType": "MEETING_SEARCH",
            "createdAt": "2025-09-22T12:30:00Z",
            "suggestedMeetings": [
              {
                "meetingId": 123,
                "title": "코스트코 견과류 대용량 소분 모임",
                "suggestionReason": "'견과류' 관련 요청에 적합한 모임입니다.",
                "martName": "코스트코 양재점",
                "meetingDate": "2025-09-25T14:00:00",
                "status": "RECRUITING"
              }
            ]
          },
          {
            "conversationId": 455,
            "userMessage": "안녕하세요",
            "botResponse": "안녕하세요! 반띵 AI 도우미입니다. 무엇을 도와드릴까요?",
            "intentType": "GENERAL",
            "createdAt": "2025-09-22T12:00:00Z",
            "suggestedMeetings": []
          }
        ],
        "message": "대화 기록을 성공적으로 조회했습니다."
      }
      ```
    * **응답 필드 설명**:
        * `conversationId` (`number`): 대화 기록 ID
        * `userMessage` (`string`): 사용자 메시지
        * `botResponse` (`string`): 챗봇 답변
        * `intentType` (`string`): 의도 유형
        * `createdAt` (`string`): 대화 기록 시간 (ISO 8601)
        * `suggestedMeetings` (`array`): 해당 대화에서 추천된 모임 목록

* **오류 (Error)**:
    * **401 Unauthorized**:
      ```json
      {
        "status": "error",
        "message": "인증 토큰이 유효하지 않습니다.",
        "code": "UNAUTHORIZED"
      }
      ```
    * **500 Internal Server Error**:
      ```json
      {
        "status": "error",
        "message": "데이터베이스 오류가 발생했습니다.",
        "code": "DATABASE_ERROR"
      }
      ```

-----

## 4. 서비스 상태 확인

* **API 엔드포인트**: `GET /chatbot/health`
* **설명**: 챗봇 서비스의 상태를 확인합니다.
* **요청**: 헤더 없음
* **응답**:
    * **정상 (200 OK)**:
      ```json
      {
        "status": "success",
        "data": "HEALTHY",
        "message": "챗봇 서비스가 정상 작동 중입니다."
      }
      ```
    * **오류 (200 OK)**:
      ```json
      {
        "status": "error",
        "message": "챗봇 서비스에 일시적인 문제가 있습니다."
      }
      ```

-----

## 5. 서비스 소개

* **API 엔드포인트**: `GET /chatbot/intro`
* **설명**: 반띵 서비스 소개 메시지를 반환합니다.
* **요청**: 헤더 없음
* **응답 (200 OK)**:
  ```json
  {
    "status": "success",
    "data": "안녕하세요! 반띵 AI 도우미입니다. 😊\n\n🛒 반띵은 대용량 상품을 여러 명이 함께 구매하고 소분하는 서비스예요.\n\n📍 서울 지역 8개 마트에서 다양한 소분 모임이 진행되고 있어요.\n\n💡 1-2인 가구도 대용량 상품을 합리적으로 구매할 수 있도록 도와드려요!\n\n궁금한 점이 있으시면 언제든 말씀해주세요!",
    "message": "서비스 소개입니다."
  }
  ```

-----

## 6. 데이터 모델

### 6.1. ChatbotConversation 엔티티

```java
public enum IntentType {
    MEETING_SEARCH,    // 모임 검색
    SERVICE_GUIDE,     // 서비스 가이드  
    GENERAL           // 일반 대화
}
```

### 6.2. 엔티티 구조

**Meeting 엔티티:**
```java
public enum MeetingStatus {
    RECRUITING,  // 모집중
    FULL,        // 마감  
    ONGOING,     // 진행중
    COMPLETED,   // 완료
    CANCELLED    // 취소
}
```

**User 엔티티:**
```java
public enum TrustGrade {
    WARNING,  // 0-99점: 노쇼 이력이 있는 사용자
    BASIC,    // 300점 기본: 일반 사용자  
    GOOD      // 500점 이상: 신뢰도가 높은 우수 사용자
}
```

**Mart 엔티티:**
```java
public enum MartBrand {
    COSTCO,      // 코스트코
    TRADERS,     // 이마트 트레이더스
    LOTTE_MART   // 롯데마트 맥스
}
```

### 6.3. 지원 마트 정보

**코스트코 (4곳):**
- 코스트코 양평점: 서울특별시 영등포구 선유로 156
- 코스트코 양재점: 서울특별시 서초구 양재대로 159
- 코스트코 상봉점: 서울특별시 중랑구 망우로 336
- 코스트코 고척점: 서울특별시 구로구 경인로43길 49

**이마트 트레이더스 (2곳):**
- 이마트 트레이더스 월계점: 서울특별시 노원구 마들로3길 17
- 이마트 트레이더스 마곡점: 서울특별시 강서구 공항대로 165

**롯데마트 맥스 (2곳):**
- 롯데마트 맥스 금천점: 서울특별시 금천구 두산로 71
- 롯데마트 맥스 영등포점: 서울특별시 영등포구 영중로 125

### 6.4. 모임 유형 예시

- 견과류 소분 (아몬드, 호두 등)
- 세제/생활용품 소분 (다우니, 세정제 등)
- 베이커리 소분 (머핀, 베이글 등)
- 냉동식품 소분 (냉동만두, 냉동과일 등)
- 육류 소분 (삼겹살, 닭가슴살 등)
- 간식 소분 (과자, 견과류 등)
- 조미료 소분 (올리브오일, 소스류 등)

-----

## 7. 에러 코드 정리

| HTTP Status | Error Code | 설명 |
|-------------|------------|------|
| 400 | INVALID_INPUT | 잘못된 요청 데이터 |
| 401 | UNAUTHORIZED | 인증 실패 |
| 500 | AI_API_ERROR | Google Gemini API 호출 실패 |
| 500 | DATABASE_ERROR | 데이터베이스 오류 |

-----

## 8. 설정 정보

### 8.1. Google AI 설정

```yaml
google:
  ai:
    api-key: ${GOOGLE_AI_API_KEY}
    model: ${GOOGLE_AI_MODEL:gemini-1.5-flash}
    temperature: ${GOOGLE_AI_TEMPERATURE:0.7}
    max-tokens: ${GOOGLE_AI_MAX_TOKENS:1000}
```

### 8.2. 환경변수

```bash
# Google Gemini API 설정
GOOGLE_AI_API_KEY=your_google_gemini_api_key_here
GOOGLE_AI_MODEL=gemini-1.5-flash
GOOGLE_AI_TEMPERATURE=0.7
GOOGLE_AI_MAX_TOKENS=1000
```

>자세한 방법은 다음 문서를 참조하세요:
- **[Google Gemini API 설정 가이드](../../guides/API_SETUP_GOOGLE_GEMINI.md)**

-----

## 9. 기술적 특징

### 9.1. AI 장애 대응
- Google Gemini API 장애 시 자동으로 대체 로직으로 전환
- 키워드 기반 모임 매칭으로 서비스 연속성 보장

### 9.2. 보안
- JWT 토큰 및 HTTP-Only 쿠키 지원
- 메시지 길이 제한 (최대 1000자)
- 입력 데이터 검증

### 9.3. 성능
- 대화 기록 페이징 처리 (최근 10개)
- 모임 추천 최대 3개로 제한
- 실시간 모임 정보 캐싱

-----

## 변경 이력

| 버전   | 날짜         | 변경 내용                  | 작성자 |
|--------|--------------|--------------------------|-----|
| v1.0.0 | 2025.09.11   | 초기 문서 작성           | 송민재 |
| v1.0.1 | 2025.09.11   | JPA 엔터티 기반 응답 필드 상세화 | 송민재 |
| v1.1.0 | 2025.09.12   | 로그인 선택적 기능 추가 | 김경민 |
| v2.0.0 | 2025.09.22   | 실제 구현 코드 기반 전면 개편 | 김경민 |

-----