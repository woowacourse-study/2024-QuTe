# AOP

AOP(Aspect-Oriented Programming, 관점 지향 프로그래밍)는 프로그램을 여러 관점에서 분리하여 모듈화하는 방법.
객체지향 프로그래밍(OOP)이 주로 데이터와 그 데이터를 다루는 행위를 모듈화하는 반면, AOP는 공통적으로 사용되는 기능(예: 로깅, 트랜잭션 관리, 보안 검사 등)을 코드의 핵심 비즈니스 로직과 분리하여 모듈화하는 것이 특징.

### AOP 핵심 개념

스프링 AOP에서 자주 쓰는 용어와 개념은 다음과 같다:

1. **Aspect(애스펙트)**:
    - 여러 곳에 걸쳐 공통적으로 적용되는 기능이나 관심사를 모듈화한 것.
    - 예를 들어, 로깅이나 보안은 여러 클래스에서 필요하지만, 각 클래스에 중복되는 코드를 넣는 대신, AOP로 따로 관리.
2. **Advice(어드바이스)**:
    - 애스펙트가 적용될 때 실행되는 실제 코드.
    - 어드바이스는 특정 시점에 실행되도록 정의.
    - 종류는 다음과 같습니다:
        - **Before Advice**: 대상 메소드가 실행되기 전에 실행.
        - **After Advice**: 메소드 실행 후에 실행.
        - **After Returning Advice**: 메소드가 정상적으로 반환된 후에 실행.
        - **After Throwing Advice**: 예외가 발생했을 때 실행.
        - **Around Advice**: 메소드 실행 전후에 둘러싸서 실행할 수 있으며, 실행 흐름을 제어할 수 있음.
3. **Join Point(조인 포인트)**:
    - 어드바이스가 적용될 수 있는 모든 지점을 의미.
    - 메소드 호출, 예외 발생, 필드 접근 등이 조인 포인트가 될 수 있다
    - 스프링 AOP에서는 주로 메소드 실행이 조인 포인트로 사용된다.
4. **Pointcut(포인트컷)**:
    - 어드바이스를 어디에 적용할지를 결정하는 지점.
    - 스프링에서는 메소드의 이름, 매개변수, 어노테이션, 클래스 등의 패턴을 정의하여 포인트컷을 지정할 수 있다.
5. **Weaving(위빙)**:
    - 애스펙트를 실제 대상 객체에 적용하는 과정.
    - 스프링에서는 런타임에 동적으로 프록시를 사용하여 위빙을 처리.

### 스프링 AOP 구현 방식

스프링 AOP는 주로 **프록시 패턴**을 사용하여 AOP를 구현.
즉, 스프링은 대상 객체의 프록시를 생성하여 해당 프록시가 대상 객체의 메소드를 호출할 때 AOP 기능을 추가.

### 1. **프록시 방식**

- 스프링 AOP는 기본적으로 **JDK 동적 프록시** 또는 **CGLIB 프록시**를 사용합니다.
    - **JDK 동적 프록시**는 인터페이스가 있을 때 사용되고, 해당 인터페이스를 구현한 프록시 객체를 생성.
    - **CGLIB**은 인터페이스가 없는 경우 클래스의 서브클래스를 만들어 프록시를 생성.

### 2. **AOP 설정**

스프링에서는 `@Aspect`와 `@EnableAspectJAutoProxy`를 통해 AOP를 설정하고 구현할 수 있음.
예시는 다음과 같다:

```java
java
코드 복사
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("메소드 호출 전: " + joinPoint.getSignature().getName());
    }

    @After("execution(* com.example.service.*.*(..))")
    public void logAfter(JoinPoint joinPoint) {
        System.out.println("메소드 호출 후: " + joinPoint.getSignature().getName());
    }

    @Around("execution(* com.example.service.*.*(..))")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("메소드 실행 전: " + joinPoint.getSignature().getName());
        Object result = joinPoint.proceed();  // 실제 메소드 실행
        System.out.println("메소드 실행 후: " + joinPoint.getSignature().getName());
        return result;
    }
}

```

- **`@Aspect`**: 해당 클래스가 애스펙트임을 나타냄.
- **`@Before`, `@After`, `@Around`**: 어드바이스가 언제 실행될지를 결정.
- **Pointcut**: `execution`을 사용하여 특정 패턴에 맞는 메소드들에 대해 어드바이스를 적용. 예를 들어, `execution(* com.example.service.*.*(..))`는 `com.example.service` 패키지의 모든 클래스에 있는 메소드에 대해 어드바이스를 적용.


### 스프링 AOP의 주요 특징

- **런타임 기반 AOP**: 스프링 AOP는 주로 런타임에 동적으로 프록시를 생성하여 동작.
- **메소드 실행에 집중**: 스프링 AOP는 주로 메소드 실행 지점에서 AOP를 적용. 필드 접근이나 생성자 호출에 대한 AOP는 지원하지 않음.
- **경량**: 스프링 AOP는 가볍고, 주로 애플리케이션의 크로스커팅 관심사를 다루기 위한 용도로 사용.

스프링 AOP는 복잡한 코드를 줄이고, 여러 클래스에서 공통적으로 사용되는 로직을 중앙 집중식으로 관리할 수 있게 도와줌.

### 나의 AOP 경험

**로그 전략**

ERROR → RuntimeException 예상치 못한 서버 에러를 handler 함수 실행 전 기록
WARN → BanggoodException, MethodArgumenthotValidException, OauthException 등 사용자 입력 에러를 포함한 서버에서 코드상으로 예외 처리를 한 에러를 handler 함수 실행 전 기록
INFO → Service 클래스 안 Public 메서드 작업 완료 시점, 즉 함수가 끝나는 시점에 기록

로그의 기능과 서비스의 기능을 구분하기 위해 스프링 AOP를 도입
