
# 반띵 일반 API 명세서

- 본 문서는 '반띵(BanThing)' 서비스의 RESTful API를 사용하는 개발자를 위한 공식 기술 명세서입니다.

### 문서 정보

- **문서명**: 반띵 일반 API 명세서
- **버전**: v2.0.1
- **작성일**: 2025.09.11
- **작성자**: 송민재
- **최종 수정일**: 2025.09.23

-----

## 1\. 공통 가이드

### 1.1. 기본 정보

- **Base URL**: `http://localhost:9000/api`

### 1.2. 인증

- 인증이 필요한 모든 API는 요청 헤더에 JWT 기반의 Bearer Token을 포함해야 합니다.
- **Header**: `Authorization: Bearer {accessToken}`

### 1.3. 공통 응답 포맷

- 모든 API 응답은 아래의 `ApiResponse` DTO 형식으로 통일됩니다. (피드백 API 제외)
  ```json
  {
    "success": true,
    "message": "요청 처리 성공 메시지",
    "timestamp": "2025-09-22T12:00:00.000Z",
    "data": { ... } // API 별 실제 데이터, 실패 시 ErrorResponse DTO
  }
  ```

### 1.4. 공통 에러 응답

- API 요청 실패 시, `success`는 `false`가 되며 `data` 필드에는 `ErrorResponse` DTO가 포함됩니다.
  ```json
  {
      "success": false,
      "message": "에러 발생 메시지 (예: 요청한 리소스를 찾을 수 없습니다.)",
      "timestamp": "2025-09-22T12:00:00.000Z",
      "data": {
          "code": "ERROR_CODE",
          "message": "에러 상세 메시지 (예: 사용자를 찾을 수 없습니다.)"
      }
  }
  ```
  | HTTP 상태 | 에러 코드 (ErrorCode) | 설명 |
      |---|---|---|
  | 401 Unauthorized | `INVALID_TOKEN` | 유효하지 않은 토큰 또는 토큰 없음 |
  | 403 Forbidden | `FORBIDDEN` | 해당 리소스에 접근할 권한이 없음 |
  | 404 Not Found | `_NOT_FOUND` | 요청한 리소스를 찾을 수 없음 (예: `USER_NOT_FOUND`) |
  | 409 Conflict | `ALREADY_` | 리소스가 이미 존재하거나 중복된 요청 (예: `ALREADY_JOINED_MEETING`) |
  | 500 Internal Server Error | `INTERNAL_SERVER_ERROR`| 서버 내부 오류 |

-----

## 2\. 사용자 (User)

### 2.1. 내 프로필 조회

- **Endpoint**: `GET /users/me`
- **설명**: 현재 로그인한 사용자의 상세 프로필 정보를 조회합니다.
- **인증**: **필수**
- **요청**: 없음
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<UserInfoResponse>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "사용자의 정보가 성공적으로 조회되었습니다.",
        "timestamp": "2025-09-22T12:01:00.123Z",
        "data": {
          "userId": 1,
          "nickname": "반띵러123",
          "profileImageUrl": "https://k.kakaocdn.net/image/...",
          "selfIntroduction": "안녕하세요! 코스트코 소분 모임에 참여하고 싶어요.",
          "provider": "kakao",
          "trustScore": 300,
          "trustGrade": "BASIC",
          "noShowCount": 0,
          "agree": true
        }
      }
      ```
    - **상세 스펙 (`data`)**:
  
      | 필드 | 타입 | 설명 |
      |---|---|---|
      | `userId` | `Long` | 사용자 고유 ID |
      | `nickname` | `String` | 닉네임 |
      | `profileImageUrl` | `String` | 프로필 이미지 URL |
      | `selfIntroduction`| `String` | 자기소개 |
      | `provider`| `String` | 소셜 로그인 제공자 (e.g., "kakao") |
      | `trustScore`| `Integer`| 신뢰도 점수 |
      | `trustGrade`| `String` | 신뢰도 등급 (`WARNING`, `BASIC`, `GOOD`) |
      | `noShowCount` | `Integer`| 노쇼 횟수 |
      | `agree` | `Boolean`| 약관 동의 여부 |
- **오류**:
    - `401 Unauthorized`: `INVALID_TOKEN`
    - `404 Not Found`: `USER_NOT_FOUND`

### 2.2. 약관 동의 업데이트

- **Endpoint**: `PUT /users/me/agreement`
- **설명**: 사용자의 약관 동의 상태를 `true`로 업데이트합니다.
- **인증**: **필수**
- **요청**: 없음
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<UserResponse>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "사용자의 약관 동의 여부가 성공적으로 수정되었습니다.",
        "timestamp": "2025-09-22T12:02:00.456Z",
        "data": {
          "userId": 1,
          "nickname": "반띵러123"
        }
      }
      ```
    - **상세 스펙 (`data`)**:
  
      | 필드 | 타입 | 설명 |
      |---|---|---|
      | `userId` | `Long` | 사용자 고유 ID |
      | `nickname` | `String` | 닉네임 |
