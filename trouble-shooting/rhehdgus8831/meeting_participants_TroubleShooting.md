
# 🛠 Trouble Shooting: 참여자 승인 및 거절 기능 복합 오류 해결 기록

## 문서 정보
* **문서명**: 참여자 승인 및 거절 기능 복합 오류 해결 기록
* **작성자**: [고동현](https://github.com/rhehdgus8831)
* **작성일**: 2025-09-22


---

## 📅 발생 일자
- 2025-09-19

---

## 🧩 문제 상황
1.  **UI 문제**: 모임 상세 페이지에서 호스트 본인이 확정자 명단에 보이지 않고, 대기자의 신뢰도 점수가 렌더링되지 않았습니다.
2.  **API 통신 오류**: '승인' 또는 '거절' 버튼 클릭 시 `404 Not Found` 또는 `405 Method Not Allowed` 에러가 발생했습니다.
3.  **DB 정합성 문제**: API 오류 해결 후, 승인/거절 요청은 성공하는 것처럼 보였으나 페이지 새로고침 시 원래의 '대기중' 상태로 되돌아갔습니다.

---

## 🔍 원인 분석
- **ID 혼동**: 백엔드는 **`participantId`**를 요구했으나, 프론트에서는 **`userId`**를 전송하고 있었습니다.
- **DTO 필드 누락**: `MeetingParticipantResponse` DTO에 `participantId` 필드가 누락되어 프론트에서 올바른 ID를 사용할 수 없었습니다.
- **잘못된 `@Transactional` Import (핵심 원인)**: `jakarta.transaction.Transactional`을 사용하여 DB 변경 사항이 정상적으로 커밋되지 않고 조용히 롤백되고 있었습니다. Spring Boot 환경에서는 **`org.springframework.transaction.annotation.Transactional`**을 사용해야 합니다.

---

## 🛠 해결 방법
- **백엔드**: `MeetingParticipantResponse` DTO에 `participantId` 필드를 추가하고, `ManageMeetingService`의 `import` 문을 `org.springframework.transaction.annotation.Transactional`로 변경했습니다.
- **프론트엔드**: API 호출 시 `userId` 대신 `participantId`를 사용하도록 수정하고, 사용자 경험 향상을 위해 '낙관적 업데이트' 로직을 구현했습니다.

---

## ✅ 결과
모든 수정 후, 참여자 승인/거절 기능이 UI, API, DB 전반에 걸쳐 정상적으로 동작하고 데이터 정합성이 유지되는 것을 확인했습니다.

---

## 📚 교훈 / 예방책
**프론트엔드와 백엔드 간의 DTO 명세(데이터 계약)를 명확히 정의하고 공유하는 문화**가 초기 개발 단계의 오류를 크게 줄일 수 있다는 교훈을 얻었습니다. 또한, IDE의 자동완성 기능에 의존하다 보면 잘못된 라이브러리를 import하기 쉬우므로, **Spring 환경에서는 `org.springframework` 패키지를 우선적으로 확인**하는 습관을 들여야겠습니다.