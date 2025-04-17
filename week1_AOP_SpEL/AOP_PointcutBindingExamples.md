# Advice 메서드에서 대상 정보 바인딩 예시

## 1. 메서드 파라미터 바인딩 (`args`)

```java
@Before("execution(* com.example..*Service.*(..)) && args(userId,..)")
public void logUserId(Long userId) {
    System.out.println("userId: " + userId);
}
```
- `args(userId,..)` → 메서드 첫 번째 파라미터를 `userId`로 바인딩
- 해당 파라미터 값을 Advice에서 바로 사용 가능


## 2. 리턴값 바인딩 (@AfterReturning)

```java

@AfterReturning(
    pointcut = "execution(* com.example..*Service.get*(..))",
    returning = "result")
public void logResult(Object result) {
    System.out.println("리턴값: " + result);
}

```

- 메서드가 정상 종료한 후 실행됨
- `returning = "result"`로 리턴값을 바인딩해서 사용 가능


## 3. 예외 바인딩 (@AfterThrowing)
``` java
@AfterThrowing(
    pointcut = "execution(* com.example..*Service.*(..))",
    throwing = "ex")
public void logError(Throwable ex) {
    System.out.println("예외 발생: " + ex.getMessage());
}
```
- 예외가 발생했을 경우 실행됨
- 예외 객체를 Advice 메서드에 전달받아 활용 가능


## 4. 어노테이션 속성 접근 (@annotation)
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TrackAction {
    String value();
}
```

```java
@Before("@annotation(trackAction)")
public void track(TrackAction trackAction) {
    System.out.println("액션: " + trackAction.value());
}
```
- 메서드에 붙은 어노테이션 객체 자체를 바인딩한다.
- 커스텀 어노테이션에 설정된 값(속성)을 Advice에서 활용 가능


## 5. 다양한 Pointcut 표현식 정리

| 표현식 | 설명 | 예시 |
| `execution()` | 메서드 실행 지점 | `execution(* com.example..*Service.*(..))` |
| `args()` | 메서드 인자 조건 | `args(Long, ..)` |
| `@annotation()` | 메서드에 붙은 어노테이션 | `@annotation(TrackAction)` |
| `within()` | 클래스 범위 지정 | `within(com.example.service..*)` |
| `this()` / `target()` | AOP 적용 객체 타입 | `this(MyService)` → 프록시 타입 / `target(MyService)` → 실제 대상 타입 |
| `bean()` | 빈 이름 기준 | `bean(myService)` |


## 6. SpEL + Advice 조합 예시 (Spring Security)

```java
@PreAuthorize("#user.id == principal.id")
public void updateProfile(User user) {
    ...
}
```
- `#user.id`: 메서드 인자 접근
- `principal.id`: 현재 인증된 사용자 정보 접근
- SpEL을 통해 동적으로 권한 판단


## 7. Around Advice에서 인자/리턴값 활용

```java
@Around("execution(* com.example..*Service.*(..))")
public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
    String methodName = joinPoint.getSignature().getName();
    Object[] args = joinPoint.getArgs();
    System.out.println("실행 전: " + methodName + ", args: " + Arrays.toString(args));

    Object result = joinPoint.proceed();

    System.out.println("실행 후: " + result);
    return result;
}
```
- `joinPoint.getArgs()` → 인자 목록 접근
- `joinPoint.proceed()` → 원본 메서드 실행
- 전/후 처리에 자유롭게 사용 가능