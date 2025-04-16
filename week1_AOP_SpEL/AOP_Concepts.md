# Spring AOP Concepts

## AOP (Aspect Oriented Perspective)
- AOP는 OOP를 보완한다. 클래스가 핵심 단위인 OOP와 달리, Aspect를 핵심 단위로 둔다.
- Aspect는 트랜잭션 관리와 같이, 여러 클래스에 걸쳐 반복되는 **공통 기능(트랜잭션, 로깅, 보안, 캐싱 등)** - 횡단 관심사를 모듈화할 수 있게 만든다.
- 횡단 관심사는, 모든 서비스에서 공통적으로 필요하나 OOP 관점에서 볼 때 각 클래스의 주 목적은 아닌 기능을 말한다. (cross-cutting 하듯이 여러 클래스나 메서드에 공통으로 들어가는 기능이라는 뜻)

## AOP는 어떤 문제를 해결하기 위해 등장했을까?
- OOP(객체지향 프로그래밍)는 공통 관심사(cross-cutting concern)를 모듈화하기 어렵다는 한계가 있었다.
- 모든 서비스에 로그, 트랜잭션, 권한 검사 등 공통적이고 반복되는 코드가 쌓이면 핵심 비즈니스 로직을 깔끔하게 유지하기 어려워진다.

    ➞ 즉, 반복적이고 횡단적인 관심사를 Aspect로 분리해 코드 중복을 제거하면 응집도 높은 비즈니스 로직을 유지할 수 있다.



## AOP 핵심 개념
### (+번외) AspectJ가 뭘까?
- Java용 정통 AOP 프레임워크로 메서드 실행뿐 아니라 생성자 호출, 필드 접근까지 JoinPoint로 처리가 가능하다.
- 컴파일타임 위빙, 클래스 로딩 시 위빙을 지원하며
- Spring AOP는 AspectJ에서 출발하여(각종 용어들의 출처) AOP 개념 및 표현식을 가져와서, Spring에 맞게 경량화하고 쉽게 쓸 수 있도록 만든 것이라고 생각하면 된다.

| 항목 | AspectJ | Spring AOP |
|------|---------|-------------|
| 프레임워크 성격 | 정통 AOP 프레임워크 | 경량 AOP 지원 프레임워크 (스프링 통합) |
| 위빙(weaving) 방식 | 컴파일 타임 (ajc), 클래스 로딩 시 (LTW), 런타임 가능 | 런타임 시 프록시 생성 |
| 프록시 방식 | 바이트코드 조작 (`.class` 에 직접 Aspect 코드 삽입) | 프록시 객체 생성 (JDK or CGLIB 방식) |
| 지원 Join Point | 메서드, 생성자, 필드 접근, 예외 처리 등 거의 모든 지점 | 메서드 실행만 지원 |
| 성능 | 위빙 방식에 따라 성능 좋을 수 있음 | 프록시 기반이므로 약간의 오버헤드 |
| 설정 난이도 | ajc 컴파일, LTW 필요 → 복잡함 | 스프링 프로젝트에 자연스럽게 통합 가능 |

### 1. Aspect
- 여러 클래스에 걸쳐 나타나는 공통 관심사(crosscutting concern)를 모듈화한 단위
- Spring AOP에서는 일반 클래스 또는 @Aspect 애노테이션이 붙은 클래스로 Aspect를 구현한다.

### 2. Join Point
- 프로그램 실행 중의 특정 지점을 뜻한다. (ex. 메서드 실행, 예외 처리 등)
- Spring AOP에서는 오직 '메서드 실행'만 조인 포인트로 지원한다. 즉, 생성자 호출, 필드 접근 등 기존 AspectJ 에서 지원했던 것들을 지원하지 않는다.

### 3. Advice
- Aspect가 특정 Join Point에서 수행하는 작업이다.
- 종류 (아래 Advice 종류에 좀 더 구체적으로 설명)
    `@Before`: 메서드 실행 전  
    `@After`: 메서드 실행 후  
    `@Around`: 실행 전후를 모두 감싸기
- Spring을 포함한 많은 AOP 프레임워크는 Advice를 인터셉터(interceptor)로 모델링하고, Join Point를 중심으로 인터셉터 체인(chain)을 구성하여 실행한다.

### 4. Pointcut
- Join Point를 선별하기 위한 조건이다.
- 어떤 Join Point에 Advice를 적용할지를 정의한다.
    ex. `execution(* com.example.service..*(..))` → com.example.service 패키지의 모든 메서드에 적용
- Spring AOP는 기본적으로 AspectJ의 포인트컷 표현식을 사용한다.

### 5. Introduction (도입)
- 기존 클래스에 새로운 필드나 메서드를 추가하는 선언이다.
- ex. 어떤 빈에 IsModified 인터페이스를 도입해서 캐시 처리를 단순화하는 방식
- Spring AOP는 프록시를 통해 기존 객체에 새로운 인터페이스와 구현을 주입할 수 있다.

### 6. Target Object
- Aspect에 의해 조언(Advice)을 받는 실제 객체를 의미한다.
- Spring AOP는 런타임 프록시 기반으로 동작하므로, 대상 객체는 항상 프록시 객체에 감싸진 상태이다.

