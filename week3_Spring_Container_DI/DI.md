# Dependency Injection

DI는 객체 간의 의존성을 외부에서 주입하는 것을 말한다. (객체가 직접 의존성을 생성하지 않음)
DI 컨테이너가 이를 수행하며, 생성자나 setter 메서드 등을 통해 의존성을 주입한다.

**Spring에서는 Spring IoC Container가 빈을 자동으로 관리하고 주입해준다.**

## 의존성 주입 방식

- Constructor-based Dependency Injection (생성자 기반)
- Setter-based Dependency Injection (세터 기반)

---

## 생성자 기반 의존성 주입

컨테이너가 생성자를 호출하여 의존성을 주입하는 방식.

```java
public class SimpleMovieLister {

    // 의존성 대상 필드
    private final MovieFinder movieFinder;

    // 생성자 기반 의존성 주입
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // 비즈니스 로직 생략
}
```

## 세터 기반 의존성 주입

기본 생성자를 사용하여 빈을 인스턴스화하고, setter 메서드를 호출하여 의존성을 주입하는 방식.

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    // setter 메서드를 통한 주입
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // 비즈니스 로직 생략
}
```

---

## 생성자 vs 세터

Spring은 두 방식 모두 지원하며, 동시에 사용할 수도 있다.

Spring 공식 권장 방식:
- **필수 의존성**에는 생성자 주입 사용
- **선택적 의존성**에는 세터 메서드 사용

Spring은 다음과 같은 이유로 생성자 주입을 권장한다:
- 객체의 **불변성** 유지
- **null 방지**: 필수 의존성이 반드시 주입됨을 보장
- 항상 **완전히 초기화된 상태**로 반환됨

생성자의 인자 수가 많다면, 이는 클래스의 책임이 너무 많다는 신호일 수 있다.
세터의 장점은 객체를 **재구성**하거나 **재주입**할 수 있다는 것이지만, 이는 일반적으로 단점으로 간주됨.

---

## 의존성 연결 과정

1. `ApplicationContext` 생성 시 빈 구성 메타데이터를 함께 로드
2. 빈이 생성될 때 의존성 정보를 생성자/세터 등을 통해 주입
3. 컨테이너는 시작 시 빈 구성을 검증함
4. **싱글톤 범위**의 빈은 컨테이너 초기화 시점에 모두 인스턴스화됨 (재사용 목적)
5. **프로토타입 등의 빈**은 요청이 들어올 때마다 생성됨
6. 빈의 생성과정에서 필요한 의존성도 함께 생성되어 주입됨

### 순환 의존성

생성자 주입을 사용하는 경우, 서로가 서로를 참조할 경우 **순환 의존성 문제**가 발생할 수 있다.
- 이 경우 Spring은 `BeanCurrentlyInCreationException` 예외를 던짐
- 이는 초기화되지 않은 빈이 서로를 기다리는 **데드락** 상황임

Spring은 이러한 문제를 방지하기 위해 **ApplicationContext 생성 시 모든 싱글톤 빈을 미리 생성**하여 오류를 조기에 확인할 수 있도록 한다.

---

## Method Injection

### 프로토타입 빈과 싱글톤 빈의 상호작용

의존성을 속성으로 주입할 때, 생명주기가 다르면 문제가 발생함.

예: 싱글톤 A가 프로토타입 B를 의존할 경우
- A는 단 한 번만 생성되므로, B는 한 번만 주입됨
- 따라서 매번 새로운 B가 필요한 경우 작동하지 않음

### 해결 방법

IoC의 일부를 포기하고, 빈 A가 **직접** 컨테이너에서 B를 요청하도록 구성함:

```java
public class A implements ApplicationContextAware {
    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.context = applicationContext;
    }

    public void someMethod() {
        B b = (B) context.getBean("b");
        b.doSomething();
    }
}
```

이처럼 `ApplicationContextAware` 인터페이스를 구현하여,
필요할 때마다 `getBean()`을 호출하면 매번 새로운 프로토타입 빈을 얻을 수 있다.

---

