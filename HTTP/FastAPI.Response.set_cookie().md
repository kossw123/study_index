  


```
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

# 필수 데이터
    - key: 쿠키의 이름
    - value: 쿠키에 저장할 실제 값
      
# 만료 기한 제어(둘 중 하나만 사용 권장)
    - max_age: 쿠키의 수명을 초 단위로 설정, 따로 설정안하면 세션 쿠키가 된다.
    - expires: 쿠키가 만료될 정확한 날짜와 시간을 설정

# 핵심 보안 설정
    - httponly: True로 설정하면 자바스크립트로 이 쿠키에 접근할 수 없어서 XSS 공격이 방어된다.
    - secure: https:// 환경에서만 쿠키가 전송된다.
    - samesite: 서로 다른 도메인 간 쿠키가 전송되는 범위를 제한해 CSRF 공격 방어
        - "lax": 기본값, 사용자가 링크를 클릭해 이동될 때만 쿠키 전송
        - “strict": 오직 우리 도메인 주소 내에서 쿠키 전송 허용
        - ”none": 서드 파티에서도 쿠키 전송
# 유효 범위 제어
    - path: 쿠키가 어느 경로에서 유효할지 지정한다.
    - domain: 쿠키가 전송될 도메인을 제한한다.
