# Spring IoC Container

## Spring IoC Container란?
우리가 흔히 부르는 Spring Container이다. Spring Container는 빈을 인스턴스화시키고, 이를 
조립하고 구성하는 역할을 담당한다. 여기서 이를 조립 및 구성하는 방법은 구성 요소의 메타데이터를 읽어와서
수행하게 된다.
- @Component, 외부 XML 파일, Groovy 스크립트, 팩토리 메서드로 이루어진 설정 파일 등등

이를 통해 상호 의존 관계를 설정함.

***

## ApplicationContext

ApplicationContext는 상위 인터페이스인 BeanFactory의 모든 기능을 포함하며, 이를 더 확장한 인터페이스이다.
<br>

**ApplicationContext**는 Spring 컨테이너를 추상화한 인터페이스이고, 보통 
**AnnotationConfigApplicationContext** 또는 **ClassPathXmlApplicationContext**의 구현체를 사용한다고 한다.
<br>=> Spring Boot가 이를 알아서 ApplicationContext를 자동으로 초기화 시켜줘서 사용자가 따로 구현할 필요는 없다.

각각의 인터페이스가 제공하는 기능들을 보면

### BeanFactory

- 빈 등록, 생성, 조회, 관리
  - getBean()을 통해 빈 인스턴스를 가져올 수 있지만, 공식 문서에서는 애플리케이션 코드에서는 절대
  사용하지 말라고 되어 있음. 
  <br> => Spring API에 대한 의존성 제거
- 설정 파일을 통해 빈 정의
- 기본적으로 모든 빈의 지연 로딩

### ApplicationContext

- BeanFactory에서 제공하는 기능
- **AOP 지원을 통한 애너테이션 기반으로 빈 정의 가능**
- **즉시 로딩**
- 국제화, 환경설정 및 프로파일 관리(@Profile)등등

***

## Metadata 설정

![컨테이너 흐름](../week3_Spring_Container_DI/img/container_flow.png)

Spring Container는 설정 메타데이터를 통해 애플리케이션의 구성 요소를 설정하고 관리
할지 결정한다.

### Annotation-based configuration

우리가 흔히 쓰는 방식으로 클래스에 애너테이션을 추가하여 빈을 정의하고 의존성을 주입하는
방법이다. ComponentScan을 통해 빈을 자동으로 탐지하고 등록한다. 

```java
@Service
public class MyService {
    @Autowired
    private MyRepository repository;
    
    // ...
}

@Repository
public class MyRepository {
    // ...
}
```

### Java-based configuration

ComponentScan 방식과는 달리 사용자가 명시적으로 빈을 등록하여 관리하는 방식이다.

https://docs.spring.io/spring-framework/reference/core/beans/basics.html

https://sundaland.tistory.com/474