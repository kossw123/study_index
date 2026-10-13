## 왜 필요한가?

- **구조적 수명 주기 관리:** 과거의 `@app.on_event("startup" / "shutdown")` 방식은 시작과 종료 로직이 물리적으로 분리되어 있어, 초기화한 자원을 종료 시점에 공유하거나 에러 발생 시 안전하게 해제하는 구조를 짜기 복잡했다.
- **예외 안전성 및 상태 공유:** `lifespan`은 파이썬의 컨텍스트 매니저 패턴을 활용하여, **앱의 시작부터 종료까지의 자원 생명주기를 하나의 흐름으로 묶어 가독성을 높이고 안전한 예외 처리(Clean-up) 환경**을 제공한다.

## 정의

- **`FastAPI.lifespan`**은 비동기 컨텍스트 매니저(`@asynccontextmanager`)와 `yield` 구문을 활용하여, **애플리케이션 인스턴스의 시작(Startup)과 종료(Shutdown) 시점에 실행될 초기화 및 정리 작업을 정의하는 메커니즘**이다.

### ⚠️ [중요] 실무 적용 시 올바른 기술적 이해

1. **자원 정리의 주체:** `lifespan` 구조를 쓴다고 해서 메모리나 커넥션 누수가 자동으로 방지되는 것은 아니다. `yield` 이후 단계에 명시적으로 커넥션 풀을 닫거나 자원을 해제하는 코드를 개발자가 책임지고 올바르게 작성해야 비로소 누수를 막을 수 있다.
2. **실행 주기와 프로세스 모델:** 시스템 전체에서 무조건 단 한 번만 실행되는 전역 초기화가 아니다. 실제 운영 환경에서 워커(Worker) 프로세스를 다수 띄울 경우, **각 워커별로 구동되는 개별 FastAPI 앱 인스턴스의 수명 주기에 맞춰 각각 실행**된다. (예: 4개의 Uvicorn 워커 가동 시, 총 4번의 lifespan 초기화가 독립적으로 발생)

## 동작 과정

- `yield` 구문을 분기점으로 삼아, 서버가 시작될 때의 준비 작업과 서버가 종료될 때의 정리 작업을 명확히 분리하여 처리한다.

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

# 1. 비동기 컨텍스트 매니저를 통한 수명 주기 함수 정의
@asynccontextmanager
async def lifespan(app: FastAPI):
    # --- [1. 앱 시작(Startup) 시점] ---
    # 각 워커 프로세스가 구동될 때 최초 1회 실행할 중량 작업 정의
    print("해당 워커의 FastAPI 앱이 시작됩니다. 커넥션 풀을 초기화합니다.")
    db_pool = initialize_db_pool()
    
    try:
        # yield를 통해 애플리케이션의 메인 루프(요청 처리 상태)로 제어권을 넘김
        yield 
    finally:
        # --- [2. 앱 종료(Shutdown) 시점] ---
        # 서버 프로세스가 종료 신호(SIGTERM 등)를 받았을 때 실행할 정리 로직
        print("해당 워커의 FastAPI 앱이 종료됩니다. 리소스를 안전하게 반납합니다.")
        db_pool.close()

# 2. 생성자에 lifespan 컴포넌트 등록
app = FastAPI(lifespan=lifespan)
```

## 프로젝트에서는 어디에서 사용하는가?

- **인프라 및 모델 로딩:** 애플리케이션 시작 시 DB 연결 풀 생성, 머신러닝/딥러닝 모델 가중치(Weights)의 메모리 사전 로딩(Pre-loading), 혹은 대규모 캐시(Redis) 클라이언트 초기화 시점에 전면적으로 활용하고 있다.
- **안전한 프로세스 다운:** 배포 시 Blue-Green 무중단 전환이나 서버 스케일 인(Scale-in)으로 인해 프로세스가 종료될 때, 맺어져 있던 소켓 커넥션과 파일 핸들러를 안전하게 닫아주는 관문으로 사용된다.

## 관련 개념

#FastAPI #Lifespan #AsyncContextManager #Yield #ProcessWorker #ResourceManagement #CleanUp

