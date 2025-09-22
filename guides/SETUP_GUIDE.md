# **반띵(Banthing) 프로젝트 클론 및 환경설정 가이드**

본 문서는 반띵(Banthing) 프로젝트를 로컬 환경에 클론하여 정상적으로 개발 및 실행하기 위한 환경 설정 방법과 필수 유의사항을 안내합니다.

## 문서 정보
- **문서명**: 반띵(Banthing) 프로젝트 클론 및 환경설정 가이드
- **버전**: v1.1.0
- **작성일**: 2025.09.22
- **작성자**: 김경민
- **최종 수정일**: 2025.09.22

**대상 독자:**
- **백엔드/프론트엔드 개발자**: 개발 환경 세팅 및 실행 방법 숙지
- **QA/테스터**: 테스트 환경 구축 및 재현 환경 설정
- **신규 합류자**: 빠른 온보딩을 위해 로컬에서 프로젝트 실행이 필요한 인원
- **운영자**: 배포 전 로컬 환경 확인 및 기본 실행 점검

---

## **Quick Start (빠른 실행 가이드)**

프로젝트를 바로 실행해보고 싶으신 경우, 아래 절차를 따라주세요.

```bash
# 1. 프로젝트 클론
git clone https://github.com/your-username/banthing.git
cd banthing

# 2. 데이터베이스 생성
# MariaDB 콘솔에서 실행
CREATE DATABASE banthing;

# 3. 백엔드 환경변수 설정
cd backend
cp env.example .env
cd ..                 # 루트로 돌아가기
# .env 파일을 열어서 본인의 API 키로 수정

# 4. 프론트엔드 환경변수 설정
cd frontend
cp env.example .env
cd ..                 # 다시 루트로
# .env 파일을 열어서 본인의 API 키로 수정
cd ..

# 5. 백엔드 실행
./gradlew bootRun   # Mac/Linux
gradlew.bat bootRun # Windows

# 6. 프론트엔드 실행 (새 터미널)
cd frontend
npm install
npm run dev
```

**사전 준비**
* JDK 17 이상
* Node.js 18+ 및 npm
* MariaDB 설치 및 실행
* 각종 API 키 발급 (카카오 OAuth, Google Gemini, 카카오 맵)

---

## **1. 사전 준비 사항**

### **1-1. 개발 환경 설치**

- **JDK 17 이상**: Spring Boot 3.x 호환을 위해 필요
- **Node.js 18+**: React 19.1.1 실행을 위해 필요
- **MariaDB 11.4.x**: 메인 데이터베이스
- **Git**: 소스코드 버전 관리

### **1-2. MariaDB 설치 및 설정**

