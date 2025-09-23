# 카카오 SSO API 명세서

---

## 문서 정보

- **문서명**: 카카오 SSO API 명세서
- **버전**: v1.0.2
- **작성일**: 2025.09.11
- **작성자**: 강관주
- **최종 수정일**: 2025.09.15

---

## 1\. 개요

이 문서는 Spring Boot 기반 백엔드 애플리케이션의 카카오 소셜 로그인(SSO) API 명세서입니다. HTTP-Only 쿠키 기반 JWT 토큰 인증 방식을 사용하여 보안성을 강화했습니다.

**Base URL**: `http://localhost:9000`
**Frontend URL**: `http://localhost:5173`

## 2\. 인증 방식

- **토큰 타입**: JWT (JSON Web Token)
- **저장 방식**: HTTP-Only 쿠키
- **Access Token 만료시간**: 15분 (900초)
- **Refresh Token 만료시간**: 7일 (604,800초)

## 3\. API 엔드포인트

### 1. 카카오 로그인 시작

카카오 OAuth2 로그인 프로세스를 시작합니다.

```http
GET /oauth2/authorization/kakao
```

#### Response
- **Success**: 카카오 로그인 페이지로 리다이렉트
- **카카오 승인 후**: `/login/oauth2/code/kakao`로 콜백

---

### 2. 카카오 OAuth2 콜백 (자동 처리)

카카오에서 인증 완료 후 자동으로 호출되는 엔드포인트입니다. 직접 호출하지 않습니다.

```http
GET /login/oauth2/code/kakao
```

#### Parameters
| Name  | Type   | Description       |
|-------|--------|-------------------|
| code  | string | 카카오에서 제공하는 인증 코드  |
| state | string | CSRF 방지용 상태값 (선택) |

#### Success Response
- **HTTP Status**: `302 Found`
- **Location**: `http://localhost:5173/popup-close`
- **Set-Cookie**:
    - `ACCESS_TOKEN=<jwt_token>; HttpOnly; Path=/; Domain=localhost; SameSite=Lax; Max-Age=900`
    - `REFRESH_TOKEN=<jwt_token>; HttpOnly; Path=/; Domain=localhost; SameSite=Lax; Max-Age=604800`

#### Error Response
- **HTTP Status**: `302 Found`
- **Location**: `http://localhost:5173/popup-close?from=oauth2&error=<error_message>`

---

### 3. 토큰 갱신

Refresh Token을 사용하여 새로운 Access Token을 발급받습니다.

```http
POST /api/auth/refresh
```

#### Headers
```
Cookie: REFRESH_TOKEN=<refresh_token>
```

#### Success Response
- **HTTP Status**: `204 No Content`
- **Set-Cookie**:
    - `ACCESS_TOKEN=<new_jwt_token>; HttpOnly; Path=/; Domain=localhost; SameSite=Lax; Max-Age=900`

#### Error Response
- **HTTP Status**: `401 Unauthorized`

```json
{
  "timestamp": "2025-09-15T10:30:45",
  "status": 401,
  "error": "UNAUTHORIZED",
  "detail": "인증이 필요합니다.",
  "path": "/api/auth/refresh"
}
```

---

### 4. 로그아웃

사용자의 모든 인증 토큰을 무효화합니다.

```http
POST /api/auth/logout
```

#### Success Response
- **HTTP Status**: `204 No Content`
- **Set-Cookie**:
    - `ACCESS_TOKEN=; HttpOnly; Path=/; Domain=localhost; SameSite=Lax; Max-Age=0`
    - `REFRESH_TOKEN=; HttpOnly; Path=/; Domain=localhost; SameSite=Lax; Max-Age=0`

---

## 인증이 필요한 API 사용법

모든 보호된 엔드포인트는 다음 두 가지 방법 중 하나로 Access Token을 전달받습니다:

### 1. Authorization 헤더 (우선순위 1)
```http
Authorization: Bearer <access_token>
```

### 2. HTTP-Only 쿠키 (우선순위 2)
```http
Cookie: ACCESS_TOKEN=<access_token>
```

## 사용자 정보 구조

카카오 로그인 성공 후 생성/업데이트되는 사용자 정보:

```json
{
  "userId": 1,
  "nickname": "카카오사용자",
  "profileImageUrl": "https://k.kakaocdn.net/dn/.../profile.jpg",
  "selfIntroduction": null,
  "provider": "kakao",
  "providerId": "123456789",
  "trustScore": 300,
  "trustGrade": "BASIC",
  "noShowCount": 0,
  "agree": false,
  "createdAt": "2025-09-15T10:30:45",
  "updatedAt": "2025-09-15T10:30:45",
  "deletedAt": null
}
```

## JWT 토큰 구조

### Access Token
```json
{
  
  "sub": "123456789",  // 카카오 사용자 ID
  "iat": 1726401045,   // 토큰 발급 시간
  "exp": 1726401945    // 토큰 만료 시간 (15분 후)
}
```

