## 왜 필요할까?

- **보일러플레이트 코드의 최소화:** 클래스 상속이나 무거운 단언문 메서드 호출 없이, 파이썬 고유의 순수 `assert` 문 한 줄만으로 모든 조건 검증이 가능하다.
- **압도적인 가독성 및 디버깅 편의성:** 테스트 실패(Failure) 시, 단순 에러 발생에 그치지 않고 어떤 변수에 어떤 구체적인 값들이 유입되어 왜 참이 되지 못했는지 변수의 상태를 터미널에 매우 상세하게 시각화(Failure Assertion Assertion Introspection)해 준다.
- **플러그인 기반 아키텍처:** 코드 커버리지 측정(`pytest-cov`), 멀티 코어를 활용한 동시 병렬 테스트(`pytest-xdist`), 비동기 및 FastAPI 비동기 라우터 연동(`pytest-asyncio`) 등 고도화된 확장 플러그인 생태계가 매우 강력하다.

## 정의

- **`pytest`**는 소형 단위 테스트부터 복잡한 통합 테스트 및 기능 테스트까지 쉽고 확장성 있게 작성할 수 있도록 지원하는 **파이썬의 독보적인 표준 오픈소스 테스트 프레임워크**이다.

### ⚠️ [교정] 호환성 및 테스트 발견(Discovery)의 사양 정책
1. **레거시 호환성 제약:** 파이썬 표준 라이브러리인 `unittest` 기반 테스트 코드는 수정 없이 그대로 흡수하여 구동할 수 있다. (※ 단, 과거에 지원되던 **`nose` 프레임워크 호환 코드는 `pytest 8.0` 버전부터 전면 제거**되었으므로 사용이 불가능하다.)
2. **테스트 발견(Discovery) 규칙:** 단순히 함수 이름이 `test_`로 시작한다고 해서 무조건 스캔되지 않는다. 반드시 **`test_*.py` 또는 `*_test.py` 형태의 명세 파일 구조** 내부나, `Test`로 시작하는 클래스 스코프 내부에 정의되어 있어야만 `pytest` 라우팅 엔진이 테스트 케이스로 수집한다.

---

## `unittest`와 `pytest` 단언문 구조 대조

- **표준 `unittest` (클래스 상속 및 특수 메서드 의존):**
```python
import unittest

class TestString(unittest.TestCase):
    def test_upper(self):
        # unittest.TestCase가 제공하는 전용 메서드를 외워야 함
        self.assertEqual('foo'.upper(), 'FOO')
```

- **모던 `pytest` (순수 함수형 구조 및 파이썬 내장 단언문 활용):**
```python
def test_upper():
    # 별도의 프레임워크 종속적 메서드 없이 순수 파이썬 문법만으로 동작
    assert 'foo'.upper() == 'FOO'
```

---

## 동작 과정 및 피스처(Fixture) 수명 주기 매커니즘

### 💡 피스처(`@pytest.fixture`)의 정밀한 가동 원리
피스처는 무조건 일괄 선행 로드되지 않으며, 세 가지 조건(**명시적 매개변수 요청, `autouse` 여부, 범위 `scope`**)에 의해 철저히 제어된다.

```
[Test Runner 시동] ➔ 테스트 파일 스캔 및 수집 (Discovery)  
│  
▼  
[테스트 유닛 진입] ➔ 테스트 함수가 요구하는 피스처 탐색  
│  
├─> scope="function": 매 테스트 함수 실행 직전 Setup 실행 ➔ yield 호출 ➔ 종료 후 Teardown  
└─> scope="session" : 최초 1회만 공유 Setup 실행 ➔ 전역 테스트 완료 시 최종 Teardown 자원 반납
```

1. **테스트 발견 (Discovery):** 명시된 디렉토리 내부에서 파일명 패턴을 스캔하여 테스트 함수 및 `conftest.py` 전역 설정 파일을 수집한다. 
2. **피스처 분기 실행 및 의존성 주입:** - 개별 테스트 함수가 인자(Argument)로 피스처의 이름을 선언하거나, `@pytest.mark.usefixtures`, 또는 피스처 자체에 `autouse=True` 옵션이 켜져 있을 때 비로소 해당 피스처의 초기화(`Setup`) 코드가 가동된다. 
3. **수명 주기(Scope)에 따른 자원 격리 정책:** - **`scope="function"` (기본값):** 매 테스트 함수가 시작될 때마다 독립적으로 피스처가 가동되며, 함수가 끝나는 즉시 `yield` 하단의 정리 코드(`Teardown`)가 실행되어 자원을 반납한다. - **`scope="session"` / `"module"`:** 전역적인 비용 최적화를 위해 사용된다. 예를 들어, 무거운 `SQLModel` 데이터베이스 엔진이나 외부 모킹 소켓 채널은 전체 테스트 세션 동안 딱 1번만 생성되어 공유되며, 모든 테스트가 완전히 끝난 뒤 러너가 종료되는 시점에 `yield` 이후 로직이 실행되어 자원을 최종 반납한다. 
4. **결과 취합 및 시각화 보고:** 모든 테스트 스퀀스가 완료되면 터미널에 통과(`PASSED`), 실패(`FAILED`), 조건부 스킵(`SKIPPED`) 상태 기호와 통계 리포트를 출력한다. 
## 프로젝트에서는 어디에서 사용하는가? 
- **FastAPI 통합 테스트 파이프라인:** `conftest.py` 내부에 테스트용 독립 데이터베이스 메모리 엔진과 FastAPI 내장 `TestClient` 인스턴스를 생성하는 `@pytest.fixture(scope="session")` 환경을 구성하였다. 
- **CUD 유효성 단언:** 각 도메인 비즈니스 서비스 레이어와 API 라우터 계층으로 더미 데이터 패킷을 쏘아 보낸 후, DB 엔티티 상태 변화 및 HTTP 응답 값이 의도한 불변식 규칙을 충족하는지 `assert` 문으로 철저히 검증하는 테스트 자동화 CI/CD 환경 구축에 활용 중이다. 
## 관련 개념 
#Pytest #PythonTesting #AssertionIntrospection #TestDiscovery #PytestFixture #FixtureScope #TeardownMechanism #TDD #CI_CD_Pipeline