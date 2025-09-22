# 카카오 맵 SDK 적용 명세서

---

## 문서 정보

- **문서명**: 카카오 맵 SDK 적용 명세서
- **버전**: v1.0.0
- **작성일**: 2025.09.22
- **작성자**: [고동현](https://github.com/rhehdgus8831)
- **최종 수정일**: 2025.09.22
- **관련 기술 스택**: `react-kakao-maps-sdk`, Kakao Maps Javascript SDK v2

---

## 1. 개요

본 문서는 '반띵' 서비스의 프론트엔드에서 **카카오 맵 API**를 어떻게 사용하는지 구체적인 기능과 컴포넌트, 로직을 정의합니다. 백엔드 REST API 명세가 아닌, 클라이언트(React)단에서의 지도 SDK 활용법을 다룹니다.

---

## 2. 핵심 기능 및 구현 방식

'반띵'의 지도 기능은 `KakaoMap.jsx` 컴포넌트를 통해 구현되며, 별도의 SDK 래퍼 컴포넌트(`<Map>`) 대신 Kakao 공식 Javascript SDK를 직접 호출하여 다음의 핵심 기능을 구현합니다.

| 기능 | 구현 방식 | 설명 |
| :--- | :--- | :--- |
| **지도 스크립트 로딩** | `loadKakaoMapScript` 헬퍼 함수 | Kakao 지도 SDK 스크립트를 동적으로 페이지에 삽입합니다. |
| **지도 렌더링** | `window.kakao.maps.Map` | 서비스의 메인 페이지(`MeetingListPage`)에 지도를 표시하는 기본 컨테이너를 생성합니다. |
| **마트 위치 표시** | `window.kakao.maps.Marker` | 백엔드 API로부터 받은 마트 좌표에 커스텀 아이콘(`TbShoppingCartFilled`)을 사용하여 핀(마커)을 표시합니다. |
| **선택된 마트 정보 표시**| React State 및 JSX | 마커 클릭 시, 선택된 마트의 이름을 지도 좌측 상단에 별도의 UI로 표시합니다 (`TbMapPinFilled` 아이콘 사용). |

---

## 3. 기능 상세 구현 로직

### 3.1. 메인 페이지 지도 초기화 및 마커 생성

- **관련 파일**: `frontend/src/pages/MeetingListPage.jsx`, `frontend/src/components/Meeting/KakaoMap.jsx`
- **로직**:
    1.  `MeetingListPage` 컴포넌트가 마운트될 때, `useEffect` 훅을 통해 `searchMeetings` API를 호출하여 전체 모임 및 마트 정보를 가져옵니다.
    2.  가져온 `meetings` 데이터를 `KakaoMap` 컴포넌트에 props로 전달합니다.
    3.  `KakaoMap` 컴포넌트는 `useEffect` 내에서 `loadKakaoMapScript` 함수를 호출하여 카카오맵 SDK를 로드합니다.
    4.  SDK 로드가 완료되면 `window.kakao.maps.load` 콜백 함수 내에서 지도를 초기화합니다.
    5.  전달받은 `meetings` 배열에서 중복을 제거한 고유한 마트 목록을 만들고, 각 마트의 위치에 커스텀 아이콘을 사용한 마커를 생성하여 지도에 표시합니다.

```jsx
// KakaoMap.jsx
useEffect(() => {
    // ...
    const initializeMap = async () => {
        // ...
        await loadKakaoMapScript(); // 스크립트 로드
        window.kakao.maps.load(() => { // 지도 초기화
            // ...
            uniqueMarts.forEach((meeting) => { // 마커 생성
                // ...
                const marker = new window.kakao.maps.Marker({ /* ... */ });
                // ...
                marker.setMap(map);
            });
        });
    };
    initializeMap();
}, [meetings, onMarkerClick]);
````

### 3.2. 마커 클릭 및 모임 목록 필터링

- **관련 파일**: `frontend/src/pages/MeetingListPage.jsx`, `frontend/src/components/Meeting/KakaoMap.jsx`
- **로직**:
    1.  `KakaoMap` 컴포넌트 내에서 각 마커에 `click` 이벤트 리스너를 추가합니다.
    2.  마커 클릭 시, `onMarkerClick(meeting.martId)` 콜백 함수가 호출됩니다. 이 함수는 `MeetingListPage`에 정의되어 있습니다.
    3.  `handleMarkerClick` 함수는 `selectedMartId`와 `selectedMartName` 상태를 업데이트합니다. 이 상태 변경으로 인해 `filteredMeetings`가 재계산됩니다.
    4.  `MeetingList` 컴포넌트는 필터링된 모임 목록만 props로 받아 화면에 렌더링합니다.
    5.  `KakaoMap` 컴포넌트는 업데이트된 `selectedMartName`을 받아 지도 위에 현재 선택된 마트 이름을 표시합니다.

### 3.3. 선택된 마트 초기화

- **관련 파일**: `frontend/src/pages/MeetingListPage.jsx`, `frontend/src/components/Meeting/KakaoMap.jsx`
- **로직**:
    1.  지도 위에 선택된 마트 이름이 표시될 때, 'X' 버튼이 함께 노출됩니다.
    2.  이 버튼을 클릭하면 `onClearSelectedMart` 콜백 함수가 호출됩니다.
    3.  `MeetingListPage`에 정의된 `handleClearSelectedMart` 함수는 `selectedMartId`와 `selectedMartName`을 `null`로 초기화하여 전체 모임 목록이 다시 표시되도록 합니다.

-----

## 4\. API 키 관리

- **보안**: 카카오 맵 JavaScript SDK를 사용하기 위한 `APP_KEY`는 노출되어도 무방한 `JavaScript 키`를 사용합니다.
- **관리**: 프로젝트 루트의 `.env` 파일에 `VITE_KAKAO_APP_KEY` 환경 변수로 키를 저장합니다. `KakaoMap.jsx`의 `loadKakaoMapScript` 함수 내에서 `import.meta.env.VITE_KAKAO_APP_KEY`를 통해 키를 동적으로 참조하여 스크립트 URL을 생성합니다.

<!-- end list -->

```javascript
// KakaoMap.jsx > loadKakaoMapScript
const kakaoMapKey = import.meta.env.VITE_KAKAO_APP_KEY;
// ...
script.src = `//dapi.kakao.com/v2/maps/sdk.js?appkey=${kakaoMapKey}&autoload=false`;
```

-----

## 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
| :--- | :--- | :--- | :--- |
| v1.0.0 | 2025.09.22 | 클라이언트 측 SDK 사용법 중심으로 초기 문서 작성 | 고동현 |

```