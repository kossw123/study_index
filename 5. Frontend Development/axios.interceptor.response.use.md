## 왜 필요할까?

- **글로벌 응답 공통 가공:** 모든 API 응답에서 공통으로 발생하는 중복 코드(예: 매번 `response.data`를 추출하는 행위)를 한곳에서 전처리하여 컴포넌트 계층의 코드 생산성을 높인다.
- **중앙 집중식 에러 핸들링 및 세션 복구:** 인증 만료(401), 서버 장애(500) 등의 예외 상황을 전역 관문에서 가로채 일괄적으로 공통 로깅을 남기거나, 만료된 액세스 토큰을 백엔드에서 비동기로 재발급받아 클라이언트가 실패를 인지하지 못하게 **재요청(Silent Refresh)**을 보장하는 인프라를 구축하기 위함이다.

## 정의

- **`axios.interceptors.response.use`**는 Axios 라이브러리에서 제공하는 인터페이스로, 서버로부터 전달된 HTTP 응답(`Response`) 메시지가 애플리케이션의 `then`이나 `catch` 블록으로 **도달하기 직전, 중간에서 응답 패킷을 가로채 제어 흐름을 변경할 수 있게 돕는 글로벌 미들웨어 역체계**이다.

---

## 동작 과정 및 런타임 제약 조건

### ⚠️ [교정] 성공(resolve)과 실패(reject)를 가르는 진짜 기준
1. **`validateStatus` 설정의 종속성:** 기본적으로 Axios는 HTTP 상태 코드가 `2xx` 범위일 때 성공 핸들러를, 그 외에는 에러 핸들러를 실행한다. 하지만 이는 절대적인 법칙이 아니며, 인스턴스 설정인 **`validateStatus` 옵션**의 조건식 변경에 따라 404나 500 오류도 성공 핸들러로 유입되도록 커스텀 분기 기준을 바꿀 수 있다.
2. **HTTP 응답 객체가 없는 에러 (`error.response` 부재):** 모든 에러 핸들러에 HTTP 응답 메시지가 들어오는 것은 아니다. **네트워크 단절(Offline), CORS 정책 위반, 타임아웃(Timeout)**이 발생했을 때는 서버로부터 어떠한 HTTP 응답 메시지도 받지 못하므로 `error.response` 객체가 `undefined` 상태로 유입된다. 따라서 에러 핸들러 내부는 반드시 예외 가드 코드가 수반되어야 한다.

---

## 표준 문법 및 실무 아키텍처 예시 (무한 루프 방지)

> **⚠️ [주의] 인터셉터 무한 재귀 호출 방지 법칙**
> 에러 핸들러 내부에서 실패한 요청을 복구하기 위해 다시 `axios.post`나 `axios(originalRequest)`를 호출할 때, 기본 설정이 주입된 동일한 axios 인스턴스를 무작정 사용하면 **새로 보낸 재요청마저 실패했을 때 다시 에러 인터셉터로 재진입하여 무한 루프**에 빠지게 된다.
> 이를 방지하기 위해 **"이미 재시도한 요청인지 플래그를 검증하는 절차"** 또는 **"인터셉터가 걸려있지 않은 순수 기본 Axios 인스턴스로 토큰 재발급 요청을 보내는 격리 설계"**가 필수적이다.

```javascript
import axios from "axios";

// 1. 응답 인터셉터 등록 구조
axios.interceptors.response.use(
  (response) => {
    // [성공 핸들러] validateStatus 기준(기본값 2xx)에 부합할 때 실행
    // 매번 데이터 레이어 진입 시 .data를 치는 번거로움을 중앙 전처리로 해소
    return response.data;
  },
  async (error) => {
    // [실패 핸들러] 2xx 범위를 벗어나거나 네트워크 단절 시 실행
    const { config, response } = error;

    # 케이스 A: 서버 응답 자체가 없는 네트워크 자체 장애 예외 처리
    if (!response) {
      console.error("네트워크 연결 불안정 또는 CORS 에러 발생");
      return Promise.reject(error);
    }

    # 케이스 B: 401 Unauthorized (인증 만료) 및 무한 루프 방지 조건
    # config._retry 플래그를 활용해 단 1회만 재시도(Retry Count 제한)하도록 락을 겁니다.
    if (response.status === 401 && !config._retry) {
      config._retry = true; // 재시도 마킹을 하여 무한 재귀 진입 차단

      try {
        // ⚠️ 토큰 재발급 요청은 인터셉터가 없는 독립 인스턴스나 별도 라우트를 쓰는 것이 안전합니다.
        const tokenResponse = await axios.post("/v1/auth/refresh");
        const { accessToken } = tokenResponse.data;

        // 기존 실패했던 요청의 헤더를 새 토큰으로 교체한 뒤 재발송
        config.headers["Authorization"] = `Bearer ${accessToken}`;
        return axios(config); 
        
      } catch (refreshError) {
        // 토큰 재발급조차 실패한 경우 세션을 완전히 파괴하고 로그인 페이지로 튕겨냄
        handleLogoutAction();
        return Promise.reject(refreshError);
      }
    }

    // 공통 에러 범위를 벗어난 개별 비즈니스 에러는 컴포넌트 catch 단으로 격리 전파
    return Promise.reject(error);
  }
);
```

---

## 프로젝트에서는 어디에서 사용하는가?

- **프론트엔드 API 클라이언트 네트워크 관문:** `src/api/client.js` 내부에서 전역 Axios 인스턴스를 싱글톤으로 생성할 때 부착하여 사용 중이다.
- **인증 세션 자동 유지 (Silent Refresh):** 유저가 서비스를 이용하는 도중 JWT 액세스 토큰이 만료되어 백엔드가 `401` 상태 코드를 반환했을 때, 사용자가 튕기거나 인지하지 못하도록 뒤편에서 자동으로 리프레시 토큰 유효성을 대조해 세션을 연장하고 기존 유저 행위를 마저 완결해 주는 **인프라 보안 레이어 복구 가드**로 전면 가동되고 있다.

## 관련 개념

- **프론트엔드 상태 아키텍처 결합:** 전역 토큰의 갱신 및 상태 변경 시, 앞서 정비했던 **`Zustand 전역 스토어`**의 `accessToken` 상태 제어 함수를 불러와 연동 보완하는 구조를 띤다.
- **백엔드 매핑 정합성:** 백엔드 레이어인 **`FastAPI`**와 데이터 레이어인 **`SQLModel.Session`**에서 올바르지 않은 자원 접근에 대해 `401` 혹은 `403` 예외를 정확하게 내보내 주어야 프론트엔드의 본 응답 인터셉터가 올바르게 바인딩되어 통신 시퀀스가 성립한다.

#AxiosInterceptor #ResponseInterception #ValidateStatus #NetworkExceptionGuard #InfiniteLoopDefense #SilentRefresh #FrontEndArchitecture
