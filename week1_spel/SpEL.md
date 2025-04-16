# Spring Expression Language (SpEL)

SpEL은 Spring 프레임워크 내에서 런타임 시 객체의 프로퍼티, 메서드, 컬렉션 등 다양한 값을 동적으로 평가·조작할 수 있도록 지원하는 표현식 언어이다.

- 실행 시점에 평가함으로써, 설정 파일이나 애노테이션에 기록된 표현식을 **동적으로** 수정할 수 있다.
- 설정 파일 또는 애노테이션 내에서 복잡한 로직을 표현식으로 구현함으로써 **비즈니스 로직과 설정 정보를 분리**할 수 있다.

이를 통해 스프링 설정(XML, 애노테이션) 및 자바 코드 내에서 정적 값이 아닌 런타임 결정 값, 동적 로직을 쉽게 구현할 수 있다.

<br/>

## 1. Evaluation

SpEL은 표현식을 평가하는 과정은 주로 2가지 구성 요소에 의해 이루어진다. <br/>
먼저 **표현식 파서를 통해 문자열을 파싱**한 후, **EvaluationContext**에 설정된 정보를 기준으로 실제 값을 산출하는 구조로 이루어진다.

### 1.1 표현식 파서 (SpelExpressionParser)

표현식 파서는 문자열 형태의 SpEL 표현식을 읽어 내부적으로 추상 구문 트리(AST)를 생성하고, 이를 기반으로 재사용 가능한 `Expression` 객체로 변환한다.

```java
  ExpressionParser parser = new SpelExpressionParser();
  Expression expr = parser.parseExpression("10 + 20");
  int result = expr.getValue(Integer.class);  // 결과: 30
```

### 1.2 EvaluationContext (StandardEvaluationContext)

EvaluationContext는 SpEL 표현식을 평가할 때 필요한 모든 환경 정보를 제공한다. 주로 **StandardEvaluationContext**를 사용하며, 이 컨텍스트는 다음과 같은 기능을 포함한다.

- **루트 객체 설정:**  
  EvaluationContext에 지정된 루트 객체는 표현식에서 별도의 대상이 명시되지 않은 경우 평가의 기본 기준이 된다.  

- **변수 등록:**  
  `setVariable("변수명", 값)`을 통해 추가 데이터를 등록한 후, 표현식 내에서 `#변수명`으로 참조할 수 있다.

- **함수 등록:**  
  EvaluationContext에 정적 메서드를 등록하면, 표현식 내에서 `#함수명(인자)` 형식으로 호출할 수 있다.

- **타입 변환 지원:**  
  EvaluationContext는 내부 ConversionService를 사용하여 평가 결과를 필요한 타입으로 자동 변환해준다.

- **커스텀 확장:**  
  필요에 따라 PropertyAccessor, MethodResolver, ConstructorResolver 등을 추가하거나 교체하여 평가 동작을 세밀하게 제어할 수 있다.

```java
  // 도메인 클래스
  public class Person {
      private String name;
      private int age;

      public Person(String name, int age) {
          this.name = name;
          this.age = age;
      }

      public String getName() { return name; }
      public void setName(String name) { this.name = name; }
      public int getAge() { return age; }
      public void setAge(int age) { this.age = age; }
  }

  // 사용자 정의 함수: 문자열을 뒤집는 함수
  public class MySpelFunctions {
      public static String reverse(String input) {
          return new StringBuilder(input).reverse().toString();
      }
  }

  // 평가 및 EvaluationContext 활용 예제
  public class SpelEvaluationExample {
      public static void main(String[] args) throws Exception {
          // 1. 루트 객체 생성
          Person person = new Person("Alice", 30);

          // 2. ExpressionParser를 사용하여 표현식 파싱
          ExpressionParser parser = new SpelExpressionParser();
          Expression nameExpr = parser.parseExpression("name");
          Expression ageExpr = parser.parseExpression("age");

          // 3. StandardEvaluationContext 생성 및 루트 객체 지정
          StandardEvaluationContext context = new StandardEvaluationContext(person);

          // 4. 변수 등록: #defaultName 변수 등록 (기본값 "Unknown")
          context.setVariable("defaultName", "Unknown");

          // 5. 사용자 정의 함수 등록: MySpelFunctions의 reverse() 메서드 등록
          context.registerFunction("reverse",
              MySpelFunctions.class.getDeclaredMethod("reverse", String.class));

          // 6. 표현식 평가
          String name = nameExpr.getValue(context, String.class);    // "Alice" 반환
          int age = ageExpr.getValue(context, Integer.class);          // 30 반환

          // 7. 등록된 변수와 함수 사용
          String varValue = parser.parseExpression("#defaultName")
                                  .getValue(context, String.class);  // "Unknown"
          String reversedName = parser.parseExpression("#reverse(name)")
                                      .getValue(context, String.class);  // "ecilA" 반환

          // 출력
          System.out.println("Name: " + name);              // Name: Alice
          System.out.println("Age: " + age);                // Age: 30
          System.out.println("Default Name: " + varValue);  // Default Name: Unknown
          System.out.println("Reversed Name: " + reversedName);  // Reversed Name: ecilA
      }
  }
```

<br/>

## 2. Spring과의 통합

Spring 프레임워크는 SpEL을 빈 정의, 의존성 주입, 환경 설정 등 다양한 영역에 통합하여 활용할 수 있도록 지원한다.

### 2.1. @Value 애노테이션에서의 활용

