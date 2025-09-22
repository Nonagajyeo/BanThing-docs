# 🛠 Trouble Shooting: JWT 인증 시 providerId, userId 타입 불일치 문제 해결 기록

## 문서 정보
* **문서명**: JWT 인증 시 providerId, userId 타입 불일치 문제 해결 기록
* **작성자**: [고동현](https://github.com/rhehdgus8831)
* **작성일**: 2025-09-22

---

## 📅 발생 일자
- 2025-09-16

---

## 🧩 문제 상황
유효한 `ACCESS_TOKEN`을 헤더에 담아 API(`POST /api/meetings`)를 호출했으나, `401 Unauthorized` 또는 커스텀 에러(`AUTHENTICATION_NOT_FOUND`)가 반환되었습니다. 서버 로그에는 토큰 자체는 유효하지만 사용자를 식별하지 못하는 것으로 나타났습니다.

---

## 🔍 원인 분석
- `JwtAuthenticationFilter`는 토큰의 `subject`에 저장된 **`providerId`(String 타입)**를 `SecurityContext`의 `Principal`로 설정합니다.
- `MeetingController`에서는 `@AuthenticationPrincipal Long currentUserId` 어노테이션을 통해 `Principal` 값을 주입받으려고 했습니다.
- **핵심 원인**: Spring Security는 `String` 타입의 `providerId`를 `Long` 타입의 `currentUserId`로 자동 형변환할 수 없어 `null`이 주입되었고, 서비스 로직에서 사용자를 찾지 못해 예외가 발생했습니다.

---

## 🛠 해결 방법
인증 과정 전반에 걸쳐 사용자 식별자를 **`String` 타입인 `providerId`로 통일**했습니다.

```java
// MeetingController.java
@PostMapping
public ResponseEntity<ApiResponse<MeetingCreateResponse>> createMeeting(
        @Valid @RequestBody MeetingCreateRequest request,
        @AuthenticationPrincipal String providerId) { // ✅ String 타입으로 올바르게 주입
    Meeting newMeeting = createMeetingService.createMeeting(request, providerId);
    // ...
}
````

-----

## ✅ 결과

`providerId`를 `String` 타입으로 일관되게 사용한 후, 인증이 필요한 모든 API가 정상적으로 동작하는 것을 Postman을 통해 확인했습니다.

-----

## 📚 교훈 / 예방책

단일 소셜 로그인 환경에서는 `providerId`를 직접 `Principal`로 사용하는 것이 직관적입니다. 하지만 추후 다수의 SSO 공급자를 도입할 경우를 대비해, 필터 단에서 `providerId`로 `User`를 조회한 뒤 시스템 고유 식별자인 **`userId(Long)`를 `Principal`로 설정**하는 방식이 더 유연하고 확장성 있는 구조가 될 수 있음을 고려해야 합니다.

