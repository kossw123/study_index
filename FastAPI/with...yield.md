
## 왜 필요한가?

- 자원을 사용하고 관리하기 위해 사용된다.
- with라는 open-close 문법을 축약 시킨 키워드와 yield라는 반환점(endpoint)를 둬서 결과적으로 자원을 사용하고 반환하게 된다.

## 정의

- with는 자원을 획득하고 사용 후 반납해야 하는 코드를 decorator로 축약 시킨 키워드다.

## 동작 과정

yield는 호출자에게 값을 반환하고 함수의 실행을 일시적으로 중지시킨다.

def get_uow():
    with Session(engine) as session:
        yield UnitOfWork(session)

이 코드와 같은 경우는 다음과 같이 절차적으로 실행된다.

1. FastAPI는 Depends 키워드를 만나면 일단 Depends의 인자 함수(get_uow)를 실행 시킨다.
2. with 문법으로 session이라는 이름의 Session(engine) 객체를 생성한다.
3. yield 키워드를 만나 UnitOfWork(session) 객체(generator 함수)를 생성하고 반환한다.
4. Depends가 선언된 함수(상위 함수)로 이동해서 코드를 이어 실행한다.
5. login 검증 흐름을 하고 있으니 다음 코드인 service = AuthService() 이후를 실행하고 return을 반환한 후 다시 yield 키워드로 이동해 yield 다음 코드를 실행한다.
6. scope가 끝나면 with 블록이 끝나면서 자동으로 session.close()를 실행 후 다음 코드를 실행한다.


## 프로젝트에서는 어디에서 사용하는가?

- Transaction 관리 객체(UnitOfWork)로 event, command를 관리한다.
- FastAPI.Session 객체를 with...yield를 통해 관리한다.


## 관련 개념

#transaction , #unitofwork, #python, #with, #yield, #Depends