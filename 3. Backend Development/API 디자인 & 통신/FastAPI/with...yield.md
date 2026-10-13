## 왜 필요할까?

- **안전한 자원 생명주기 관리:** 데이터베이스 커넥션이나 파일 핸들러처럼 사용 후 반드시 닫아주어야(Close) 하는 한정된 자원을 다룰 때, 예외(Error)가 발생하더라도 누수 없이 자원을 확실하게 반납하기 위해 사용된다.
- **선후 처리 분리:** `with` 문법과 `yield` 분기점을 결합하면, 비즈니스 로직이 실행되기 전의 '준비 단계(Setup)'와 실행된 후의 '정리 단계(Teardown)'를 단 하나의 함수 안에서 직관적으로 격리할 수 있다.

## 정의

- **`with` 구문:** 파이썬의 **컨텍스트 매니저(Context Manager)** 프로토콜을 구현한 객체를 다루는 문법으로, 블록에 진입할 때와 빠져나갈 때의 자원 획득/해제 메커니즘을 보장한다.
- **제네레이터 함수 (Generator Function):** 함수 내부에 `yield` 키워드를 사용하여, 실행 상태를 유지한 채 값을 호출자에게 일시적으로 반환하고 나중에 제어권을 다시 돌려받을 수 있는 파이썬의 특수 함수 구조이다.

## 동작 과정 (FastAPI `Depends`와의 연동 흐름)

```python
# get_uow 함수 자체가 '제네레이터 함수'이다.
def get_uow():
    # 1. Setup: SQLAlchemy의 데이터베이스 세션 컨텍스트 오픈
    with Session(engine) as session:
        # 2. Yield: 생성된 세션을 주입한 비즈니스 객체를 라우터로 토스하고 일시 중지
        yield UnitOfWork(session)
        
    # 3. Teardown: 상위 스코프가 끝나면 돌아와 with 블록이 닫히며 session.close() 자동 수행
```

FastAPI 환경에서 `Depends(get_uow)`를 주입받은 엔드포인트는 다음과 같은 정밀한 절차적 메커니즘에 따라 구동된다.

1. **의존성 탐색 및 실행:** 클라이언트 요청이 들어오면 FastAPI는 `Depends` 내부에 선언된 제네레이터 함수 `get_uow()`를 호출한다.
2. **컨텍스트 오픈 및 객체 반환:** `with` 구문에 의해 데이터베이스 세션(`Session`) 객체가 안전하게 생성되고, `yield` 키워드를 만나 비즈니스 트랜잭션 관리자인 `UnitOfWork` 인스턴스를 라우터(상위 함수)로 전달한 뒤 `get_uow` 함수는 그 자리에 멈춘다.
3. **비즈니스 로직 실행:** 라우터와 서비스 계층(`AuthService` 등)이 주입받은 `UnitOfWork` 객체를 활용해 비즈니스 연산 및 DB 쿼리를 정상 수행하고 결과를 `return`한다.
4. **자원 정리 (Clean-up) 타이밍:** 
   - 사용자의 엔드포인트 동작이 완료되고 **최종 HTTP 응답(Response)이 클라이언트에게 안전하게 송신 완료되는 시점(Dependency Scope의 종료)**에 FastAPI는 일시 중지되어 있던 제네레이터 함수로 제어권을 다시 돌려준다.
   - `yield` 하단의 남은 코드가 마저 실행되며, `with` 블록을 탈출하는 순간 컨텍스트 매니저의 작동 원리에 의해 `session.close()`가 확실하게 호출되어 자원이 온전히 반납된다.

## 프로젝트에서는 어디에서 사용하는가?

- **트랜잭션 일관성 보장 (Unit Of Work):** 데이터베이스 쓰기/수정 작업 시 여러 리포지토리의 작업들을 단 하나의 데이터베이스 세션(Single Transaction) 안에서 일관성 있게 커밋(`commit`)하거나 예외 발생 시 롤백(`rollback`)하도록 제어하는 핵심 인프라 파이프라인에 이 패턴을 사용하고 있다.

## 관련 개념

#Python #ContextManager #WithStatement #GeneratorFunction #Yield #FastAPI #Depends #SQLAlchemySession #UnitOfWork
