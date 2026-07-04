

## 왜 필요한가?

- 도메인 별로 경로를 나눠서 구조화 시키고 중복 코드를 제거할 수 있다.
- Swagger UI 자동 문서화를 통해 한 눈에 알아볼 수 있다.

## 정의

APIRouter는 주소를 쪼개서 인식하는 파일 분리 시스템이다.

즉, 주소를 쪼개서 각기 다른 파일에 APIRouter 객체를 선언하고 FastAPI 객체에 추가함으로써 여러 주소를 쉽게 관리하는 라이브러리다.

## 동작 과정

APIRouter는 다음과 같이 주소를 쪼개서 인식한다.

" 대분류 + 소분류로 나눠서 인식한다”

예를 들어서 다음과 같은 주소가 있을 때를 가정한다.

http://localhost:3000/users/profiles

보통 axios와 같은 서버와 통신하는 API는 전체 주소를 보내는데,
fastapi는 미들웨어에서 설정한 allow_origins의 값들로 도메인 주소를 인식한다.
그리고 뒤에 있는 경로를 대분류 + 소분류로 분류한다.

auth_router = APIRouter(prefix="/auth", tags=["login"])

APIRouter에서 대분류는 prefix로 분류하고 소분류는 router 함수를 선언할 때 decorator를 통해 소분류 경로만 지정하면 된다.

tags는 swagger UI에서 router들을 시각적으로 묶어서 보여준다.


## 프로젝트에서는 어디에서 사용하는가?

- 백엔드에서 ASGI(uvicorn)을 만들기 위해 해당 프레임워크를 사용한다.

## 관련 개념

#fastapi, #backend, #router, #uvicorn 





