# Spring IoC Container

## Spring IoC Container란?
우리가 흔히 부르는 Spring Container이다. Spring Container는 빈을 인스턴스화시키고, 이를 
조립하고 구성하는 역할을 담당한다. 여기서 이를 조립 및 구성하는 방법은 구성 요소의 메타데이터를 읽어와서
수행하게 된다.
- @Component, 외부 XML 파일, Groovy 스크립트, 팩토리 메서드로 이루어진 설정 파일 등등

이를 통해 상호 의존 관계를 설정함.

*IoC
제어의 역전이란 뜻으로 애플리케이션의 제어 흐름을 개발자가 아닌 프레임워크가 수행한다는 뜻이다. 
=> 객체의 생성과 의존성 주입을 Spring Container가 수행함. 이때 컨테이너가 프록시로 감싸서 만드는 객체를 빈이라고 하는 거임.

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

#### @Autowired

Spring Framework 4.3부터는 생성자가 하나만 존재할 경우에는 @Autowired가 필요하지 않지만, 여러 개의 생성자가 존재할 시
반드시 어떤 생성자를 사용할지 지정해주기 위해 하나의 생성자에는 @Autowired를 추가해야 한다.


### Java-based configuration

ComponentScan 방식과는 달리 사용자가 명시적으로 빈을 등록하여 관리하는 방식이다.

- @Configuration: 애노테이션을 통해 해당 클래스가 빈 정의를 제공하는 역할을 한다는 것을 나타낸다. 
- @Bean: 애노테이션을 통해 새로운 객체를 생성, 설정, 초기화하여 Spring IoC 컨테이너에서 관리하도록 사용(@Component와도 사용이 가능)

Spring IoC 컨테이너는 @Bean이 붙은 친구들을 내부에서 프록시로 감싸서 새 객체를 생성하지 않고 항상 같은 인스턴스를 반환하게 한다.

~~~java
@Configuration
public class AppConfig {

	@Bean
	public MyServiceImpl myService() {
		return new MyServiceImpl();
	}
}
~~~

### @Configuration 클래스에서의 @Bean 메서드 간 호출

일반적인 상황에서는 @Configuration 안에 @Bean 메서드들을 등록하여 관리한다. => full configuration class processing으로
적용되어 생명 주기를 Spring이 관리

하지만 다음과 같은 상황에서는 lite mode가 적용된다.
- @Configuration없이 @Bean만 사용
- @Configuration(proxyBeanMethods = false)

라이트 모드에서는 컨테이너가 개입하지 않아 프록시로 감싸지 않고 일반 자바 호출로 판단하여, 매번 새로운 객체를 만들어낸다.