### 7. AOP Proxy
- Advice를 적용하기 위해 AOP 프레임워크가 생성한 프록시 객체이다.
- Spring AOP 에서는 두 가지 방식이 있다.
    - JDK Dynamic Proxy (인터페이스 기반)
    - CGLIB Proxy (클래스 상속 기반)

### 8. Weaving (위빙)
- Aspect를 실제 코드에 연결하는 과정이다.
- 기본적으로 위빙 시점에는 3가지가 (컴파일 타임, 클래스 로딩 시, 런타임) 있는데, (AspectJ 프레임워크 기준) Spring AOP는 순수 Java 기반의 런타임 프록시 방식으로 위빙을 수행한다.

> Join Point + Pointcut의 개념은 AOP의 핵심이다.  
> AOP는 단순한 인터셉터보다 더 정밀하게 동작하며, Pointcut은 조인 포인트를 객체지향 구조와 무관하게 타겟팅할 수 있다.


---

## Advice 종류

### 1. Before Advice (실행 전 어드바이스)
- 조인 포인트가 실행되기 전에 실행되는 Advice이다.
- 이 Advice는 조인 포인트의 실행을 막을 수는 없지만, 예외를 던지면 실행을 중단시킬 수 있다.
- ex. 메서드 실행 전에 로그를 남기거나 권한을 확인하는 로직에 적합

### 2. After Returning Advice (정상 반환 후 어드바이스)
- 조인 포인트가 정상적으로 종료(예외 없이 반환)된 후에 실행된다.
- ex. 메서드 결과를 캐시에 저장하거나, 성공 로그 기록 등에 적합

### 3. After Throwing Advice (예외 발생 후 어드바이스)
- 조인 포인트가 예외를 던지며 종료될 경우 실행되는 Advice이다.
- ex. 예외 내역 로깅, 에러 알림, 모니터링용으로 사용

### 4. After (Finally) Advice (항상 실행되는 어드바이스)
- 조인 포인트가 정상 종료든 예외 종료든 무조건 실행된다.
- Java의 finally 블록처럼 동작한다.
- ex. 리소스 해제, 트랜잭션 정리, 공통 후처리 로직 등에 적합

### 5. Around Advice (둘러싼 어드바이스)
- 조인 포인트 전체를 감싸는 가장 강력한 Advice이다.
- 메서드 실행 전과 후 모두에서 커스텀 로직을 실행할 수 있고, 조인 포인트의 실행을 수행할지 말지 결정할 수 있다.
- 즉, 직접 결과를 반환하거나 예외를 던져서 실행 자체를 대체할 수도 있다.
- ex. 트랜잭션 관리, 퍼포먼스 측정, 완전한 실행 흐름 제어 등에 사용

> 🌟 Spring Docs: “가장 약하고 구체적인 Advice부터 사용하라”
> - Around Advice는 강력하지만 실수할 여지가 많다.
> - 예를 들어, proceed() 호출을 빼먹으면 조인 포인트가 실행되지 않는다.
> - 그래서 가능하면 필요한 기능을 수행할 수 있는 가장 구체적이고 약한 타입의 Advice를 쓰는 게 좋다.
> - ex. 리턴값을 기반으로 캐시만 갱신하고 싶다면 → After Returning을 사용하면 된다. (굳이 Around를 쓸 필요 x)

### Advice의 파라미터는 정적으로 타입 지정
- Advice 메서드의 파라미터들은 적절한 타입으로 타입이 명시되어 있다.
- ex. `@AfterReturning` 에서는 메서드 리턴값을 그대로 `String`, `User`, `Long` 등으로 받을 수 있어서 `Object` 배열을 파싱하거나 캐스팅할 필요가 없음 (안전하고 편리)

---

## (최종적으로) Spring AOP의 기능과 목표
- **IoC 컨테이너와의 통합** :AOP 구현과 Spring IoC를 긴밀하게 통합하여 엔터프라이즈 애플리케이션에서 발생하는 문제 해결에 도움을 주고자 한다.

- Spring AOP는 Bean에 프록시를 씌움으로서 컨테이너 관리 하에 비침투적으로 AOP를 가능하게 한다.
    - ex. `@Component`, `@Service` 등으로 등록된 Bean이 IoC에 등록될 때 AOP가 적용된 Bean이라면, 실제 Bean이 아니라 Proxy Bean이 IoC에 들어가는 형태 

- AspectJ처럼 강력하고 다양한 기능을 제공하는 것은 아니지만, 웹 개발엔 충분한 수준의 JoinPoint 제어를 제공한다. (AspectJ와 경쟁하는 것이 아니라 상호 보완하며 필요한 기능을 사용하자는 취지)

AspectJ | Spring AOP
순수 AOP 중심 | AOP + IoC 통합 중심
강력한 위빙/범용 AOP | Spring 개발에 최적화된 비침투적 AOP
트랜잭션, DI 등 직접 구현 필요 | 스프링 기능과 결합된 AOP 구성 가능


---
ref.
- [docs: AOP Concepts](https://docs.spring.io/spring-framework/reference/core/aop/introduction-defn.html)
- [docs: Spring AOP Capabilities and Goals](https://docs.spring.io/spring-framework/reference/core/aop/introduction-spring-defn.html)

