## FastAPI `Response.set_cookie()` 상세 사양 정책

```python
def set_cookie(
    key: str,
    value: str = "",
    max_age: int | None = None,
    expires: datetime | int | None = None,
    path: str = "/",
    domain: str | None = None,
    secure: bool = False,
    httponly: bool = False,
    samesite: str | None = "lax",
) -> None:
```

### 1. 필수 데이터
- **`key`**: 쿠키의 고유 식별 이름 (Key)
- **`value`**: 쿠키에 저장할 실제 값 (Value)

### 2. 생명주기 및 만료 기한 제어
- **세션 쿠키 (Session Cookie):** `max_age`와 `expires` 속성을 **둘 다 지정하지 않은 경우** 생성된다. 브라우저가 열려있는 동안만 유지되며, 브라우저가 닫히면 자동으로 소멸한다.
- **영속 쿠키 (Persistent Cookie):** 만료 기한을 명시하여 브라우저가 닫혀도 유지되는 쿠키이다. (하단의 두 속성 중 하나만 사용하는 것을 권장)
  - **`max_age`**: 쿠키의 수명을 현재 시점으로부터 **초 단위**로 설정한다. (예: 3600 = 1시간)
  - **`expires`**: 쿠키가 만료될 **정확한 날짜와 시간(`datetime` 객체)**을 지정한다.

### 3. 핵심 웹 보안 설정 (CSRF 및 XSS 방어)
- **`httponly`**
  - `True` 설정 시 브라우저 내의 자바스크립트 코드가 `document.cookie` 등을 통해 이 쿠키에 접근하는 것을 원천 차단한다.
  - **주의:** XSS(Cross-Site Scripting) 공격 자체를 막아주는 방화벽은 아니며, **XSS 취약점이 발생하더라도 악성 스크립트에 의해 토큰이나 쿠키 데이터가 외부로 유출(탈취)되는 치명적인 2차 피해를 방지**하는 대책이다.
- **`secure`**
  - `True` 설정 시 암호화된 `HTTPS` 프로토콜 통신 환경에서만 쿠키가 네트워크를 통해 전송된다. 일반 `HTTP` 환경에서는 쿠키를 전송하지 않아 스니핑 등의 탈취 공격을 예방한다.
- **`samesite`**
  - 크로스 사이트 간의 쿠키 전송 범위를 제한하여 **CSRF(Cross-Site Request Forgery) 공격을 효과적으로 방어**하는 브라우저 정책이다. (※ 기준은 '도메인'이 아닌 주소창의 '레벨 1 사이트' 기준이다.)
  - **`"lax"` (기본값):** 타 사이트에서 우리 사이트로 향하는 크로스 사이트 요청의 경우, 링크 클릭이나 주소창 입력, `GET` 전환과 같은 **안전한 최상위 탐색(Safe Top-level Navigation)** 환경에서만 쿠키 전송을 허용한다. 이미지 로드(`<img>`)나 `POST` 폼 전송 시에는 쿠키를 전송하지 않는다.
  - **`"strict"`:** 서드 파티 환경을 일절 허용하지 않으며, **오직 사용자가 위치한 현재 사이트와 쿠키의 대상 사이트가 완벽히 일치할 때만** 쿠키를 전송한다.
  - **`"none"`:** 모든 크로스 사이트(서드 파티) 환경에서도 쿠키 전송을 허용한다. 
    - **⚠️ 필수 조건:** 현대 브라우저 규격상 `SameSite="none"`을 활성화하려면 **반드시 `secure=True` 설정이 함께 지정**되어야만 브라우저가 쿠키를 정상 저장한다.

### 4. 유효 범위 제어
- **`path`**: 쿠키가 유효한 서버의 웹 경로를 제한한다. 기본값인 `"/"`로 설정 시 서비스 전체 경로에서 쿠키를 전송한다.
- **`domain`**: 쿠키가 전송될 수 있는 사이트 범위를 지정한다. 명시하지 않으면 쿠키를 발급한 현재 호스트 도메인으로 자동 제한되며, 하위 도메인(`://example.com` 등) 간에 쿠키를 공유해야 할 때 명시적으로 상위 도메인(`.example.com`)을 지정하여 범위를 넓힐 수 있다.

## 프로젝트 적용 정책 및 관련 개념

- **인증 토큰(JWT) 관리:** 우리 백엔드 프로젝트에서는 로그인 인증 토큰 발급 시 보안성 극대화를 위해 `httponly=True`, `secure=True`, `samesite="lax"` 조합을 기본 정책으로 채택하여 쿠키를 설정하고 있다. 로컬 테스트 환경(`http://localhost`)에서는 `secure=False` 상태로 검증한 뒤, 실 운영 배포 시 환경변수를 통해 `True`로 스위칭되도록 구현한다.

#FastAPI #set_cookie #HttpOnly #SecureCookie #SameSite #XSS방어 #CSRF방어