- **오류**:
    - `401 Unauthorized`: `INVALID_TOKEN`
    - `404 Not Found`: `USER_NOT_FOUND`

-----

## 3\. 마트 (Mart)

### 3.1. 전체 마트 목록 조회

- **Endpoint**: `GET /marts`
- **설명**: 서비스에 등록된 모든 마트의 목록을 조회합니다.
- **인증**: 선택
- **요청**: 없음
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<List<MartResponse>>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "전체 마트 목록이 성공적으로 조회되었습니다.",
        "timestamp": "2025-09-22T12:03:00.789Z",
        "data": [
          {
            "martId": 1,
            "martName": "코스트코 양평점",
            "martBrand": "COSTCO",
            "address": "서울특별시 영등포구 선유로 156",
            "latitude": 37.52762720,
            "longitude": 126.89215420
          }
        ]
      }
      ```
    - **상세 스펙 (`data[]`)**:
  
      | 필드 | 타입 | 설명 |
      |---|---|---|
      | `martId` | `Long` | 마트 고유 ID |
      | `martName` | `String` | 마트 이름 (지점 포함) |
      | `martBrand`| `String` | 마트 브랜드 (`COSTCO`, `TRADERS`, `LOTTE_MART`) |
      | `address` | `String` | 주소 |
      | `latitude` | `Double` | 위도 |
      | `longitude`| `Double` | 경도 |
- **오류**: 없음

-----

## 4\. 모임 (Meeting)

### 4.1. 모임 검색 및 목록 조회

- **Endpoint**: `GET /meetings/search`
- **설명**: 키워드를 통해 모임을 검색하거나, 키워드가 없으면 전체 모임 목록을 조회합니다.
- **인증**: 선택
- **요청**:
    - **Query Parameter**:
      | 이름 | 타입 | 필수 | 설명 |
      |---|---|---|---|
      | `keyword`| `String`| 선택 | 검색어 (제목, 내용, 마트 이름) |
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<List<MeetingSimpleResponse>>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "모임 목록이 성공적으로 조회되었습니다.",
        "timestamp": "...",
        "data": [
          {
            "meetingId": 101,
            "martId": 2,
            "title": "코스트코 견과류 소분해요!",
            "description": "아몬드, 호두 등 견과류를 같이 나눠요.",
            "martName": "코스트코 양재점",
            "meetingDate": "2025-09-20T14:00:00",
            "currentParticipants": 1,
            "maxParticipants": 4,
            "status": "RECRUITING",
            "thumbnailImageUrl": "/media/some-image.jpg",
            "latitude": 37.46187560,
            "longitude": 127.03614020
          }
        ]
      }
      ```
    - **상세 스펙 (`data[]`)**:
  
      | 필드 | 타입 | 설명 |
      |---|---|---|
      | `meetingId`| `Long`| 모임 ID |
      | `martId`| `Long`| 마트 ID |
      | `title`| `String`| 모임 제목 |
      | `description`| `String`| 모임 설명 (일부) |
      | `martName`| `String`| 마트 이름 |
      | `meetingDate`| `String`| 모임 날짜 (ISO 8601) |
      | `currentParticipants`| `Integer`| 현재 참여 인원 |
      | `maxParticipants`| `Integer`| 최대 모집 인원 |
      | `status`| `String`| 모임 상태 |
      | `thumbnailImageUrl`| `String`| 썸네일 이미지 URL |
      | `latitude`| `Double`| 마트 위도 |
      | `longitude`| `Double`| 마트 경도 |
- **오류**: 없음

### 4.2. 모임 상세 조회