- **동적 값 평가:**  
  `@Value` 애노테이션 내부에 `#{...}` 형식의 SpEL 표현식을 사용하면, 빈 생성 시점에 해당 표현식을 평가하여 그 결과를 주입할 수 있다.
- **빈 간의 참조:**  
  표현식 내에서 다른 빈의 이름과 프로퍼티에 접근할 수 있어, 빈 간 의존성을 자연스럽게 연결할 수 있다.
- **시스템 및 환경 변수 사용:**  
  `systemProperties`와 `systemEnvironment`를 이용해 자바 시스템 속성이나 OS 환경 변수의 값을 주입할 수 있다.
- **기본값 및 조건부 값:**  
  Elvis 연산자(`?:`)를 사용하면, 평가 결과가 null일 때 기본값을 제공할 수 있다.

```java
@Component
public class MyBean {

    // 단순 산술 및 문자열 결합 표현식
    @Value("#{10 + 20}")
    private int sum;                    // 결과: 30

    @Value("#{'Hello ' + 'SpEL'}")
    private String greeting;            // 결과: "Hello SpEL"

    // 다른 빈의 프로퍼티 참조
    @Value("#{otherBean.name}")
    private String otherName;

    // 자바 시스템 프로퍼티 접근
    @Value("#{systemProperties['user.home']}")
    private String userHomeDir;

    // 값이 null인 경우 기본값 'defaultValue'를 사용 (Elvis 연산자 활용)
    @Value("#{someBean.someProperty ?: 'defaultValue'}")
    private String value;
}
```

### 2.2. XML 기반 빈 정의에서의 활용

Spring XML 설정 파일에서 `<property>` 태그의 value 속성에 SpEL 표현식을 작성하면, 해당 값이 표현식 평가 결과로 할당된다.

- **빈의 동적 참조**:
  XML 설정에서 다른 빈을 참조할 때 표현식 문법 `#{...}`을 사용한다.
- **조건 및 계산**:
  XML 내에서도 SpEL을 이용해 조건부 로직이나 복잡한 계산을 수행할 수 있다.

```xml
  <bean id="engine" class="com.example.Engine">
    <property name="capacity" value="3200"/>
    <property name="horsePower" value="250"/>
    <property name="numberOfCylinders" value="6"/>
  </bean>

  <bean id="car" class="com.example.Car">
      <property name="make" value="SomeMake"/>
      <property name="model" value="SomeModel"/>
      <!-- engine 빈 참조, horsePower 값을 동적으로 주입 -->
      <property name="engine" value="#{engine}"/>
      <property name="horsePower" value="#{engine.horsePower}"/>
  </bean>
```

### 2.3. 프로퍼티 플레이스홀더와 SpEL의 혼용

Spring은 PropertyPlaceholderConfigurer 또는 @PropertySource를 통해 외부 프로퍼티 파일의 값을 불러올 수 있다.

- **혼용 가능성**:
  @Value에서 `${property.name}`은 외부 프로퍼티 값을 대체한다.

```java
  // 외부 프로퍼티 파일에 some.number=3 으로 정의되어 있을 경우
  @Value("#{${some.number} + 2}")
  private int computedNumber;  // 결과: 5 (프로퍼티 값 3 + 2)
```

<br/>

## 3. 컬랙션 및 객체 그래프 접근

SpEL은 단순히 개별 객체의 프로퍼티만이 아니라, 복잡하게 중첩된 객체 그래프와 컬렉션을 탐색하는 기능도 제공한다.

### 3.1. 객체 그래프 접근

- **속성 체이닝:**  
  객체에 포함된 다른 객체의 프로퍼티에 접근할 때는 마침표(`.`) 표기를 사용한다.
- **안전한 내비게이션 연산자:**  
  중첩 객체 중 어느 하나라도 `null`인 경우, 일반적인 접근 방식은 `NullPointerException`을 발생시킬 수 있다. 이 문제를 해결하기 위해 SpEL은 `안전한 내비게이션 연산자(?.)`를 제공한다.

### 3.2. 리스트, 배열 및 맵 접근

- 리스트나 배열의 개별 요소는 대괄호(`[ ]`)를 이용해 인덱스 기반으로 접근할 수 있다.
- 맵의 경우, 키를 대괄호(`['key']`)로 감싸서 접근힌다.

### 3.3. 컬렉션 필터링과 프로젝션 (선택 및 변환)

SpEL은 컬렉션에 대해 보다 정교한 처리가 가능하도록 다음과 같은 기능을 제공한다.

- **선택(selection) 연산자**:
  특정 조건에 맞는 요소들만 필터링할 수 있습니다. 선택 연산자는 [`?(조건)`] 문법을 사용한다.
- **프로젝션(projection) 연산자**:
  컬렉션의 각 원소에 동일한 변환을 적용해 새로운 컬렉션으로 반환할 수 있다.
  프로젝션 연산자는 `![]` 문법을 사용한다.

  ```java
  // 리스트 변수 등록 예제
  List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
  context.setVariable("names", names);

  // 선택: 이름 길이가 4 이상인 요소들만 선택
  List<?> filteredNames = parser.parseExpression("#names.?[#this.length() >= 4]")
                                  .getValue(context, List.class);
  // filteredNames: ["Alice", "Charlie"]

  // 프로젝션: 모든 이름을 대문자로 변환
  List<?> projectedNames = parser.parseExpression("#names.![toUpperCase()]")
                                .getValue(context, List.class);
  // projectedNames: ["ALICE", "BOB", "CHARLIE"]
  ```
