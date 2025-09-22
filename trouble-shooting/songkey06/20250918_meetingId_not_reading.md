# 🛠 Trouble Shooting

## 📅 발생 일자
- 2025-09-18

---

## 🧩 문제 상황
- 브랜치 병합 후 `MeetingDetailPage.jsx` 컴포넌트에서 React Router의 `useParams` 훅을 사용해 URL의 `id` 값을 읽어오지 못하는 문제 발생. 이로 인해 모임 상세 페이지가 올바르게 렌더링되지 않음.

---

## 🔍 원인 분석
- `MeetingDetailPage.jsx` 파일은 `const { id } = useParams();` 코드를 사용해 URL의 `:id` 파라미터 값을 가져오도록 작성되어 있음.
- 하지만 브랜치 병합 과정에서 라우트를 정의하는 파일(`router-config.jsx` 또는 `App.js` 등)의 코드가 변경되면서, 라우트 경로의 변수명이 `:id`가 아닌 다른 이름(예: `/meetings/:meetingId`)으로 수정되었기 때문.
- 결과적으로 컴포넌트가 예상하는 변수명(`id`)과 실제 라우트의 변수명(`meetingId`)이 일치하지 않아 `undefined` 값을 가져와서 오류가 발생함.

---

## 🛠 해결 방법
다음 두 가지 방법 중 하나를 선택하여 문제를 해결할 수 있습니다.

1.  **라우트 변수명 수정 (권장)**:
    - 라우트 설정 파일(예: `router-config.jsx`)을 열어 `MeetingDetailPage`로 연결되는 라우트의 변수명을 `:id`로 통일합니다.
    ```jsx
    // 라우트 설정 파일
    <Route path="/meetings/:id" element={<MeetingDetailPage />} />
    ```

2.  **컴포넌트 코드 수정**:
    - 라우트의 변수명을 유지해야 한다면, `MeetingDetailPage.jsx` 파일에서 `useParams`를 통해 받는 변수명을 라우트 설정과 동일하게 변경합니다.
    ```jsx
    // MeetingDetailPage.jsx
    import { useParams, useNavigate } from 'react-router-dom';

    const MeetingDetailPage = () => {
        // 라우트 변수명에 맞게 'id'를 'meetingId'로 변경
        const { meetingId } = useParams(); 
        // ...
    };
    ```

---

## ✅ 결과
- 라우트 변수명과 컴포넌트의 `useParams` 변수명을 일치시킨 후, 모임 상세 페이지 URL에서 `id` 값을 정상적으로 읽어와 데이터를 불러오고 화면에 표시되는 것을 확인.

---

## 📚 교훈 / 예방책
- **라우트 변수명 통일**: 팀원 간 협업 시, React Router의 동적 라우트 변수명은 미리 정해진 규칙에 따라 일관되게 사용해야 합니다.(내가 임의대로 바꿔서 생긴 문제)
- **코드 병합 시 검토**: 브랜치 병합 전, 라우트 정의와 관련된 코드 변경 사항이 기존 컴포넌트에 영향을 주지 않는지 교차 확인하는 절차를 마련하여 같은 문제를 예방할 수 있습니다.