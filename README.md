# BanThing (반띵) - 대용량 마켓 소분 플랫폼

## 목차

- [프로젝트 정보](#프로젝트-정보)
- [Getting Started](#getting-started)
- [프로젝트 비전](#프로젝트-비전)
  - [BanThing이 해결하는 문제](#banthing이-해결하는-문제)
  - [BanThing의 철학](#banthing의-철학)
  - [플랫폼 차별성](#플랫폼-차별성)
- [주요 기능](#주요-기능)
- [플랫폼 흐름](#플랫폼-흐름)
- [팀 구성](#팀-구성)
- [개발 기간](#개발-기간)
- [기술 스택](#기술-스택)
- [대표 문서](#대표-문서)
  - [전체 문서 폴더](#전체-문서-폴더)
  - [협업 규칙 & 기여 문서](#협업-규칙--기여-문서)
  - [라이선스](#라이선스)
- [향후 업데이트 계획](#향후-업데이트-계획)
  - [Phase 1 (v1.1.0) — 기능 개선](#phase-1-v110--기능-개선)
  - [Phase 2 (v1.2.0) — 기능 확장](#phase-2-v120--기능-확장)
  - [Phase 3 (v2.0.0) — 차세대 확장](#phase-3-v200--차세대-확장)

---

## 프로젝트 정보

| 항목           | 내용                                                                                                                     |
| ------------ |------------------------------------------------------------------------------------------------------------------------|
| **팀명**       | **노나가져** — *"나눠 가져"라는 말의 정겨운 사투리 표현*                                                                                   |
| **프로젝트명**    | **나띵 (Natthing)** — *나누기 + Think*, 단순히 물건을 나누는 것(나누기)을 넘어, '어떻게 하면 더 합리적이고 현명하게 나눌 수 있을까?'에 대한 깊은 고민(Think)에서 시작된 프로젝트 |
| **플랫폼명**     | **반띵 (BanThing)** — *대용량 상품 구매의 부담감을 절반으로 줄이고, 이웃과 함께하는 즐거움을 통해 가치를 채운다*                                               |
| **버전**       | v1.0.0                                                                                                                 |
| **Base URL** | `http://localhost:5173/`                                                                                               |

---

## Getting Started

- [설치 및 실행 가이드](guides/SETUP_GUIDE.md)
- [API 스펙 (GeneralAPI)](architecture/api/general-api-spec.md)
- [API 스펙 (ChatBotAPI)](architecture/api/ai-chatbot-api-spec.md)
- [API 스펙 (KakaoMapAPI)](architecture/api/kakao-map-sdk-spec.md)
- [API 스펙 (KakaoSSOAPI)](architecture/api/kakao-sso-api-spec.md)
- [프로젝트 디렉토리 구조](guides/STRUCTURE_GUIDE.md)

---

## 프로젝트 비전

### BanThing이 해결하는 문제

**반띵**은 창고형 할인점(코스트코, 트레이더스 등)의 대용량 상품 구매 부담을 해결하는 지역 기반 소분 커뮤니티 플랫폼입니다.

**해결하는 핵심 문제:**
- **경제적 부담**: 1-2인 가구의 대용량 상품 구매 부담 해소
- **탐색의 어려움**: 기존 SNS/카페에 파편화된 소분 모임을 중앙화된 플랫폼으로 통합
- **신뢰성 부족**: 투명한 정보 공개와 피드백 시스템으로 안전한 거래 환경 조성
- **접근성 문제**: 지도 기반 직관적 탐색으로 근처 모임을 쉽게 발견

### BanThing의 철학

**"나눔을 통한 합리적 소비와 이웃 간의 연결"**

기술을 통해 잊혀져 가는 '나눔'의 가치를 현대적으로 재해석하고, 단순한 비용 절약을 넘어 사람과 사람을 연결하는 따뜻한 커뮤니티를 만듭니다.

### 플랫폼 차별성

** 지도 기반 탐색**: 카카오 지도 API를 통한 마트 지점별 모임 직관적 확인  
** AI 챗봇 검색**: 자연어 질의를 통한 대화형 모임 검색  
** 간편 로그인**: 카카오 SSO를 통한 별도 회원가입 없는 서비스 이용  
** 댓글 중심 소통**: 참여 신청부터 세부 조율까지 모든 소통을 댓글로 처리  
** 피드백 시스템**: 모임 종료 후 참여자 간 피드백을 통한 신뢰도 관리

---

## 주요 기능

###  메인 페이지 (지도 기반 모임 탐색)
- **카카오 지도 연동**: 마트 지점별 모임 위치 시각화
- **마커 클릭 시 모임 리스트**: 해당 매장의 예정된 소분 모임 목록 확인
- **검색 기능**: 특정 상품이나 지역으로 모임 검색

###  사용자 인증 시스템
- **카카오 SSO 로그인**: 간편한 소셜 로그인
- **권한 관리**: 비회원(조회만)/회원(전체 기능) 구분
- **프로필 관리**: 자기소개, 신뢰도 점수 표시

###  모임 관리 시스템
- **모임 CRUD**: 생성/조회/삭제 기능
- **참여자 관리**: 호스트의 참여 신청 승인/거절
- **상태 자동 관리**: 스케줄러를 통한 모임 상태 자동 변경
- **댓글 시스템**: 모임별 참여 신청 및 소통

### AI 챗봇 기능
- **자연어 검색**: "양재동 근처 세제 소분 찾아줘" 같은 대화형 질의
- **모임 추천**: 사용자 조건에 맞는 모임 추천
- **정보 제공**: 소분 준비물, 방법 등 정보성 질문 답변

### 마이페이지
- **모임 내역 관리**: 참여한/주최한 모임 목록
- **신뢰도 시스템**: 피드백 기반 점수 및 배지
- **프로필 편집**: 자기소개 수정

---

## 플랫폼 흐름

### 1. 사용자 진입
```
비회원 → 메인페이지(지도) → 모임 리스트 조회 → 로그인 유도 → 카카오 SSO → 회원가입 완료
```

### 2. 모임 참여 흐름
```
회원 → 지도/검색으로 모임 탐색 → 모임 상세 확인 → 댓글로 참여 신청 → 호스트 승인 → 모임 참여
```

### 3. 모임 생성 흐름
```
회원 → 모임 생성 → 상품정보/시간/장소 입력 → 생성 완료 → 참여 신청 관리 → 모임 진행
```

### 4. 피드백 시스템
```
모임 종료 → 참여자들 상호 피드백 → 신뢰도 점수 반영 → 배지 등급 업데이트
```

---

## 팀 구성

| 역할 | 이름                                     | 주요 기술 및 담당 기능                                                                                                         | 회고록 링크                                                                                                             |
| -- |----------------------------------------|-----------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| 팀장 | [김경민](https://github.com/minee0505)    | **Backend**: AI 챗봇(Google Gemini API), 모임 삭제/완료, 모임 신청 **Frontend**: 모임 상세 페이지 **Docs**: 회의록, 환경설정 가이드                | [김경민 회고록](https://github.com/minee0505/memoir)                                                                     |
| 팀원 | [고동현](https://github.com/rhehdgus8831) | **Backend**: 모임 생성/조회, 모임 승인/거절   **Frontend**: 모임 생성/메인 페이지(지도), 참여자 관리 **Docs**: 프로젝트 개요, 와이어프레임                    | [고동현 회고록](https://github.com/rhehdgus8831/Project-retrospective/tree/main/Nonagajyeo)                              |
| 팀원 | [강관주](https://github.com/Kanggwanju)   | **Backend**: 카카오 OAuth, Spring Security, JWT 인증 **Frontend**: 로그인/로그아웃, 마이페이지, 약관동의, 공통 헤더 **Docs**: 기능 요구사항 명세서, ERD | [강관주 회고록](https://github.com/Kanggwanju/Nonagajyeo/tree/main/memoir)                                               |
| 팀원 | [송민재](https://github.com/songkey06)    | **Backend**: 피드백/댓글 시스템, 마이페이지 API **Frontend**: 피드백 UI, 댓글 **Docs**: API 명세서, 기술 스택 문서                               | [송민재 회고록](https://github.com/songkey06/Project-Recollection/tree/main/Nonagajyeo) |
| 팀원 | [박현수](https://github.com/hsp64)        | **Docs**: 사용자 플로우, 비즈니스 로직                                                                                            | [박현수 회고록](https://github.com/hsp64/memoir/tree/main/Nonagajyeo) |

> Team 나띵의 팀 구성 및 역할은 [상세 문서](planning/team_roles.md)에서 더 자세히 확인할 수 있습니다.

---

## 개발 기간

**2025.09.05 ~ 2025.09.23 (총 19일)**

| **Phase**                          | **기간**         | **주요 내용**                               |
| ---------------------------------- |----------------|-----------------------------------------|
| **Phase 1. Planning**              | 09.05 ~ 09.11  | 프로젝트 기획, 기술스택 선정, 문서 작성, ERD 설계        |
| **Phase 2. Backend Development**   | 09.12 ~ 09.17  | Spring Boot API 개발, DB 연동, 인증 시스템      |
| **Phase 3. Frontend Development**  | 09.18 ~ 09.21  | React UI 개발, API 연동, 카카오 맵/SSO 연동      |
| **Phase 4. Integration & Testing** | 09.22          | 백엔드-프론트엔드 통합, 기능 테스트, 버그 수정          |
| **Phase 5. Documentation**         | 09.22 ~ 09.23  | 최종 문서 정리, README 완성, 배포 준비           |
| **Phase 6. Final Review**          | 09.23          | 최종 점검, 발표 준비                           |

---

## 기술 스택

| 구분 | 기술 |
|------|------|
| **언어** | ![Java](https://img.shields.io/badge/Java-17-007396?logo=openjdk&logoColor=white) |
| **프레임워크** | ![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5.4-6DB33F?logo=springboot&logoColor=white) |
| **프론트엔드** | ![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white) ![CSS3](https://img.shields.io/badge/CSS3-1572B6?logo=css3&logoColor=white) ![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?logo=javascript&logoColor=black) ![React](https://img.shields.io/badge/React-19.1.1-61DAFB?logo=react&logoColor=black) ![Vite](https://img.shields.io/badge/Vite-Build_Tool-646CFF?logo=vite&logoColor=white) |
| **상태관리** | ![Zustand](https://img.shields.io/badge/Zustand-5.0.8-FF6B35?logo=zustand&logoColor=white) |
| **HTTP** | ![Axios](https://img.shields.io/badge/Axios-HTTP_Client-5A29E4?logo=axios&logoColor=white) |
| **보안/인증** | ![Spring Security](https://img.shields.io/badge/Spring%20Security-6DB33F?logo=springsecurity&logoColor=white) ![JWT](https://img.shields.io/badge/JWT-0.12.3-000000?logo=jsonwebtokens&logoColor=white) |
| **AI 연동** | ![Google Gemini](https://img.shields.io/badge/Google%20Gemini-4285F4?logo=google&logoColor=white) |
| **데이터베이스** | ![MariaDB](https://img.shields.io/badge/MariaDB-003545?logo=mariadb&logoColor=white) ![Hibernate](https://img.shields.io/badge/Hibernate-59666C?logo=hibernate&logoColor=white) ![QueryDSL](https://img.shields.io/badge/QueryDSL-5.0.0-1C8D73) |
| **빌드/의존성 관리** | ![Gradle](https://img.shields.io/badge/Gradle-1.1.7-02303A?logo=gradle&logoColor=white) |
| **테스트** | ![JUnit5](https://img.shields.io/badge/JUnit5-25A162?logo=junit5&logoColor=white) ![Mockito](https://img.shields.io/badge/Mockito-000000?logo=mockito&logoColor=white) ![H2](https://img.shields.io/badge/H2%20Database-007396) |
| **편의 도구** | ![Lombok](https://img.shields.io/badge/Lombok-grey?logo=java&logoColor=white) ![DevTools](https://img.shields.io/badge/Spring%20DevTools-6DB33F) |
| **로깅** | ![Spring Logging](https://img.shields.io/badge/Logging-grey?logo=springboot&logoColor=white) |
| **협업 도구** | ![Git](https://img.shields.io/badge/Git-F05032?logo=git&logoColor=white) ![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github&logoColor=white) ![Discord](https://img.shields.io/badge/Discord-5865F2?logo=discord&logoColor=white) |
| **개발 환경** | ![IntelliJ IDEA](https://img.shields.io/badge/IntelliJ%20IDEA-000000?logo=intellijidea&logoColor=white) ![VS Code](https://img.shields.io/badge/VS%20Code-007ACC?logo=visualstudiocode&logoColor=white) ![Windows11](https://img.shields.io/badge/Windows%2011-0078D6?logo=windows11&logoColor=white) |
| **테스트 환경** | ![Chrome](https://img.shields.io/badge/Chrome-4285F4?logo=googlechrome&logoColor=white) ![Postman](https://img.shields.io/badge/Postman-FF6C37?logo=postman&logoColor=white) |

---

## 대표 문서

- [프로젝트 개요 및 기능](planning/overview.md)
- [플랫폼 시나리오 상세 명세서](user-flow/userFlow.md)
- [시스템 아키텍처](architecture/conceptual_ERD.md)
- [논리 ERD](architecture/Logical_ERD.md)
- [스케줄러 명세서](architecture/scheduler-spec.md)
- [테이블 스키마](architecture/table_schema.md)
- [API 스펙 (GeneralAPI)](architecture/api/general-api-spec.md)
- [API 스펙 (ChatBotAPI)](architecture/api/ai-chatbot-api-spec.md)
- [API 스펙 (KakaoMapAPI)](architecture/api/kakao-map-sdk-spec.md)
- [API 스펙 (KakaoSSOAPI)](architecture/api/kakao-sso-api-spec.md)
- [프로젝트 디렉토리 구조](guides/STRUCTURE_GUIDE.md)

### 전체 문서 폴더

- [Architecture 문서](architecture)
- [Guides 문서](guides)
- [Troubleshooting 문서](trouble-shooting)
- [회의록](meeting-notes)
- [Planning 문서](planning)
- [Presentations 문서](presentations)
- [Rules 문서](rules)

### 협업 규칙 & 기여 문서

- [팀 규칙 허브](rules/workflow.md)
- [GitHub PR/ISSUE 템플릿](.github)

### 라이선스

- [MIT License](LICENSE)

---

## 향후 업데이트 계획

### Phase 1 (v1.1.0) — 기능 개선

**목표**: MVP 피드백 반영 및 사용자 경험 개선

**주요 개선사항:**
- **실시간 알림 시스템**: 참여 신청 승인/거절, 새 댓글 등 푸시 알림
- **모임 관리 강화**: 신고 기능, 모집 마감 자동화
- **AI 챗봇 고도화**: 더 정확한 모임 추천, 대화 맥락 이해 개선
- **UI/UX 개선**: 모바일 반응형 최적화, 로딩 성능 향상

**예상 기간**: 2개월 (2025.10 ~ 2025.11)

---

### Phase 2 (v1.2.0) — 기능 확장

**목표**: 커뮤니티 활성화 및 사용자 참여도 증대

**신규 기능:**
- **실시간 채팅**: 1:1 채팅 및 모임별 그룹 채팅 도입
- **가격 비교 기능**: 마트별 상품 가격 정보 제공
- **모임 템플릿**: 인기 상품별 소분 가이드 템플릿
- **커뮤니티 게시판**: 소분 노하우, 절약 팁 공유 공간
- **모바일 앱**: iOS/Android 네이티브 앱 개발

**예상 기간**: 3개월 (2025.12 ~ 2026.02)

---

### Phase 3 (v2.0.0) — 차세대 확장

**목표**: 전국 서비스 확대 및 비즈니스 모델 구축

**확장 계획:**
- **전국 서비스**: 주요 도시별 단계적 확산
- **안전 결제 시스템**: PG 연동을 통한 안전한 정산 기능
- **파트너십 구축**: 대형마트와의 제휴 할인 혜택
- **수익 모델 도입**:
  - 지역 기반 소상공인 광고
  - 플랫폼 내 결제 수수료
  - 반띵 공식 굿즈(소분 용품) 판매
- **데이터 분석**: 소분 트렌드 분석 및 개인화 추천

**예상 기간**: 6개월 (2026.03 ~ 2026.08)

---

**🎯 최종 비전**: "대용량 상품 소분을 넘어 신뢰 기반 하이퍼로컬 커뮤니티 플랫폼으로 성장"

---