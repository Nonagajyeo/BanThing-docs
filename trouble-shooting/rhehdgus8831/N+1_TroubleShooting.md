# 🛠 Trouble Shooting: N+1 쿼리 문제 해결 기록

## 문서 정보
* **문서명**: N+1 쿼리 문제 해결 기록
* **작성자**: [고동현](https://github.com/rhehdgus8831)
* **작성일**: 2025-09-22

---

## 📅 발생 일자
- 2025-09-15


---

## 🧩 문제 상황
- 메인 페이지에서 전체 모임 목록을 조회하는 API(`GET /api/meetings/search`) 테스트 중, 모임(Meeting)의 개수(N)만큼 추가적인 쿼리가 발생하는 **N+1 문제**를 발견했습니다.
- 모임이 8개일 경우, 총 9개의 쿼리(모임 목록 1번 + 각 모임의 마트 정보 8번)가 발생하여 DB 부하 및 API 응답 지연 문제가 확인되었습니다.

---

## 🔍 원인 분석
- **지연 로딩(Lazy Loading)**: `Meeting`과 `Mart` 엔티티는 다대일(@ManyToOne) 관계로, 기본 지연 로딩 전략으로 설정되어 있습니다.
- **DTO 변환 시점**: `MeetingSimpleResponse` DTO로 변환하는 과정에서 각 모임의 `getMart().getMartName()`을 호출할 때, 지연 로딩으로 인해 `Mart` 엔티티를 조회하는 별도의 SELECT 쿼리가 매번 실행되었습니다.

---

## 🛠 해결 방법
`MeetingsRepository`에 JPQL과 **`JOIN FETCH`**를 사용하여, 모임 목록을 조회할 때 연관된 `Mart` 엔티티 정보까지 **한 번의 쿼리로 함께** 불러오도록 수정했습니다.

```java
// MeetingsRepository.java
@Query("SELECT m FROM Meeting m JOIN FETCH m.mart ORDER BY m.createdAt DESC")
List<Meeting> findAllWithMartByOrderByCreatedAtDesc();
````

-----

## ✅ 결과

해결 후, 모임 목록 조회 시 단 하나의 쿼리만 실행되어 성능이 개선되었습니다.

-----

## 📚 교훈 / 예방책

JPA의 지연 로딩과 N+1 문제에 대해 깊이 이해하게 되었습니다. 기능이 정상 동작하더라도 쿼리 로그를 통해 숨어있는 성능 병목을 확인하는 습관이 중요합니다. \*\*즉시 로딩(Eager Loading)\*\*보다 필요한 시점에 명시적으로 함께 조회하는 \*\*`JOIN FETCH`\*\*가 성능과 유연성 측면에서 더 나은 해결책임을 깨달았습니다.





