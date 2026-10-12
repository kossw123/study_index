## 왜 필요할까?

- **라우트 구조화 및 모듈화:** 거대해지는 애플리케이션의 엔드포인트를 도메인 및 기능별로 깔끔하게 분리하여 관리할 수 있다.
- **중복 코드 제거:** 공통적인 경로 접두사(Prefix)나 보안 의존성(Dependencies), 태그(Tags) 등을 하나의 그룹에 일괄 적용하여 코드 중복을 최소화한다.
- **Swagger UI 자동 문서화 최적화:** 도메인별로 API 그룹을 시각적으로 묶어주어 API 명세서의 가독성을 높인다.

## 정의

- **`APIRouter`**는 FastAPI에서 제공하는 **클래스로, 연관된 경로 작업(Path Operations)들을 하나의 논리적인 집합으로 묶어주는 라우팅 컴포넌트**이다.
- 메인 애플리케이션 객체(`FastAPI()`)에 여러 개의 `APIRouter` 인스턴스를 등록(`include_router`)하는 방식으로 대규모 애플리케이션을 유연하게 확장한다.

## 동작 과정 및 올바른 이해

FastAPI의 라우팅 아키텍처는 들어오는 HTTP 요청의 URL 구조를 기반으로 동작한다.

```
[클라이언트 요청] ──> http://localhost:8000/auth/login  
│  
▼  
[FastAPI 라우팅 엔진] ──> 전체 URL 중 경로(Path)인 `/auth/login` 추출  
│  
├─> 공통 대분류(prefix="/auth") 확인 ──> `auth_router`로 토스  
└─> 상세 소분류(decorator="/login") 매핑 ──> 최종 함수 실행
```

- **대분류 (`prefix`)**: `APIRouter` 인스턴스를 생성할 때 공통으로 사용할 경로를 지정한다. (예: `prefix="/auth"`) 
- **소분류 (Decorator 경로)**: 개별 엔드포인트 함수 선언 시 데코레이터에 세부 경로를 지정한다. (예: `@router.post("/login")`) 
- **시각적 그룹화 (`tags`)**: Swagger UI 문서에서 해당 라우터에 속한 API들을 논리적인 그룹으로 묶어서 보여준다. 
### ⚠️ [중요] CORS(`allow_origins`) 설정과의 관계 교정 

- `CORSMiddleware`의 `allow_origins` 설정은 **서버가 URL 주소를 인식하거나 라우팅하는 규칙과 전혀 무관**하다. 
- 이는 브라우저의 보안 정책인 **CORS(Cross-Origin Resource Sharing)**를 충족하기 위해, "어떤 외부 도메인(Origin)의 프론트엔드 앱이 우리 백엔드 API에 접근할 수 있는지" 허가증을 발급해 주는 방화벽 성격의 설정이다. 

## 프로젝트에서는 어디에서 사용하는가? 

- **도메인별 컨트롤러 분리:** 프로젝트 내의 `users`, `products`, `orders` 등 각 핵심 도메인의 디렉토리마다 `router.py` 파일을 독립적으로 두고 `APIRouter`를 적용하였다. 
- **메인 앱에 통합:** 이렇게 분리되어 설계된 라우터들을 메인 진입점 파일(`main.py`)에서 `app.include_router()`를 통해 통합 등록하는 구조로 프로젝트를 아키텍처링하였다. 
- **Uvicorn과의 연동:** 이렇게 완성된 FastAPI 인스턴스를 고성능 ASGI 서버인 **Uvicorn 런타임**이 로드하여 클라이언트의 요청을 처리하게 된다. 
## 관련 개념 

#FastAPI #APIRouter #CORS #CORSMiddleware #Routing #Uvicorn #CleanArchitecture