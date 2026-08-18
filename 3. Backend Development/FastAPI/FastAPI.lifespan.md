## 왜 필요한가?

- 과거에는 `@app.on_event("startup")`을 썼지만, 구조적인 한계로 인해 코드가 지저분해지고, 비동기 환경에서 안전하지 않았다.
- FastAPI.lifespan은 FastAPI lifecycle에 맞춰 리소스를 단 한 번만 생성하고, 서버가 꺼질 때 메모리 누수 없이 안전하게 정리하게 해준다.
## 정의

- yield를 이용한 앱 시작과 종료 시 필요한 작업을 처리하는 함수다.

## 동작 과정

- yield를 통해 서버가 시작 시 해야할 작업과, 서버가 종료 시 해야할 작업을 구분하여 작업한다.

```
from contextlib import asynccontextmanager
from fastapi import FastAPI

# 1. 수명 주기 함수 정의
@asynccontextmanager
async def lifespan(app: FastAPI):
    # --- [시작(Startup) 이벤트] ---
    print("서버가 시작됩니다! DB나 모델을 불러옵니다.")
    # 예: ml_model = load_model()
    
    yield # 이 부분을 기준으로 시작/종료가 나뉩니다.
    
    # --- [종료(Shutdown) 이벤트] ---
    print("서버가 종료됩니다! 리소스를 정리합니다.")
    # 예: db_connection.close()

# 2. 앱 생성 시 lifespan 등록
app = FastAPI(lifespan=lifespan)

@app.get("/")
def read_root():
    return {"message": "서버가 잘 작동 중입니다."}

```

## 관련 개념

#fastapi , #framework, #backend , #async 