- **Endpoint**: `GET /meetings/search/{meetingId}`
- **설명**: 특정 모임의 상세 정보를 조회합니다.
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `meetingId` (Long, 조회할 모임 ID)
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<MeetingDetailResponse>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "모임 상세 정보가 성공적으로 조회되었습니다.",
        "timestamp": "...",
        "data": {
          "meetingId": 101,
          "title": "코스트코 견과류 소분해요!",
          "description": "아몬드, 호두 등 견과류를 4명이서 나눠 가져요...",
          "martName": "코스트코 양재점",
          "meetingDate": "2025-09-20T14:00:00",
          "currentParticipants": 2,
          "maxParticipants": 4,
          "status": "RECRUITING",
          "thumbnailImageUrl": "/media/some-image.jpg",
          "hostInfo": {
            "nickname": "김코스트",
            "profileImageUrl": "...",
            "trustScore": 320
          },
          "participants": [
            {"nickname": "김코스트", "profileImageUrl": "...", "participantType": "HOST"},
            {"nickname": "이소분", "profileImageUrl": "...", "participantType": "PARTICIPANT"}
          ]
        }
      }
      ```
    - **상세 스펙 (`data`)**:
  
      | 필드 | 타입 | 설명 |
      |---|---|---|
      | `meetingId`| `Long`| 모임 ID |
      | `title`| `String`| 제목 |
      | `description`| `String`| 상세 설명 |
      | `martName`| `String`| 마트 이름 |
      | `meetingDate`| `String`| 모임 날짜 (ISO 8601) |
      | `currentParticipants`| `Integer`| 현재 인원 |
      | `maxParticipants`| `Integer`| 최대 인원 |
      | `status`| `String`| 모임 상태 |
      | `thumbnailImageUrl`| `String`| 썸네일 이미지 URL |
      | `hostInfo`| `Object`| 호스트 정보 |
      | `participants`| `Array<Object>`| 참여자 목록 |
- **오류**: `401 Unauthorized`, `404 Not Found` (`MEETING_NOT_FOUND`)

### 4.3. 프로필 모임 목록 조회

- **Endpoint**: `GET /meetings/condition`
- **설명**: 주어진 사용자의 특정 참여 상태 모임 목록을 페이징 처리하여 조회합니다.
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `page` (int, 조회할 모임 페이지)
    - **Path Parameter**: `size` (size, 한번에 조회할 모임의 개수)
    - **Path Parameter**: `status` (MeetingParticipant.ApplicationStatus, 사용자의 참여 상태)
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<MeetingProfilePageResponse>`
      - **JSON 응답 예시**:
        ```json
        {
          "success": true,
          "message": "모임 상세 정보가 성공적으로 조회되었습니다.",
          "timestamp": "...",
          "data": {
            "content": [
              {
                "meetingId": 101,
                "title": "코스트코 견과류 소분해요!",
                "description": "아몬드, 호두 등 견과류를 4명이서 나눠 가져요...",
                "martName": "코스트코 양재점",
                "meetingDate": "2025-09-20T14:00:00",
                "currentParticipants": 2,
                "maxParticipants": 4,
                "status": "RECRUITING",
                "thumbnailImageUrl": "/media/some-image.jpg",
                "hostInfo": {
                  "nickname": "김코스트",
                  "profileImageUrl": "...",
                  "trustScore": 320
                },
                "participants": [
                  {"nickname": "김코스트", "profileImageUrl": "...", "participantType": "HOST"},
                  {"nickname": "이소분", "profileImageUrl": "...", "participantType": "PARTICIPANT"}
                ]
              }
            ],
            "page": 0,
            "size": 4,
            "totalElements": 1
          }
        }
        ```
        - **상세 스펙 (`data`)**:

          | 필드                    | 타입              | 설명                 |
          |-----------------------|-----------------|--------------------|
          | `meetingId`           | `Long`          | 모임 ID              |
          | `title`               | `String`        | 제목                 |
          | `description`         | `String`        | 상세 설명              |
          | `martName`            | `String`        | 마트 이름              |
          | `meetingDate`         | `String`        | 모임 날짜 (ISO 8601)   |
          | `currentParticipants` | `Integer`       | 현재 인원              |
          | `maxParticipants`     | `Integer`       | 최대 인원              |
          | `status`              | `String`        | 모임 상태              |
          | `thumbnailImageUrl`   | `String`        | 썸네일 이미지 URL        |
          | `hostInfo`            | `Object`        | 호스트 정보             |
          | `participants`        | `Array<Object>` | 참여자 목록             |
          | `page`                | `int`           | 조회할 모임 페이지         |
          | `size`                | `int`           | 한번에 조회할 모임의 개수     |
          | `totalElements`       | `long`          | 조건에 해당하는 모든 모임의 개수 |
