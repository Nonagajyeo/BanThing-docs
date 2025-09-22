# 🛠️ 트러블슈팅: 챗봇 모임 추천 카드 렌더링 실패

## 문서 정보
* **문서명**: 챗봇 모임 추천 카드 렌더링 실패 해결
* **작성자**: 김경민
* **작성일**: 2025-09-23

---

## 📅 발생 일자
- 2025-09-20~23

---

## 📌 상황

AI 챗봇에서 사용자가 "김치", "피자" 등의 상품명으로 검색할 때, 텍스트 응답은 "총 1개의 추천 모임이 있습니다"라고 나오지만 실제로는 모임 카드가 렌더링되지 않는 문제가 발생했습니다.

**기대하는 동작:**
- 사용자가 상품명 검색 시 관련 모임이 카드 형태로 렌더링
- 로그인/비로그인 사용자 모두 동일하게 작동

**실제 발생한 동작:**
- 텍스트로는 "모임을 찾았습니다"라고 응답
- `suggestedMeetings` 배열이 비어있음 (`[]`)
- 카드 컴포넌트가 렌더링되지 않음

---

## 📌 문제

### 1차 문제: 쉼표 기준 키워드 검색 실패
- "김치, 피자" 모임에서 "김치" 검색 시 조회 실패
- 쉼표 앞의 키워드는 매칭되지 않고 뒤의 키워드만 매칭됨

### 2차 문제: 의도 분류(`intentType`) 오분류
- 상품명 검색 시 `intentType`이 `GENERAL`로 분류됨
- `MEETING_SEARCH`로 분류되어야 모임 추천 로직이 실행됨

### 3차 문제: 로그인 사용자의 모임 추천 배열 생성 실패
- `intentType`은 `MEETING_SEARCH`로 올바르게 설정됨
- 하지만 `suggestedMeetings` 배열이 여전히 비어있음
- `generateMeetingSuggestions` 메서드에서 DB 저장 과정 중 오류 발생 추정

---

## 📌 해결

### 1단계: 키워드 추출 로직 개선
구두점 제거 및 키워드 분리 로직을 개선하여 쉼표 앞뒤 키워드 모두 인식되도록 수정

### 2단계: 의도 분류 개선
`determineIntentType` 메서드에 상품명 키워드를 추가하여 상품 검색 시 `MEETING_SEARCH`로 분류되도록 수정

### 3단계: 로그인 사용자 모임 추천 로직 단순화
복잡한 DB 저장 과정을 거치는 `generateMeetingSuggestions` 메서드 대신, 게스트 사용자와 동일한 단순한 DTO 생성 방식으로 변경

### ✅ 수정된 코드

**ChatbotServiceImpl.java - determineIntentType 메서드:**
```java
private ChatbotConversation.IntentType determineIntentType(String userMessage) {
    String lowerMessage = userMessage.toLowerCase();

    // 모임 검색 관련 키워드 (상품명도 포함)
    String[] searchKeywords = {
            "찾", "검색", "추천", "모임", "소분", "참여", "신청", "있나", "어디",
            "김치", "피자", "세제", "견과류", "아몬드", "호두", "다우니", "베이커리",
            "머핀", "베이글", "냉동식품", "만두", "과일", "육류", "삼겹살", "닭가슴살",
            "간식", "과자", "조미료", "올리브오일", "소스"
    };

    if (Arrays.stream(searchKeywords).anyMatch(lowerMessage::contains)) {
        log.info("모임 검색 키워드 감지: {}", userMessage);
        return ChatbotConversation.IntentType.MEETING_SEARCH;
    }
    
    // ... 기존 코드
}
```

**ChatbotServiceImpl.java - processAuthenticatedMessage 메서드:**
```java
// 6. 모임 추천 생성 - 게스트와 동일한 방식으로 변경
List<ChatbotMessageResponse.MeetingSuggestionResponse> suggestedMeetings = new ArrayList<>();
if (intentType == ChatbotConversation.IntentType.MEETING_SEARCH && !activeMeetings.isEmpty()) {
    List<Meeting> relevantMeetings = findRelevantMeetings(keywords, activeMeetings);
    
    // 관련 모임이 없으면 최신 모임 3개 추천
    if (relevantMeetings.isEmpty()) {
        relevantMeetings = activeMeetings.stream()
                .limit(3)
                .collect(Collectors.toList());
    }

    // DB 저장 없이 DTO만 생성
    for (Meeting meeting : relevantMeetings) {
        String suggestionReason = generateSuggestionReason(userMessage, meeting, keywords);

        suggestedMeetings.add(ChatbotMessageResponse.MeetingSuggestionResponse.builder()
                .meetingId(meeting.getMeetingId())
                .title(meeting.getTitle())
                .martName(meeting.getMart().getMartName())
                .meetingDate(meeting.getMeetingDate())
                .suggestionReason(suggestionReason)
                .currentParticipants(meeting.getCurrentParticipants())
                .maxParticipants(meeting.getMaxParticipants())
                .status(meeting.getStatus().toString())
                .martAddress(meeting.getMart().getAddress())
                .build());
    }
}
```

---

## 📌 결과

✅ **"김치" 검색 시 정상적으로 관련 모임 카드 렌더링**  
✅ **로그인/비로그인 사용자 모두 동일하게 작동**  
✅ **`suggestedMeetings` 배열에 데이터 정상 포함**  
✅ **`intentType`이 `MEETING_SEARCH`로 올바르게 분류**

**최종 API 응답:**
```json
{
    "success": true,
    "data": {
        "response": "안녕하세요! 😊 반띵 AI 도우미입니다...",
        "suggestedMeetings": [
            {
                "meetingId": 123,
                "title": "김치 소분 모임",
                "martName": "코스트코 양재점",
                // ... 기타 데이터
            }
        ],
        "intentType": "MEETING_SEARCH"
    }
}
```

---

## 📌 배운 점

### 1. 복잡한 로직보다 단순한 구조가 안정적
- 로그인 사용자의 경우 DB 저장을 포함한 복잡한 `generateMeetingSuggestions` 메서드 사용
- 게스트 사용자는 단순한 DTO 생성만 수행
- **결과**: 게스트는 정상 작동, 로그인 사용자는 실패
- **교훈**: 일관된 로직 사용의 중요성

### 2. 의도 분류의 중요성
- 챗봇에서 `intentType` 분류가 잘못되면 전체 로직이 실행되지 않음
- 상품명 키워드를 포함한 포괄적인 키워드 리스트 필요
- **교훈**: 엣지 케이스를 고려한 키워드 설계 필요

### 3. 디버깅 시 API 응답 구조 우선 확인
- 프론트엔드 렌더링 문제라고 생각했지만 실제로는 백엔드 데이터 생성 문제
- Network 탭에서 실제 API 응답을 먼저 확인하는 것이 효율적
- **교훈**: 문제 발생 지점을 정확히 파악한 후 디버깅 진행

### 4. 로그인/비로그인 사용자 간 로직 일관성
- 동일한 기능이지만 다른 메서드를 사용하여 결과가 달라짐
- 기능이 동일하다면 동일한 로직을 사용하는 것이 유지보수에 유리
- **교훈**: 코드 중복 제거와 로직 통일의 중요성