## 왜 필요할까?

- **비동기 코드의 가독성 혁신:** 자바스크립트나 파이썬 비동기 환경에서 필연적으로 발생하는 콜백 함수와 `.then()` 체이닝의 구조적 복잡함을 제거한다.
- **동기식 패러다임의 유지:** 비동기적으로 처리되는 제어 흐름을 동기식 코드(위에서 아래로 순차적으로 실행되는 코드)와 완벽히 유사한 형태로 작성할 수 있게 하여, 개발자의 인지 과부하를 줄이고 에러 처리 및 디버깅을 직관적으로 만든다.

## 정의

- **`async` / `await`**는 프로그래밍 언어 레벨에서 제공하는 문법 설탕(Syntactic Sugar)으로, **비동기 작업의 결과(Promise/Future)를 다루는 코드를 동기식 코드 구조처럼 평평하게 작성할 수 있도록 돕는 비동기 추상화 키워드**이다.

### ⚠️ [핵심] 비동기 아키텍처에 대한 올바른 기술적 이해
1. **비동기와 스레드의 관계:** 비동기 처리가 무조건 "백그라운드 스레드의 생성 및 병렬 실행"을 뜻하는 것은 아니다. 자바스크립트나 FastAPI의 비동기는 **싱글 스레드 이벤트 루프(Event Loop)**를 기반으로 하며, CPU 연산이 아닌 네트워크 I/O 등의 대기 시간 동안 스레드를 블로킹하지 않고 다른 이벤트를 먼저 처리하는 동시성(Concurrency) 기법이다.
2. **`await` 뒤의 코드 진행 (일시 정지):** `await`를 만난 코드가 런타임 상에서 곧바로 직진하는 것은 아니다. `await` 구문을 통과하는 순간, **해당 `async` 함수의 나머지 하단 코드 실행은 일시 정지(Suspend)**된다. 이벤트 루프는 비동기 작업이 완료되어 신호를 줄 때까지 제어권을 양도받아 다른 작업을 처리하다가, 작업 완료 시점에 비동기 함수 내부의 남은 로직을 이어 나간다.

---

## 동작 과정 및 코드 제어 흐름 분석

### 1. 과거의 비동기 처리 구조 (Promise Chaining의 한계)
과거 방식도 리턴을 활용해 우측 중첩을 줄일 수는 있었으나, 여전히 비동기 데이터를 활용하려면 `.then()` 내부의 익명 콜백 함수라는 격리된 방 안으로 진입해야만 했다.

```javascript
// 가상의 비동기 요청 함수 스텁 (결과는 Promise 객체)
const login = () => Promise.resolve({ userId: 1 });
const getProfile = (userId) => Promise.resolve({ name: "홍길동" });

function runWithPromise() {
  console.log("1. 로그인 시작");

  // .then() 콜백 스코프 안으로 제어가 꺾여 들어가야만 유저 데이터를 쓸 수 있음
  login()
    .then((user) => {
      console.log("2. 로그인 완료, 유저 ID:", user.userId);
      return getProfile(user.userId); // 체이닝 유도
    })
    .then((profile) => {
      console.log("3. 프로필 가져오기 완료:", profile.name);
      console.log("4. 비즈니스 로직 종료");
    })
    .catch((error) => {
      console.log("에러 처리 전담 영역");
    });
}
```

### 2. 현대의 비동기 처리 구조 (`async` / `await` 코드 플래트닝)
비동기 결과를 담기 위한 별도의 콜백 함수 방을 개설하지 않고, 일반 지역 변수(`const user`)에 값을 담듯 일렬로 평평하게 코드를 작성할 수 있다.

```javascript
async function runWithAsyncAwait() {
  console.log("1. 로그인 시작");

  try {
    // 💡 이 지점에서 login 비동기 작업이 완료될 때까지 현재 함수의 실행 흐름은 잠시 정지(Suspend)됨
    const user = await login(); 
    
    // 비동기 작업이 완료되면 이벤트 루프가 돌아와 아래의 코드를 순차 실행함
    console.log("2. 로그인 완료", user.userId);
    
    const profile = await getProfile(user.userId); 
    console.log("3. 프로필 완료", profile.name);
    
  } catch (error) {
    // 💡 특수한 비동기 에러 객체가 아닌, 표준 동기식 try...catch 구문으로 에러 완벽 통합 제어
    console.error("비동기 처리 중 오류 발생:", error);
  }
}
```

---

## 프로젝트에서는 어디에서 사용하는가?

- **프론트엔드 (React + Vite):** 
  - 사용자 양식 제출 처리기(`handleSubmit`) 등 이벤트 헨들러에 `async`를 지정하여 구동한다.
  - HTTP 통신 라이브러리인 **Axios** 호출 시 `await axios.get()` 패턴을 결합해 백엔드 API 데이터를 동기식 비즈니스 시퀀스 흐름처럼 가져와 상태(State)에 반영한다.
- **백엔드 (FastAPI):** 
  - 어플리케이션 수명 주기를 관장하는 최신 스펙인 **`lifespan` 컨텍스트 매니저** 선언 시 `@asynccontextmanager`와 `async def lifespan(app: FastAPI)` 구조를 사용하여 안전한 비동기 초기화 아키텍처를 구축한다.

## 관련 개념

#AsyncAwait #EventLoop #SingleThreadConcurrency #NonBlockingIO #PromiseSpec #AsynchronousSuspension #TryCatchErrorHandling