- **오류**: `401 Unauthorized`, `404 Not Found`

### 4.3. 모임 생성

- **Endpoint**: `POST /meetings`
- **설명**: 새로운 소분 모임을 생성합니다.
- **인증**: **필수**
- **요청**:
    - **Content-Type**: `multipart/form-data`
    - **Body Parts**: `request` (JSON), `imageFile` (File, 선택)
- **응답 (201 Created)**:
    - **Body**: `ApiResponse<MeetingCreateResponse>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "모임이 성공적으로 생성되었습니다.",
        "timestamp": "...",
        "data": {
          "meetingId": 102
        }
      }
      ```
    - **상세 스펙 (`data`)**:
  
      | 필드 | 타입 | 설명 |
      |---|---|---|
      | `meetingId`| `Long`| 생성된 모임 ID |
- **오류**: `400 Bad Request`, `401 Unauthorized`, `404 Not Found`

### 4.4. 모임 수정

- **Endpoint**: `PUT /meetings/update/{meetingId}`
- **설명**: 자신이 생성한 모임의 정보를 수정합니다. (호스트만 가능)
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `meetingId` (Long)
    - **Body (`application/json`)**:
      ```json
      {
        "martId": 2,
        "title": "수정된 모임 제목",
        "description": "수정된 설명",
        "meetingDate": "2025-09-21T15:00:00"
      }
      ```
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<MeetingUpdateResponse>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "모임 정보가 성공적으로 수정되었습니다.",
        "timestamp": "...",
        "data": {
          "updatedMeetingId": 101
        }
      }
      ```
    - **상세 스펙 (`data`)**:
  
      | 필드 | 타입 | 설명 |
      |---|---|---|
      | `updatedMeetingId`| `Long`| 수정된 모임 ID |
- **오류**: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

### 4.5. 모임 삭제

- **Endpoint**: `DELETE /meetings/delete/{meetingId}`
- **설명**: 자신이 생성한 모임을 논리적으로 삭제합니다. (호스트만 가능)
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `meetingId` (Long)
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<Void>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "모임이 성공적으로 삭제되었습니다.",
        "timestamp": "...",
        "data": null
      }
      ```
- **오류**: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

### 4.6. 모임 참여 신청

- **Endpoint**: `POST /meetings/{meetingId}/join`
- **설명**: 특정 모임에 참여를 신청합니다.
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `meetingId` (Long)
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<Void>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "모임에 참여 신청되었습니다.",
        "timestamp": "...",
        "data": null
      }
      ```
- **오류**: `401 Unauthorized`, `404 Not Found`, `409 Conflict`

### 4.7. 모임 탈퇴

- **Endpoint**: `POST /meetings/{meetingId}/leave`
- **설명**: 참여 신청했거나 참여 중인 모임에서 탈퇴합니다.
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `meetingId` (Long)
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<Void>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "모임에서 탈퇴하였습니다.",
        "timestamp": "...",
        "data": null
      }
      ```
- **오류**: `401 Unauthorized`, `404 Not Found`

### 4.8. 참여자/신청자 목록 조회

- **Endpoint**: `GET /meetings/{meetingId}/participants`
- **설명**: 모임의 참여자 및 신청자 목록을 조회합니다. (호스트만 가능)
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `meetingId` (Long)
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<ParticipantListResponse>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "참여자 및 신청자 목록이 성공적으로 조회되었습니다.",
        "timestamp": "...",
        "data": {
          "participants": [
            { "id": 1, "nickname": "김호스트", "status": "APPROVED" }
          ],
          "applicants": [
            { "id": 3, "nickname": "박신청", "status": "PENDING" }
          ]
        }
      }
      ```
    - **상세 스펙 (`data`)**:
  
      | 필드 | 타입 | 설명 |
      |---|---|---|
      | `participants`| `Array<Object>`| 확정된 참여자 목록 |
      | `applicants`| `Array<Object>`| 승인 대기중인 신청자 목록 |
- **오류**: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

### 4.9. 참여 신청 승인

- **Endpoint**: `POST /meetings/{meetingId}/participants/{participantId}/approve`
- **설명**: 모임 참여 신청을 승인합니다. (호스트만 가능)
- **인증**: **필수**
- **요청**:
    - **Path Parameters**: `meetingId` (Long), `participantId` (Long)
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<Void>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "참여가 승인되었습니다.",
        "timestamp": "2025-09-22T12:13:00.123Z",
        "data": null
      }
      ```
