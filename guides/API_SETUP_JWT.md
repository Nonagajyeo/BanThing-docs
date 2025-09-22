# JWT 토큰 설정 가이드

반띵 프로젝트의 인증/인가 시스템을 위한 JWT 토큰 설정 방법을 안내합니다.

## 문서 정보
- **문서명**: 반띵 프로젝트 JWT 토큰 설정 가이드
- **버전**: v1.1.0
- **작성일**: 2025.09.22
- **작성자**: 김경민
- **최종 수정일**: 2025.09.22

---

## 1. JWT 개요

### 1-1. JWT란?
**JSON Web Token**의 줄임말로, 사용자 인증 정보를 안전하게 전달하는 토큰 방식입니다.

### 1-2. 반띵 프로젝트에서의 역할
- **Access Token**: 짧은 시간(15분) 동안 유효한 인증 토큰
- **Refresh Token**: 긴 시간(7일) 동안 유효한 갱신 토큰
- **쿠키 기반 저장**: XSS 공격 방지를 위한 HttpOnly 쿠키 사용

---

## 2. JWT Secret 키 생성

### 2-1. 보안 요구사항
- **최소 256비트 (32자)** 이상의 랜덤 문자열
- **영문 대소문자, 숫자, 특수문자** 조합
- **예측 불가능한** 랜덤 값

### 2-2. Secret 키 생성 방법

