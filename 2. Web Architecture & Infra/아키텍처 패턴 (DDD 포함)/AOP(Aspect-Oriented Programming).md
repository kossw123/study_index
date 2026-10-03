
## 왜 필요한가?

- 객체 중심 프로그래밍(OOP)를 사용하더라도, 애플리케이션 전반에 걸쳐 공통으로 필요한 기능은 여러 모듈에 중복되어 나타난다.
	- 이를 횡단 관심사(Cross-cutting Concerns)라 부른다.
- 중복되는 코드는 당연하게도 유지보수 난이도 상승, 가독성 저하를 일으킨다.

## 정의

전체 코드 중에서 핵심 비즈니스 로직과 공통 부가 기능을 분리하는 방법론이다.
흩어져 있는 횡단 관심사를 한 곳에 모아 독립적인 모듈(Aspect)로 정의해, 원하는 곳에 적용할 수 있다.

## 알아야  할 것 
AOP를 이해하기 위해서는 다음의 주요 개념들을 알아야 합니다: 
[참고](https://www.youtube.com/watch?v=Hm0w_9ngDpM&t=211)

- **Aspect (관점):** 
	- 횡단 관심사를 모듈화한 것입니다. 
	- 부가 기능(어드바이스)과 그것이 적용될 위치(포인트컷)가 함께 정의됩니다. 
	- [참고](https://www.youtube.com/watch?v=Hm0w_9ngDpM&t=211)
- **Advice (어드바이스):** 
	- Aspect가 실제로 수행해야 하는 '부가 기능' 자체를 의미합니다. 
	- [참고](https://www.youtube.com/watch?v=Hm0w_9ngDpM&t=211)
- **Join Point (조인 포인트):** 
	- 어드바이스가 적용될 수 있는 애플리케이션 실행의 특정 지점입니다 
	- (예: 메서드 호출, 예외 발생). 
	- [참고](https://www.youtube.com/watch?v=Hm0w_9ngDpM&t=211), [참고2](https://csg1353.tistory.com/91)
- **Pointcut (포인트컷):** 
	- Join Point 중에서 실제로 어드바이스를 적용할 곳을 필터링하는 조건식입니다. 
	- [참고](https://www.youtube.com/watch?v=Hm0w_9ngDpM&t=211)
- **Target (타겟):** 
	- 어드바이스가 적용되는 실제 객체(클래스)입니다. 
	- [참고](https://www.youtube.com/watch?v=Hm0w_9ngDpM&t=211)
	- [참고2](https://yoon4360.tistory.com/119)
- **Weaving (위빙/엮기):**
    - **정의:** 분리된 애스펙트(Aspect)를 핵심 비즈니스 로직(Target) 코드와 결합하여 AOP가 적용된 최종 결과물을 만드는 **과정**이다.
    - **종류:** 위빙이 일어나는 시점에 따라 3가지 방식으로 분류된다.
        1. **컴파일 시 엮기 (Compile-time Weaving):** 컴파일러가 코드를 빌드할 때 애스펙트 코드를 직접 끼워 넣는 방식 (예: AspectJ).
        2. **클래스 로딩 시 엮기 (Load-time Weaving):** 컴파일된 클래스 파일이 JVM 메모리에 로드될 때 바이트코드를 조작하여 엮는 방식.
        3. **런타임 엮기 (Runtime Weaving):** 소스 코드나 바이너리 변경 없이, 프로그램 실행 중에 **프록시(Proxy) 대리 객체를 동적으로 생성**하여 엮는 방식 (예: Spring AOP).

## 활용 분야

1. 로깅
2. 보안
3. 트랜잭션
4. 성능 모니터링

## 하는 방법

보통 AOP는 3단계 과정을 통해 코드로 작성된다.
1. Target: 핵심 비즈니스 로직 작성
2. Advice: 공통 부가 기능 작성
3. Weaving(Proxy): 두 기능을 엮어서 대리 객체 만들기

```
import time

# 1. 부가 기능 및 위빙 메커니즘 (Aspect & Proxy)
def timer_aop(target_func):
    def proxy():
        print("[시작] 공통 로그")  # Before Advice (실행 전 수행)
        
        try:
            target_func()         # Target (핵심 로직 실행)
        finally:
            # ⚠️ 예외가 발생하더라도 '항상' 수행되는 정리 작업을 구현하려면 
            # 반드시 try...finally 구조를 사용하여 안전장치를 마련해야 합니다.
            print("[종료] 공통 로그")  # After (finally) Advice
            
    return proxy

# 2. 핵심 로직 (Target)
def say_hello():
    print("-> 안녕하세요!")

# 3. 엮기 (Weaving - 런타임 프록시 방식)
# 핵심 함수를 부가 기능으로 감싸서 대리 함수(Proxy)로 바꿉니다.
say_hello = timer_aop(say_hello)

# 4. 실행
say_hello()

```

### 참고 자료

- https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte4.3:fdl:aop

## 관련 개념

#dynamic, #web, #programming, #AOP, #architect 