[MariaDB Server 다운로드](https://mariadb.org/download/)

### **1-3. 데이터베이스 생성**

```sql
CREATE DATABASE banthing;
```

> 💡 DB 이름은 반드시 `banthing`으로 생성해야 합니다.

---

## **2. 프로젝트 클론하기**

```bash
git clone https://github.com/your-username/banthing.git
cd banthing
```

---

## **3. 백엔드 환경설정**

### **3-1. 백엔드 .env 파일 생성**

프로젝트 루트 디렉토리에서:

```bash
cp env.example .env
```

### **3-2. `.env` 파일 설정**

생성된 `.env` 파일을 열어서 본인의 환경에 맞게 수정하세요:

```bash
# 서버 설정
SERVER_PORT=9000

# JWT 설정 (본인만의 비밀키로 변경 필요)
JWT_SECRET=your_jwt_secret_key_minimum_256_bits
JWT_ACCESS_TOKEN_EXPIRATION=900000
JWT_REFRESH_TOKEN_EXPIRATION=604800000

# COOKIE 설정
COOKIE_ACCESS_TOKEN_MAX_AGE=900
COOKIE_REFRESH_TOKEN_MAX_AGE=604800

# OAuth2 설정 (카카오 개발자센터에서 발급)
KAKAO_CLIENT_ID=your_kakao_client_id
KAKAO_CLIENT_SECRET=your_kakao_client_secret

# Google Gemini API 설정 (Google AI Studio에서 발급)
GOOGLE_AI_API_KEY=your_google_gemini_api_key
GOOGLE_AI_MODEL=gemini-1.5-flash
GOOGLE_AI_TEMPERATURE=0.7
GOOGLE_AI_MAX_TOKENS=1000

# 파일 업로드 경로 (본인 PC 환경에 맞게 수정)
FILE_UPLOAD_PATH=C:/banthing_uploads/
```

> ⚠️ **보안 주의**: `.env` 파일은 `.gitignore`에 등록되어 Git에 추적되지 않습니다.

---

## **4. 프론트엔드 환경설정**

### **4-1. 프론트엔드 디렉토리로 이동**

```bash
cd frontend
```

### **4-2. 프론트엔드 .env 파일 생성**

```bash
cp env.example .env
```

### **4-3. 프론트엔드 `.env` 파일 설정**

```bash
# API 서버 URL (Vite용 - VITE_ 접두사 필요)
VITE_API_URL=http://localhost:9000/api

# 챗봇 설정
VITE_CHATBOT_ENABLED=true
VITE_CHATBOT_LOAD_HISTORY=true

# 개발 환경 설정
GENERATE_SOURCEMAP=true
VITE_APP_VERSION=1.0.0

# 카카오 맵 API 키 (카카오 개발자센터에서 발급)
VITE_KAKAO_APP_KEY=your_kakao_map_api_key
```

### **4-4. 의존성 설치**

```bash
npm install
```

---

## **5. API 키 발급 및 설정**

각종 API 키를 발급받아야 합니다. 자세한 방법은 다음 문서들을 참조하세요:

- **[카카오 OAuth 설정 가이드](API_SETUP_KAKAO_OAUTH.md)**
- **[Google Gemini API 설정 가이드](API_SETUP_GOOGLE_GEMINI.md)**
- **[카카오 맵 API 설정 가이드](API_SETUP_KAKAO_MAP.md)**
- **[JWT 토큰 설정 가이드](API_SETUP_JWT.md)**

---

## **6. 프로젝트 실행**

### **6-1. 백엔드 실행**

프로젝트 루트 디렉토리에서:

```bash
./gradlew bootRun   # Mac/Linux
gradlew.bat bootRun # Windows
```

백엔드 서버는 `http://localhost:9000`에서 실행됩니다.

### **6-2. 프론트엔드 실행**

새 터미널을 열고:

```bash
cd frontend
npm run dev
```

프론트엔드는 `http://localhost:5173`에서 실행됩니다.

### **6-3. 개발 환경 초기화**

1. `dev` 브랜치 최신화
2. DB 콘솔에서 `CREATE DATABASE banthing;` 실행
3. 서버 실행 시 테이블 자동 생성 및 더미데이터 자동 삽입

---

## **7. 관련 문서**

환경 설정 외 프로젝트 관련 상세 정보는 다음 문서들을 참조하세요:

- **[프로젝트 구조 가이드](STRUCTURE_GUIDE.md)** - 디렉토리 구조 및 파일 역할
- **[기술 스택 명세서](../tech/tech-stack.md)** - 사용 기술 및 라이브러리 상세
- **[데이터베이스 스키마](../architecture/table_schema.md)** - 테이블 구조 및 관계
- **[프로젝트 개요](../planning/overview.md)** - 서비스 목표 및 핵심 기능

---

## **8. HTTP 통신**

- **공통**: `apiClient` 파일의 axios 인스턴스만 사용
- **아이콘**: React Icons 공식 사이트 활용

---

## **9. 파일 구조**

```
banthing/
├── backend/
│   ├── .env                       # 백엔드 환경변수
│   ├── .env.example               # 백엔드 환경변수 예시
│   ├── src/main/java/             # Spring Boot 소스코드
│   ├── src/main/resources/        # 백엔드 리소스 및 설정
│   ├── build.gradle               # Gradle 빌드 파일
│   └── gradlew                    # Gradle Wrapper
├── frontend/
│   ├── .env                       # 프론트엔드 환경변수
│   ├── .env.example               # 프론트엔드 환경변수 예시
│   ├── src/                       # React 소스코드
│   ├── package.json               # npm 패키지 설정
│   └── vite.config.js             # Vite 빌드 설정
└── README.md                      # 프로젝트 메인 설명서

```
> 📁 별도 레포지토리: 문서(docs)는 별도 레포지토리에서 관리
> 🔗 자세한 구조는 **[프로젝트 구조 가이드](STRUCTURE_GUIDE.md)** 참조

---

## **10. 자주 발생하는 문제들** 

**Q: MariaDB 연결 오류**
A: `.env` 파일의 데이터베이스 설정 및 MariaDB 서비스 실행 상태 확인

**Q: API 키 관련 오류**
A: `.env` 파일의 API 키 설정 및 유효성 확인

**Q: 프론트엔드 빌드 오류**
A: Node.js 버전 확인 (18+ 필요) 및 `npm install` 재실행

**Q: 포트 충돌 오류**
A: `.env` 파일에서 `SERVER_PORT` 변경 또는 사용 중인 포트 종료
 

---

## **11. 문의 및 지원**

* **GitHub**: [https://github.com/Nonagajyeo]
* **팀 Discord**: 개발 관련 문의 및 소통
* **문서**: `docs/` 디렉토리의 각종 가이드 참조

---

> **환경변수 설정을 완료한 후 프로젝트를 실행해주세요. API 키 발급이 어려우시면 팀 문서를 참조하거나 팀원에게 문의하세요.**
 
---

## 변경 이력

| 버전     | 날짜         | 변경 내용                                  | 작성자 |
|--------|------------|----------------------------------------|-----|
| v1.0.0 | 2025.09.12 | 초기 문서 작성                               | 김경민 |
| v1.1.0 | 2025.09.22 | .env 기반 환경설정으로 전면 개편, API 키 발급 가이드 분리  | 김경민 |


---