#### 방법 1: 온라인 생성기 사용
1. [JWT.io](https://jwt.io/) 접속
2. 하단의 "Verify Signature" 섹션에서 랜덤 키 생성
3. 또는 [Random.org](https://www.random.org/passwords/) 에서 32자 이상 생성

#### 방법 2: 명령어로 생성 (Mac/Linux)
```bash
# OpenSSL 사용
openssl rand -base64 32

# 또는 Python 사용
python3 -c "import secrets; print(secrets.token_urlsafe(32))"
```

#### 방법 3: Node.js로 생성
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

### 2-3. 생성 예시
```
kq2B8xv4zJ9L1t6Q3w8Y2u5R7o0M3n6B9c2E5h8J1k4=
```

---

## 3. .env 파일 설정

### 3-1. JWT 관련 환경변수
백엔드 `.env` 파일에 다음 설정을 추가:

```bash
# JWT 설정
JWT_SECRET=your_generated_secret_key_here
JWT_ACCESS_TOKEN_EXPIRATION=900000
JWT_REFRESH_TOKEN_EXPIRATION=604800000

# COOKIE 설정
COOKIE_ACCESS_TOKEN_MAX_AGE=900
COOKIE_REFRESH_TOKEN_MAX_AGE=604800
```

### 3-2. 설정값 설명

| 설정 | 설명 | 기본값 | 단위 |
|------|------|--------|------|
| **JWT_SECRET** | JWT 서명용 비밀키 | - | 문자열 |
| **JWT_ACCESS_TOKEN_EXPIRATION** | Access Token 유효시간 | 900000 | 밀리초 (15분) |
| **JWT_REFRESH_TOKEN_EXPIRATION** | Refresh Token 유효시간 | 604800000 | 밀리초 (7일) |
| **COOKIE_ACCESS_TOKEN_MAX_AGE** | Access Token 쿠키 유효시간 | 900 | 초 (15분) |
| **COOKIE_REFRESH_TOKEN_MAX_AGE** | Refresh Token 쿠키 유효시간 | 604800 | 초 (7일) |

---

## 4. 시간 설정 가이드

### 4-1. 권장 설정값

#### 개발 환경
```bash
# 개발 편의를 위해 더 긴 시간 설정
JWT_ACCESS_TOKEN_EXPIRATION=3600000    # 1시간
JWT_REFRESH_TOKEN_EXPIRATION=604800000 # 7일
```

#### 운영 환경
```bash
# 보안을 위해 짧은 시간 설정
JWT_ACCESS_TOKEN_EXPIRATION=900000     # 15분
JWT_REFRESH_TOKEN_EXPIRATION=604800000 # 7일
```

### 4-2. 시간 단위 변환표

| 시간 | 밀리초 | 초 |
|------|--------|-----|
| 15분 | 900000 | 900 |
| 30분 | 1800000 | 1800 |
| 1시간 | 3600000 | 3600 |
| 1일 | 86400000 | 86400 |
| 7일 | 604800000 | 604800 |

---

## 5. Spring Security 설정 확인

### 5-1. JWT 필터 체인
프로젝트에 이미 구현된 JWT 관련 클래스들:

```
src/main/java/com/nathing/banthing/config/
├── JwtAuthenticationFilter.java      # JWT 인증 필터
├── JwtTokenProvider.java             # JWT 토큰 생성/검증
├── SecurityConfig.java               # Spring Security 설정
└── OAuth2SuccessHandler.java         # OAuth2 로그인 성공 처리
```

### 5-2. 쿠키 설정
`application.yml`에서 쿠키 관련 설정:

```yaml
cookie:
  domain: localhost
  secure: false          # HTTPS 환경에서는 true
  http-only: true        # XSS 공격 방지
  same-site: Lax         # CSRF 공격 방지
  access-token-name: ACCESS_TOKEN
  refresh-token-name: REFRESH_TOKEN
```

---

## 6. 토큰 플로우 이해

### 6-1. 로그인 프로세스
1. **카카오 OAuth 로그인** 수행
2. **사용자 정보 검증** 후 회원가입/로그인 처리
3. **Access Token & Refresh Token 생성**
4. **HttpOnly 쿠키**로 브라우저에 저장
5. **JWT 토큰**으로 인증 상태 유지

### 6-2. API 요청 프로세스
1. 브라우저가 **쿠키에서 토큰** 자동 전송
2. **JwtAuthenticationFilter**에서 토큰 검증
3. 유효한 토큰이면 **사용자 인증 정보** 설정
4. **컨트롤러**에서 인증된 사용자 정보 사용

### 6-3. 토큰 갱신 프로세스
1. **Access Token 만료** 시 401 오류 발생
2. 프론트엔드에서 **Refresh Token**으로 갱신 요청
3. 유효한 Refresh Token이면 **새 Access Token 발급**
4. **새 토큰으로 요청 재시도**

---

## 7. 보안 고려사항

### 7-1. Secret 키 관리
- ✅ **환경변수**로 관리
- ✅ **충분한 길이** (32자 이상)
- ✅ **정기적 교체** (권장: 3-6개월)
- ❌ 소스코드에 하드코딩 금지
- ❌ Git에 커밋 금지

### 7-2. 토큰 저장 방식
- ✅ **HttpOnly 쿠키** 사용 (XSS 방지)
- ✅ **Secure 플래그** 설정 (HTTPS 환경)
- ✅ **SameSite 설정** (CSRF 방지)
- ❌ localStorage 사용 금지
- ❌ sessionStorage 사용 금지

### 7-3. 토큰 유효시간
- **Access Token**: 짧게 설정 (15-60분)
- **Refresh Token**: 적당하게 설정 (7-30일)
- **자동 로그아웃**: Refresh Token 만료 시

---

## 8. 테스트 방법

### 8-1. 서버 실행 및 로그인 테스트
```bash
# 백엔드 실행
./gradlew bootRun

# 프론트엔드 실행 (새 터미널)
cd frontend
npm run dev
```

### 8-2. JWT 토큰 확인
1. 브라우저에서 `http://localhost:5173` 접속
2. **카카오 로그인** 수행
3. **F12 → Application → Cookies** 확인
4. `ACCESS_TOKEN`, `REFRESH_TOKEN` 쿠키 존재 확인

### 8-3. JWT 토큰 디코딩
[JWT.io](https://jwt.io/)에서 토큰 내용 확인:
1. 쿠키에서 토큰 값 복사
2. JWT.io의 Debugger에 붙여넣기
3. 페이로드 정보 확인

---

## 9. 문제 해결

### Q: "JWT signature does not match" 오류
**해결방법:**
1. `JWT_SECRET` 키가 올바른지 확인
2. 토큰 생성과 검증에 동일한 키 사용 확인
3. 서버 재시작 후 새로 로그인

### Q: 토큰이 쿠키에 설정되지 않음
**해결방법:**
1. 쿠키 도메인 설정 확인 (`localhost`)
2. `HttpOnly` 설정으로 JavaScript에서 접근 불가능함을 이해
3. 개발자 도구 Application 탭에서 확인

### Q: 자동 로그인이 되지 않음
**해결방법:**
1. Refresh Token 유효성 확인
2. 토큰 갱신 로직 확인
3. 쿠키 만료시간 설정 확인

### Q: CORS 오류와 함께 인증 실패
**해결방법:**
1. 백엔드 CORS 설정에서 `credentials: true` 확인
2. 프론트엔드 axios 설정에서 `withCredentials: true` 확인
3. 쿠키 `SameSite` 설정 확인

---

## 10. 운영 환경 고려사항

### 10-1. HTTPS 환경
```bash
# 운영 환경에서는 Secure 쿠키 사용
cookie:
  secure: true          # HTTPS에서만 쿠키 전송
  same-site: Strict     # 더 엄격한 CSRF 방지
```

### 10-2. 로드 밸런서 환경
- **Stateless JWT** 사용으로 서버 간 세션 공유 불필요
- **동일한 Secret 키** 모든 서버 인스턴스에서 사용
- **Redis 기반 Refresh Token** 관리 고려

### 10-3. 모니터링
- **토큰 발급/갱신** 로그 모니터링
- **비정상적인 토큰 사용** 패턴 감지
- **Secret 키 교체** 주기적 수행

---

> JWT 토큰 설정이 완료되면 반띵 서비스에서 안전한 사용자 인증 시스템을 사용할 수 있습니다!

## 변경 이력

| 버전     | 날짜         | 변경 내용                                  | 작성자 |
|--------|------------|----------------------------------------|-----|
| v1.0.0 | 2025.09.22 | 초기 문서 작성                               | 김경민 |