- **오류**: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

### 4.10. 참여 신청 거절

- **Endpoint**: `POST /meetings/{meetingId}/participants/{participantId}/reject`
- **설명**: 모임 참여 신청을 거절합니다. (호스트만 가능)
- **인증**: **필수**
- **요청**:
    - **Path Parameters**: `meetingId` (Long), `participantId` (Long)
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<Void>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "참여가 거절되었습니다.",
        "timestamp": "2025-09-22T12:14:00.456Z",
        "data": null
      }
      ```
- **오류**: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

### 4.11. 모집 마감

- **Endpoint**: `POST /meetings/{meetingId}/close-recruitment`
- **설명**: 모임의 참여자 모집을 마감합니다. (호스트만 가능)
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `meetingId` (Long)
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<Void>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "모집이 마감되었습니다.",
        "timestamp": "2025-09-22T12:15:00.789Z",
        "data": null
      }
      ```
- **오류**: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

### 4.12. 모임 완료 처리

- **Endpoint**: `POST /meetings/{meetingId}/complete`
- **설명**: 모임을 완료 상태로 변경합니다. (호스트만 가능)
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `meetingId` (Long)
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<Void>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "모임이 완료 처리되었습니다.",
        "timestamp": "2025-09-22T12:16:00.123Z",
        "data": null
      }
      ```
- **오류**: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

-----

## 5\. 댓글 (Comment)

- **기본 경로**: `/api/meetings/{meetingId}/comments`

### 5.1. 댓글 목록 조회

- **Endpoint**: `GET /`
- **설명**: 특정 모임의 모든 댓글을 최신순으로 조회합니다.
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `meetingId` (Long)
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<CommentListDto>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "댓글 목록이 성공적으로 조회되었습니다.",
        "timestamp": "...",
        "data": {
            "comments": [
                {
                    "commentId": 1,
                    "userId": 1,
                    "nickname": "김호스트",
                    "profileImageUrl": "...",
                    "content": "안녕하세요! 호스트입니다.",
                    "createdAt": "2025-09-22T10:00:00"
                }
            ],
            "totalCount": 1
        }
      }
      ```
    - **상세 스펙 (`data`)**:
  
      | 필드 | 타입 | 설명 |
      |---|---|---|
      | `comments`| `Array<CommentReadDto>`| 댓글 객체 배열 |
      | `totalCount`| `Integer`| 전체 댓글 수 |
- **오류**: `401 Unauthorized`, `404 Not Found`

### 5.2. 댓글 작성

- **Endpoint**: `POST /`
- **설명**: 모임에 새로운 댓글을 작성합니다. (참여자, 호스트만 가능)
- **인증**: **필수**
- **요청 Body**: `{"content": "댓글 내용"}`
- **응답 (201 Created)**:
    - **Body**: `ApiResponse<CommentReadDto>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "댓글이 성공적으로 작성되었습니다.",
        "timestamp": "...",
        "data": {
            "commentId": 3,
            "userId": 3,
            "nickname": "박참여",
            "profileImageUrl": "...",
            "content": "댓글 내용",
            "createdAt": "2025-09-22T12:21:00"
        }
      }
      ```
    - **상세 스펙 (`data`)**:
  
      | 필드 | 타입 | 설명 |
      |---|---|---|
      | `commentId`| `Long`| 댓글 ID |
      | `userId`| `Long`| 작성자 ID |
      | `nickname`| `String`| 작성자 닉네임 |
      | `profileImageUrl`| `String`| 작성자 프로필 이미지 |
      | `content`| `String`| 댓글 내용 |
      | `createdAt`| `String`| 생성 시간 (ISO 8601) |
- **오류**: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

### 5.3. 댓글 수정

- **Endpoint**: `PUT /{commentId}`
- **설명**: 자신의 댓글을 수정합니다. (작성자, 호스트만 가능)
- **인증**: **필수**
- **요청**:
    - **Path Parameters**: `meetingId`, `commentId` (Long)
    - **Body**: `{"content": "수정된 댓글 내용"}`
