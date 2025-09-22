# 🛠️ 트러블슈팅: React 로그인 팝업 강제 종료 감지 및 인터벌 메모리 누수 문제 해결 기록

## 문서 정보
* **문서명**: React 로그인 팝업 강제 종료 감지 및 인터벌 메모리 누수 문제 해결 기록
* **작성자**: [강관주](https://github.com/Kanggwanju)
* **작성일**: 2025-09-22

---

## 📅 발생 일자
- 2025-09-16

---

## 📌 상황

카카오 로그인 기능을 구현하면서 **팝업 창을 통한 OAuth2 인증 방식**을 사용했다.
사용자가 "카카오 로그인" 버튼을 클릭하면 새 창이 열리고,
인증 완료 후 부모 창으로 결과를 전달하는 구조였다.


---


```javascript
const handleKakaoLogin = () => {
  setIsLoading(true);
  const popup = window.open(KAKAO_AUTH_URL, 'KakaoLoginPopup', 'width=460,height=600');
  // 팝업이 정상적으로 닫힐 때까지 로딩 상태 유지
};
```

---

## 📌 문제

**1. 팝업 강제 종료 시 무한 로딩 상태**

사용자가 카카오 로그인 팝업 창을 **강제로 닫으면** `isLoading` 상태가 `true`로 고정되어
**"로그인 진행 중입니다..."** 메시지가 계속 표시되었다.
---

---
```javascript
// 문제가 있었던 초기 코드
const handleKakaoLogin = () => {
  setIsLoading(true); // ✅ 로딩 시작
  const popup = window.open(KAKAO_AUTH_URL, 'KakaoLoginPopup', 'width=460,height=600');
  // ❌ 팝업이 강제로 닫혔을 때 isLoading을 false로 변경하는 로직 없음
};
```

**2. 사용자 경험 저하**

- 팝업을 실수로 닫아도 **다시 로그인 시도할 수 없음**
- 페이지를 새로고침해야만 정상 상태로 복구
- 사용자가 **로그인이 진행 중인지, 실패한 건지 알 수 없음**

---

## 📌 해결

### 1단계: 인터벌을 통한 팝업 상태 감지

`setInterval`을 사용해 **주기적으로 팝업 창이 닫혔는지 확인**하는 로직을 추가했다.

```javascript
const handleKakaoLogin = () => {
  setIsLoading(true);
  setError('');

  const popup = window.open(KAKAO_AUTH_URL, 'KakaoLoginPopup', 'width=460,height=600');

  // ✅ 팝업 강제 종료 감지를 위한 인터벌 설정
  intervalRef.current = setInterval(() => {
    if (popup.closed) {
      setIsLoading(false); // 로딩 상태 해제
      setError('로그인 창이 닫혔습니다. 다시 시도해 주세요.'); // 사용자 피드백 제공
      clearInterval(intervalRef.current); // 인터벌 정리
    }
  }, 500); // 0.5초마다 체크
};
```

### 2단계: useRef를 통한 인터벌 ID 관리

**인터벌 ID를 안전하게 관리**하기 위해 `useRef` 훅을 사용했다.

```javascript
const intervalRef = useRef(null);

// 기존 인터벌이 있다면 정리 후 새로운 인터벌 설정
if (intervalRef.current) {
  clearInterval(intervalRef.current);
}

intervalRef.current = setInterval(() => {
  // 팝업 상태 체크 로직
}, 500);
```

### 3단계: 컴포넌트 언마운트 시 메모리 누수 방지

**컴포넌트가 언마운트될 때 인터벌을 정리**하여 메모리 누수를 방지했다.

```javascript
// ✅ 컴포넌트 언마운트 시 인터벌 정리
useEffect(() => {
  return () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
    }
  };
}, []);
```
---



---

## 📌 최종 해결 코드

```javascript
const LoginPage = () => {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState('');
  const intervalRef = useRef(null); // ✅ 인터벌 ID 저장

  // ✅ 컴포넌트 언마운트 시 인터벌 정리
  useEffect(() => {
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);

  const handleKakaoLogin = () => {
    setIsLoading(true);
    setError('');

    const popup = window.open(KAKAO_AUTH_URL, 'KakaoLoginPopup', 'width=460,height=600');

    // 팝업 차단 감지
    if (!popup || popup.closed || typeof popup.closed === 'undefined') {
      setIsLoading(false);
      setError('팝업 차단을 해제하고 다시 시도해 주세요.');
      return;
    }

    // ✅ 기존 인터벌 정리
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
    }

    // ✅ 팝업 강제 종료 감지 인터벌 설정
    intervalRef.current = setInterval(() => {
      if (popup.closed) {
        setIsLoading(false);
        setError('로그인 창이 닫혔습니다. 다시 시도해 주세요.');
        clearInterval(intervalRef.current);
      }
    }, 500);
  };
};
```

---

## 📌 결과

**사용자 경험 개선:**
- 팝업을 강제로 닫아도 **즉시 정상 상태로 복구**
- **명확한 에러 메시지**로 사용자에게 상황 안내
- **다시 로그인 시도 가능**한 상태로 UI 복구

**메모리 누수 방지:**
- `useRef`를 통한 **안전한 인터벌 관리**
- 컴포넌트 언마운트 시 **자동 리소스 정리**
- 중복 인터벌 방지로 **성능 최적화**

---

## 📌 배운 점

* **팝업 기반 OAuth2에서는 예외 상황 처리**가 사용자 경험의 핵심이다.
* `setInterval` 사용 시 **메모리 누수 방지를 위한 정리 작업**이 필수다.
* `useRef`는 **리렌더링 간에 값을 유지해야 하는 ID나 참조값 관리**에 적합하다.
* **사용자의 예상치 못한 행동**을 고려한 견고한 예외 처리가 실제 서비스의 품질을 좌우한다.
