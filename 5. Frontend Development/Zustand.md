## 왜 필요한가?

- **[Prop Drilling]**
	- [문제점:]
		- React에서 부모에서 자식 컴포넌트로 props를 통해 데이터를 전달할 때, 중간 과정에 끼어있는 컴포넌트들까지 해당 props로 전달해야 하는 번거로운 상황이 발생한다.
	- [해결책:]
		- "공유 상태가 필요하면 외부에서 store를 만들고 필요한 컴포넌트가 구독하게 하자."
		- zustand 같은 상태 관리 라이브러리를 통해 하나의 state를 전역 변수처럼 취급하여 어느 컴포넌트에서도 접근할 수 있게 한다.


## 정의 

> React 프로그램에서 여러 컴포넌트가 공유해야 하는 상태를 간단하게 관리하기 위한 라이브러리다.

- Store라는 상태와 상태를 변경하는 함수를 다른 컴포넌트가 구독하여 쓸 수 있게끔 한다.

```
                 Store
       ┌──────────┼──────────┐
       ↓          ↓          ↓
     user       cart        count
       ↑          ↑          ↑
    Header      Cart      Counter
```

> 여기서 `count`가 변경된다면 다른 컴포넌트는 재렌더링이 되지 않는다.

```
1. Store
   = 여러 컴포넌트가 공유하는 상태 공간

2. Subscribe
   = 컴포넌트가 필요한 상태를 구독

3. Client State vs Server State
   = Zustand에 모든 서버 데이터를 넣으면 안 됨
```

## 동작 과정

1. Store를 만든다.
```
import { create } from "zustand";

const useStore = create((set) => ({
    count: 0,

    increment: () =>
        set((state) => ({
            count: state.count + 1
        })),
}));
```

2. Store를 사용할 컴포넌트는 Store의 **[특정 상태]** 를 구독한다.
```
function Counter() {
	// 구독
    const count = useStore((state) => state.count);
    const increment = useStore((state) => state.increment);

    return (
        <>
            <div>{count}</div>
            <button onClick={increment}>
                증가
            </button>
        </>
    );
}
```

## Redux와의 비교

> 둘 다 Application의 공유 상태를 관리하지만, 상태를 변경하는 방식과 설계 철학이 다르다.

|항목|Redux|Zustand|
|---|---|---|
|철학|엄격한 상태 변경 흐름|단순한 공유 상태|
|보일러플레이트|상대적으로 많음|적음|
|학습 난이도|높음|낮음|
|상태 변경|Action → Reducer|Store의 함수|
|구조 강제|강함|약함|
|작은 프로젝트|다소 무거울 수 있음|매우 편함|
|큰 프로젝트|구조화하기 좋음|규칙을 잘 정해야 함|
|디버깅|Redux DevTools 강력|가능하지만 상대적으로 단순|
|자유도|낮음|높음|

## 장점

1. [매우 간단하다.]
	- Redux는 Action, Reducer, Slice 등등 상태 하나를 위해 알아야 할게 많다.
	- 반면 zustand는 오직 store의 작성 방법만 알면 된다.
2. [보일러 플레이트가 적다.]
	- 상태 하나를 만들기 위해 많은 파일이 필요가 없다.
3. [컴포넌트 간 공유가 편하다.]
4. [필요한 상태만 구독할 수 있다.]
	- 특정 값만 가져갈 수 있다.
	- 이는 불필요한 렌더링을 줄일 수 있다.
5. [컴포넌트 밖에서도 store를 사용할 수 있다.]
	- Axios interceptor 같은 곳에서 인증 상태를 가져올 수도 있다.
	- 그래서 특정 기능에 유리한 구조를 만들 수 있다.
```
              Zustand
                 │
        accessToken 관리
                 │
         ┌───────┴───────┐
         ↓               ↓
      React           Axios
         │               │
         └────── API ────┘
```

## 단점

1. [규칙이 약하다.]
	- Action 흐름이 없기 때문에 자유롭게 작성할 수 있지만, 반대로 엄격한 흐름이 없어, 프로젝트가 커지면 store가 너무 많이 생긴다.

## 한계점

1. [모든 상태를 zustand에 넣기.]
	- 모든 상태를 store에 넣어서 마치 DB처럼 되는 경우가 있다.
	- 그 결과 서버 데이터는 store를 통해 모든 페이지에 뿌리는 구조가 되어 경직된다.
2. [상태 변경 규칙이 쉽게 무너진다.]
	- store는 너무 자유롭기 때문에 코드 네이밍을 잘못하거나, 커지는 경우에 추적하기가 어렵다.
	- Redux처럼 강제된 Action 흐름이 없기에 팀의 규칙이 중요해진다.
3. [거대한 Store]
	- 하나의 store에 여러 도메인들에서 사용되는 변수, 함수들을 모두 넣는 방식이 발생할 수 있다.
4. [민감 정보 저장]
	- 보안에 민감한 정보들을 `zustand store`에 넣어서 문제가 생길 수 있다.
	- `localStorage` 같은 쿠키에 인증 관련 데이터를 무작정 저장하거나 `zustand persist`를 무분별하게 사용하면 보안상 문제가 발생할 수 있다.

## Zustand를 사용 시 가장 좋은 사고 방식

> 상태를 발견하면 먼저 질문한다.
> 
> **"이 상태의 소유자는 누구인가?"**

- 컴포넌트에서만 사용 -> `useState`
- 여러 컴포넌트가 공유 -> `zustand 사용 고려`
- 서버에서 가져온 데이터 -> `TanStack Query 사용 고려`
- 브라우저에 영속적으로 저장해야 함 -> `zustand persist 고려`
## 관련 개념

#frontend , #react, #state. #library

## 관련 사이트

[zustand 정리 사이트](https://www.heropy.dev/p/n74Tgc)
