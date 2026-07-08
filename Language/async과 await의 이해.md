
## 왜 필요한가?

## 정의

보통 concurrency(동시성)이나 parallel(병렬성)을 위해 사용하는 키워드지만, 내가 착각한 점이 있다.

async을 사용한 지점에서 동시성이나, 병렬의 특징을 적용해 나가는게 아닌,
보통 async, await는 비동기 코드를 동기 코드처럼 보이게하기 위해 사용된다.

그렇다면 어떻게 비동기 코드를 동기 코드처럼 보이게끔 할까?
애초에 비동기 코드와 동기 코드는 어떤 걸까?

## 동기 코드

- 앞의 작업이 끝날 때까지 다음 작업을 하지 않고 무조건 대기한다.(직렬 처리)

## 비동기 코드

- 앞의 작업을 백그라운드에 맡겨 두고, 끝나기를 기다리지 않고 바로 다음 작업으로 넘어간다.

## 동작 과정
이런 특성이 어떻게 비동기 코드를 동기 코드처럼, 위에서 아래로 읽히는 절차적인 방식으로 바꿀 수 있을까?



async/await는
”결과를 받아와서 처리할 다음 코드를 괄호 {} 안으로 집어넣지 않고, 평평하게 아랫줄에 쓸 수 있게 줄바꿈(Line) 형태로 구조를 완전히 폈기 때문이다.


과거의 방식은 다음과 같다.
```
// 가상의 비동기 요청 함수들
const login = () => Promise.resolve({ userId: 1 });
const getProfile = (userId) => Promise.resolve({ name: "홍길동" });

function runWithoutAsyncAwait() {
  console.log("1. 로그인 시작");

  login()
    .then((user) => {
      console.log("2. 로그인 완료, 유저 ID:", user.userId);
      console.log("3. 프로필 요청 시작");

      // ⚠️ 또 다른 비동기를 위해 중괄호 안으로 한 단계 더 들어갑니다 (오른쪽 시선 이동)
      getProfile(user.userId)
        .then((profile) => {
          console.log("4. 프로필 가져오기 완료:", profile.name);
          console.log("5. 환영합니다 메시지 출력");
        }); // 프로필 then 끝

    }); // 로그인 then 끝
}

runWithoutAsyncAwait();
```

과거에는 비동기 코드의 결과를 쓰려면 반드시 함수(콜백)을 만들어서 그 내부에 코드를 가둬야 했다.

여기서 “꺾인다”는 진짜 의미는 코드가 한줄로 이어지지 않고 user를 다루기 위해 오른쪽으로 들여쓰기 된 방으로 들어가고 profile을 다루기 위해 그 방안에 있는 더 작은 방으로 또 들어가는 구조를 의미한다.

하지만 async/await를 사용하면 다음과 같이 세로로 읽을 수 있다.

```
async function runWithAsyncAwait() {
  ⬇️ (오직 아래로만 직진)
  console.log("1. 로그인 시작");

  // 💡 방 안으로 들어가지 않고, 그 자리에서 user를 받아서 바로 다음 줄로 갑니다.
  const user = await login(); 
  
  ⬇️ (오직 아래로만 직진)
  console.log("2. 로그인 완료", user.userId);
  console.log("3. 프로필 요청 시작");

  // 💡 이번에도 방을 만들지 않고, 그 자리에서 profile을 받아 다음 줄로 갑니다.
  const profile = await getProfile(user.userId); 
  
  ⬇️ (오직 아래로만 직진)
  console.log("4. 프로필 완료", profile.name);
  console.log("5. 환영합니다");
}
```

## 프로젝트에서는 어디에서 사용하는가?

- 프론트엔드에서 submit이나 handle을 실행할 때, event에 async/await를 사용한다.
- 프론트엔드에서 axios를 통해 비동기 통신을 하고 있다.
- 프론트엔드에서 error가 발생하면 async/await를 통해 다른 작업도 병행한다.
- fastapi에서 async을 통해 lifespan을 실행한다.
	- 이는 fastapi에서 권장하는 최신 방식이다.
	- lifespan

## 관련 개념


## 참고 자료






