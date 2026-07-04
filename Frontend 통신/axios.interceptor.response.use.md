
## 왜 필요한가?

## 정의

## 동작 과정

다음과 같은 기본 문법을 사용한다.

```
axios.interceptors.response.use(
	(response) => {
	
	// 2xx 범위의 상태 코드가 반환되었을 때 실행
	// 응답 데이터를 가공하여 return
	return response.data;
	},
	(error) => {
	
	// 2xx 범위를 벗어나는 에러 상태 코드가 반환되었을 때 실행
	// 글로벌 에러 처리
	return Promise.reject(error);
	}
);

// 내 프로젝트에는 다음과 같이 사용한다.
axios.interceptors.response.use(
	(response) => {
	
	// 2xx 범위의 상태 코드가 반환되었을 때 실행
	// 응답 데이터를 가공하여 return
	return response.data;
	},
	
	async (error) => {
	
	// 2xx 범위를 벗어나는 에러 상태 코드가 반환되었을 때 실행
	// 글로벌 에러 처리
	const response = await axios.post(...)
	
	...
	
	...
	
	return Promise.reject(error);
	}
);
```



## 프로젝트에서는 어디에서 사용하는가?

## 관련 개념

- [[ORM]]
- [[UnitOfWork]]

## 참고 자료

