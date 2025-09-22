# Google Gemini API 설정 가이드

반띵 프로젝트의 AI 챗봇 기능을 위한 Google Gemini API 설정 방법을 안내합니다.

## 문서 정보
- **문서명**: 반띵 프로젝트 Google Gemini API 설정 가이드
- **버전**: v1.1.0
- **작성일**: 2025.09.22
- **작성자**: 김경민
- **최종 수정일**: 2025.09.22

---

## 1. Google AI Studio 접속

[Google AI Studio](https://ai.google.dev/)에 접속하여 Google 계정으로 로그인합니다.

---

## 2. API 키 생성

### 2-1. Get API Key 클릭
- 메인 페이지에서 **"Get API key"** 버튼 클릭

### 2-2. 새 API 키 생성
- **"Create API key"** 클릭
- **"Create API key in new project"** 선택 (또는 기존 프로젝트 선택 가능)

### 2-3. API 키 복사
- 생성된 API 키를 복사하여 안전한 곳에 저장
- 🔒 **중요**: API 키는 한 번만 표시되므로 반드시 복사해 두세요

---

## 3. API 키 제한 설정 (선택사항)

### 3-1. Google Cloud Console 접속
- [Google Cloud Console](https://console.cloud.google.com/) 접속
- Gemini API 키가 생성된 프로젝트 선택

### 3-2. API 및 서비스 > 사용자 인증 정보
- **"API 및 서비스" > "사용자 인증 정보"** 메뉴 이동
- 생성한 API 키 클릭

### 3-3. API 제한 설정 (권장)
**애플리케이션 제한사항:**
- **HTTP 리퍼러(웹사이트)** 선택
- **웹사이트 제한사항**: `http://localhost:9000/*` 추가

**API 제한사항:**
- **키 제한** 선택
- **Generative Language API** 선택

---

## 4. 모델 및 설정 확인

### 4-1. 사용 가능한 모델
반띵 프로젝트에서 사용하는 모델:
- **gemini-1.5-flash**: 빠른 응답, 적은 비용 (기본값)
- **gemini-1.5-pro**: 더 정확한 응답, 높은 비용

### 4-2. API 요금 확인
- [Gemini API 가격 정책](https://ai.google.dev/pricing) 확인
- 무료 티어: 월 1,500회 요청 제한
- 개발 단계에서는 무료 티어로 충분

---

## 5. .env 파일 설정

발급받은 API 키를 백엔드 `.env` 파일에 설정합니다:

```bash
# Google Gemini API 설정
GOOGLE_AI_API_KEY=your_google_gemini_api_key_here
GOOGLE_AI_MODEL=gemini-1.5-flash
GOOGLE_AI_TEMPERATURE=0.7
GOOGLE_AI_MAX_TOKENS=1000
```

### 5-1. 설정값 설명

| 설정 | 설명 | 기본값 | 범위 |
|------|------|--------|------|
| **API_KEY** | Gemini API 키 | - | 필수 |
| **MODEL** | 사용할 모델 | gemini-1.5-flash | flash/pro |
| **TEMPERATURE** | 응답 창의성 | 0.7 | 0.0~2.0 |
| **MAX_TOKENS** | 최대 토큰 수 | 1000 | 1~8192 |

### 5-2. Temperature 설정 가이드
- **0.0**: 가장 일관된 응답 (정확성 중심)
- **0.7**: 균형 잡힌 응답 (기본값)
- **1.0**: 창의적인 응답
- **2.0**: 매우 창의적/무작위 응답

---

## 6. API 테스트

### 6-1. Google AI Studio에서 테스트
1. [AI Studio](https://ai.google.dev/aistudio) 접속
2. **"Text prompt"** 선택
3. 간단한 메시지 입력하여 응답 확인
4. 정상 작동하면 API 키가 올바르게 설정된 것

### 6-2. 백엔드에서 테스트
```bash
# 백엔드 서버 실행
./gradlew bootRun

# 프론트엔드 서버 실행 (새 터미널)
cd frontend
npm run dev
```

### 6-3. 챗봇 기능 테스트
1. 브라우저에서 `http://localhost:5173` 접속
2. 챗봇 아이콘 클릭
3. "안녕하세요" 등의 간단한 메시지 전송
4. AI 응답 확인

---

## 7. 모니터링 및 사용량 확인

### 7-1. Google Cloud Console 모니터링
- [Google Cloud Console](https://console.cloud.google.com/) 접속
- **"API 및 서비스" > "API"** 메뉴
- **"Generative Language API"** 클릭
- **"측정항목"** 탭에서 사용량 확인

### 7-2. 할당량 관리
- **"할당량"** 탭에서 현재 사용량 및 제한 확인
- 무료 티어 한도 초과 시 결제 정보 등록 필요

---

## 8. 보안 및 모범 사례

### 8-1. API 키 보안
-  **환경변수(.env)**에 저장
-  **.gitignore**에 .env 파일 등록
-  소스코드에 직접 하드코딩하지 않기
-  프론트엔드에 노출하지 않기

### 8-2. 요청 최적화
- **적절한 MAX_TOKENS** 설정으로 비용 절약
- **시스템 프롬프트** 최적화로 응답 품질 향상
- **에러 핸들링** 구현으로 안정성 확보

### 8-3. 사용량 관리
- 개발 단계: 무료 티어 활용
- 운영 단계: 사용량 모니터링 및 예산 설정

---

## 9. 문제 해결

### Q: "API key not valid" 오류
**해결방법:**
1. API 키 재확인 및 재설정
2. Google Cloud Console에서 Generative Language API 활성화 확인
3. API 키 제한사항 확인

### Q: "Quota exceeded" 오류
**해결방법:**
1. Google Cloud Console에서 현재 사용량 확인
2. 무료 티어 한도 초과 시 결제 정보 등록
3. 요청 빈도 조절

### Q: 응답이 느리거나 품질이 낮음
**해결방법:**
1. **gemini-1.5-pro** 모델 사용 고려
2. **TEMPERATURE** 값 조정
3. **시스템 프롬프트** 개선

### Q: 한국어 응답 품질 문제
**해결방법:**
1. 시스템 프롬프트에 한국어 응답 명시
2. 예시 대화 포함하여 응답 형식 지정
3. TEMPERATURE 값을 0.5~0.8로 조정

---

> Google Gemini API 설정이 완료되면 반띵 서비스에서 AI 챗봇 기능을 사용할 수 있습니다!

## 변경 이력

| 버전     | 날짜         | 변경 내용                                  | 작성자 |
|--------|------------|----------------------------------------|-----|
| v1.0.0 | 2025.09.22 | 초기 문서 작성                               | 김경민 |