### Refresh Token
```json
{
  "sub": "123456789",  // 카카오 사용자 ID 
  "iat": 1726401045,   // 토큰 발급 시간
  "exp": 1727005845    // 토큰 만료 시간 (7일 후)
}
```

## 에러 응답 형식

### 표준 에러 응답
```json
{
  "timestamp": "2025-09-15T10:30:45",
  "status": 400,
  "error": "BAD_REQUEST",
  "detail": "요청이 올바르지 않습니다.",
  "path": "/api/some-endpoint"
}
```


## 보안 고려사항

### 쿠키 설정
- **HttpOnly**: `true` - XSS 공격 방지
- **Secure**: `false` (개발환경) / `true` (운영환경)
- **SameSite**: `Lax` - CSRF 공격 방지
- **Domain**: `localhost` (개발환경)
- **Path**: `/`

### CORS 설정
- **허용 Origin**: `http://localhost:5173`, `http://127.0.0.1:5173`
- **허용 Methods**: `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`
- **허용 Headers**: `*`
- **Credentials**: `true`

## 프론트엔드 통합 가이드

### 1. 로그인 버튼
```javascript
// 카카오 로그인 페이지로 이동
window.location.href = 'http://localhost:9000/oauth2/authorization/kakao';
```

### 2. 토큰 갱신
```javascript
const refreshToken = async () => {
  try {
    const response = await fetch('http://localhost:9000/api/auth/refresh', {
      method: 'POST',
      credentials: 'include' // 쿠키 포함
    });
    
    if (response.status === 204) {
      console.log('토큰 갱신 성공');
      return true;
    }
    return false;
  } catch (error) {
    console.error('토큰 갱신 실패:', error);
    return false;
  }
};
```

### 3. 로그아웃
```javascript
const logout = async () => {
  try {
    await fetch('http://localhost:9000/api/auth/logout', {
      method: 'POST',
      credentials: 'include'
    });
    
    // 로그인 페이지로 이동
    window.location.href = '/login';
  } catch (error) {
    console.error('로그아웃 실패:', error);
  }
};
```

### 4. 인증이 필요한 API 호출
```javascript
const fetchProtectedData = async () => {
  try {
    const response = await fetch('http://localhost:9000/api/protected-endpoint', {
      credentials: 'include' // HTTP-Only 쿠키 자동 포함
    });
    
    if (response.status === 401) {
      // 토큰 갱신 시도
      const refreshed = await refreshToken();
      if (refreshed) {
        // 재시도
        return fetchProtectedData();
      } else {
        // 로그인 페이지로 이동
        window.location.href = '/login';
        return;
      }
    }
    
    return await response.json();
  } catch (error) {
    console.error('API 호출 실패:', error);
  }
};
```

## 환경변수 설정

```bash
# 서버 설정
SERVER_PORT=9000

# 카카오 OAuth2 설정
KAKAO_CLIENT_ID=your_kakao_app_key
KAKAO_CLIENT_SECRET=your_kakao_secret_key

# JWT 설정
JWT_SECRET=your_base64_encoded_secret_key
JWT_ACCESS_TOKEN_EXPIRATION=900000
JWT_REFRESH_TOKEN_EXPIRATION=604800000

# 쿠키 설정  
COOKIE_ACCESS_TOKEN_MAX_AGE=900
COOKIE_REFRESH_TOKEN_MAX_AGE=604800
```

## 테스트 방법

1. **카카오 로그인 테스트**
    - 브라우저에서 `http://localhost:9000/oauth2/authorization/kakao` 접속
    - 카카오 계정으로 로그인
    - `http://localhost:5173`로 리다이렉트 확인
    - 개발자 도구에서 쿠키 확인

2. **토큰 갱신 테스트**
   ```bash
   curl -X POST http://localhost:9000/api/auth/refresh \
     -H "Cookie: REFRESH_TOKEN=<your_refresh_token>" \
     -v
   ```

3. **로그아웃 테스트**
   ```bash
   curl -X POST http://localhost:9000/api/auth/logout \
     -H "Cookie: ACCESS_TOKEN=<your_access_token>; REFRESH_TOKEN=<your_refresh_token>" \
     -v
   ```

---

## 변경 이력

| 버전     | 날짜         | 변경 내용                                    | 작성자 |
|:-------|:-----------|:-----------------------------------------|:----|
| v1.0.0 | 2025.09.11 | 초기 문서 작성 (백엔드 인가 코드 처리 방식 명시)            | 송민재 |
| v1.0.1 | 2025.09.11 | JPA 엔터티 기반 명세 재확인 및 수정                   | 송민재 |
| v1.0.2 | 2025.09.15 | 백엔드 카카오 SSO 개발 진행에 따라 토큰 전달 방식, 세부 사항 변경 | 강관주 |

---