- **응답 (200 OK)**:
    - **Body**: `ApiResponse<CommentReadDto>`
    - **JSON 응답 예시**:
      ```json
      {
        "success": true,
        "message": "댓글이 성공적으로 수정되었습니다.",
        "timestamp": "...",
        "data": {
            "commentId": 3,
            "userId": 3,
            "nickname": "박참여",
            "profileImageUrl": "...",
            "content": "수정된 댓글 내용",
            "createdAt": "2025-09-22T12:21:00"
        }
      }
      ```
- **오류**: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

### 5.4. 댓글 삭제

- **Endpoint**: `DELETE /{commentId}`
- **설명**: 자신의 댓글을 삭제합니다. (작성자, 호스트만 가능). 성공 시 **204 No Content**를 반환합니다.
- **인증**: **필수**
- **요청**:
    - **Path Parameters**: `meetingId`, `commentId` (Long)
- **응답 (204 No Content)**:
    - **Body**: 없음
- **오류**: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

-----

## 6\. 피드백 (Feedback)

- **참고**: 피드백 API는 `CommonResponse` 래퍼를 사용합니다.

### 6.1. 피드백 작성

- **Endpoint**: `POST /feedbacks`
- **설명**: 모임 종료 후 다른 참여자에게 피드백을 남깁니다.
- **인증**: **필수**
- **요청 (`application/json`)**:
  ```json
  {
    "meetingId": 8, "giverId": "1", "receiverId": "2", "feedbackType": "POSITIVE"
  }
  ```
- **응답 (201 Created)**:
    - **Body**: `CommonResponse<FeedbackScoreResponse>`
    - **JSON 응답 예시**:
      ```json
      {
        "status": "success", "message": "피드백이 성공적으로 등록되었습니다.",
        "data": { "userId": 2, "score": 355, "trustGrade": "GOOD" }
      }
      ```
    - **상세 스펙 (`data`)**:
  
      | 필드 | 타입 | 설명 |
      |---|---|---|
      | `userId`| `Long`| 피드백 받은 사용자 ID |
      | `score`| `Integer`| 갱신된 신뢰도 점수 |
      | `trustGrade`| `String`| 갱신된 신뢰도 등급 |
- **오류**: `400 Bad Request`, `401 Unauthorized`, `404 Not Found`, `409 Conflict`

### 6.2. 사용자 피드백 조회

- **Endpoint**: `GET /feedbacks/users/{userId}`
- **설명**: 특정 사용자가 주거나 받은 피드백 목록을 조회합니다.
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `userId` (Long)
    - **Query Parameter**: `type` (String, `RECEIVED` 또는 `GIVEN`)
- **응답 (200 OK)**:
    - **Body**: `CommonResponse<List<FeedbackResponse>>`
    - **JSON 응답 예시**:
      ```json
      {
        "status": "success", "message": "받은 피드백 목록 조회 성공",
        "data": [
          {
            "feedbackId": 1,
            "giverUser": { "userId": 1, "nickname": "김코스트" },
            "receiverUser": { "userId": 2, "nickname": "이소분" },
            "feedbackType": "POSITIVE",
            "createdAt": "2025-09-22T11:00:00"
          }
        ]
      }
      ```
- **오류**: `401 Unauthorized`, `404 Not Found`

### 6.3. 사용자 신뢰도 점수 조회

- **Endpoint**: `GET /feedbacks/users/{userId}/score`
- **설명**: 특정 사용자의 현재 신뢰도 점수와 등급을 조회합니다.
- **인증**: **필수**
- **요청**:
    - **Path Parameter**: `userId` (Long)
- **응답 (200 OK)**:
    - **Body**: `CommonResponse<FeedbackScoreResponse>`
    - **JSON 응답 예시**:
      ```json
      {
        "status": "success", "message": "사용자 신뢰 점수 조회 성공",
        "data": { "userId": 2, "score": 355, "trustGrade": "GOOD" }
      }
      ```
- **오류**: `401 Unauthorized`, `404 Not Found`

-----

## 7\. 변경 이력

| 버전     | 날짜         | 변경 내용                              | 작성자 |
|:-------|:-----------|:-----------------------------------|:----|
| v2.0.1 | 2025.09.23 | 프로필 모임 목록 조회 REST API 추가           | 강관주 |
| v2.0.0 | 2025.09.22 | 백엔드 코드 기반으로 명세서 현행화 및 일부 상세 스펙 추가. | 고동현 |
| v1.0.1 | 2025.09.11 | JPA 엔터티 기반 응답 필드 및 신규 API 추가       | 송민재 |
| v1.0.0 | 2025.09.11 | 초기 문서 작성                           | 송민재 |