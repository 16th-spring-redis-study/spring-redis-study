## 1. 스프링의 트랜잭션 관리

### 1-1. 트랜잭션 종류

1. **로컬 트랜잭션**
하나의 데이터베이스에만 적용되는 트랜잭션이다. JDBC를 통해 DB에 직접 연결해서 처리할 때 주로 사용된다.
2. **글로벌 트랜잭션**
둘 이상의 자원에 동시에 걸쳐 있는 트랜잭션이다. 이때 자원은 데이터베이스나 메세지 큐 등을 의미한다. 모든 자원이 함께 성공하거나 함께 실패해야 하므로 보다 복잡한 트랜잭션 관리가 필요하다.

### 1-2. 장점

1. **일관된 프로그래밍 모델 제공**
스프링은 JDBC, JPA, Hibernate, JTA 등 다양한 트랜잭션 기술을 하나의 공통된 방식으로 다룰 수 있도록 지원한다. 기술마다 코드를 따로 작성할 필요 없이, 하나의 코드로 트랜잭션을 관리할 수 있다.
2. **선언적 트랜잭션 관리 지원**
`@Transactional` 애너테이션만으로 트랜잭션을 자동으로 시작하고 커밋하거나 롤백할 수 있다. 설정만으로 제어가 가능해 코드가 간결하고 유지보수가 쉽다.
3. **프로그래밍 방식 지원**
필요할 경우 트랜잭션을 직접 시작하고 커밋하거나 롤백하는 방식도 가능하다. 선언적 트랜잭션 관리보다 유연한 제어가 가능하다.
4. **애플리케이션 서버 없이 사용 가능**
글로벌 트랜잭션이 필요한 상황이 아니라면 별도의 애플리케이션 서버 없이도 대부분의 트랜잭션 처리가 가능하다.
5. **유연한 트랜잭션 전략 전환**
코드 수정 없이 설정 변경만으로 로컬 트랜잭션에서 글로벌 트랜잭션으로 전환할 수 있다.

## 2. 트랜잭션 추상화

### 2-1. 인터페이스

스프링에서는 트랜잭션 관리를 하나의 전략으로 보고 있다. 이 전략은 상황에 따라 유연하게 바뀔 수 있으며, 이를 담당하는 것이 바로 `TransactionManager` 인터페이스이다. 스프링은 이 추상화를 통해 다양한 트랜잭션 기술을 하나의 통일된 구조로 제어할 수 있도록 지원한다. `getTransaction()`을 호출하면 트랜잭션을 시작하고, `commit()` 또는 `rollback()`으로 트랜잭션을 종료할 수 있다. 또한 인터페이스는 환경에 따라 다음 2가지로 나눌 수 있다.

1. **`PlatformTransactionManager`**
    
    전통적인 명령형 방식의 Java 애플리케이션에서 사용되며, JDBC, JPA, Hibernate 등 블로킹 기반의 동기 처리 환경에서 트랜잭션을 관리할 때 사용한다.
    
    ```java
    public interface PlatformTransactionManager extends TransactionManager {
    
    	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
    
    	void commit(TransactionStatus status) throws TransactionException;
    
    	void rollback(TransactionStatus status) throws TransactionException;
    }
    ```
    
2. **`ReactiveTransactionManager`**
    
    WebFlux와 같이 비동기 스트림을 사용하는 리액티브 프로그래밍 환경에서 트랜잭션을 관리할 때 사용하며, `Mono`, `Flux` 같은 리액티브 타입을 기반으로 동작한다.
    
    ```java
    public interface ReactiveTransactionManager extends TransactionManager {
    
    	Mono<ReactiveTransaction> getReactiveTransaction(TransactionDefinition definition) throws TransactionException;
    
    	Mono<Void> commit(ReactiveTransaction status) throws TransactionException;
    
    	Mono<Void> rollback(ReactiveTransaction status) throws TransactionException;
    }
    ```
    

### 2-2. 속성

트랜잭션을 정의할 때는 다음과 같은 속성들을 설정할 수 있다.

1. **전파 (Propagation)**
    
    트랜잭션이 이미 존재할 때 기존 트랜잭션에 참여할지, 별도로 새 트랜잭션을 시작할지와 같은 트랜잭션 간 관계를 설정한다.
    
2. **격리수준 (Isolation)**
    
    이 트랜잭션이 다른 트랜잭션의 작업으로부터 어느 정도까지 보호되어야 하는지 설정한다.
    
3. **제한 시간 (Timeout)**
    
    트랜잭션이 유지될 수 있는 제한 시간을 설정한다.
    
4. **읽기 전용 (ReadOnly)**
    
    트랜잭션이 데이터를 읽기만 하는 경우 성능 최적화를 위해 사용한다.
    

### 2-3. 상태 관리

트랜잭션이 실제로 어떻게 작동 중인지 확인하거나 제어하기 위해 `TransactionStatus`라는 객체를 사용한다. 이 객체를 통해 현재 트랜잭션이 새로 시작된 것인지, 롤백 상태인지, 완료되었는지 등을 판단할 수 있다. 또한 수동으로 `setRollbackOnly()`를 호출해서 강제로 롤백 지시를 내릴 수도 있다.

```java
public interface TransactionStatus extends TransactionExecution, SavepointManager, Flushable {

	@Override
	boolean isNewTransaction();

	boolean hasSavepoint();

	@Override
	void setRollbackOnly();

	@Override
	boolean isRollbackOnly();

	void flush();

	@Override
	boolean isCompleted();
}
```

### 2-4. 구현체

스프링에서 트랜잭션 매니저를 설정하려면 어떤 기술을 사용하는지에 따라 적절한 구현체를 지정해줘야 한다. 대표적인 구현체는 다음과 같다.

1. **`DataSourceTransactionManager`**
    
    JDBC처럼 하나의 데이터베이스에 직접 접근하는 로컬 트랜잭션 환경에서 사용한다.
    
2. **`HibernateTransactionManager`**
    
    Hibernate 세션을 사용하는 경우에 적합하며, `SessionFactory`와 연동된다.
    
3. **`JtaTransactionManager`**
    
    JTA 기반의 환경에서 둘 이상의 자원에 걸친 글로벌 트랜잭션이 필요한 경우에 사용한다.
    

그리고 트랜잭션 매니저가 사용할 데이터 소스는 보통 `DataSource` 빈으로 등록하여 의존성 주입을 통해 연결해준다. 이처럼 인터페이스-구현체 방식을 통해 트랜잭션 구현체를 바꾸더라도 애플리케이션 코드는 수정할 필요가 없도록 해준다.

## 3. 리소스 동기화

트랜잭션 매니저는 내부적으로 DB 커넥션이나 세션 같은 리소스들과 연결되어 있다. 스프링에서는 고수준과 저수준 두 가지 방식으로 트랜잭션과 리소스를 동기화할 수 있다.

### 3-1. 고수준 방식

가장 일반적이며 권장되는 방식으로, `JdbcTemplate`, `JpaTemplate` 같은 스프링의 템플릿 기반 API나 Hibernate의 `SessionFactory`처럼 트랜잭션을 인식하는 자원 팩토리를 사용하는 방식이다. 이 방식을 사용하면 아래와 같은 작업을 스프링이 자동으로 처리해주기 때문에, 개발자는 복잡한 트랜잭션 관리 코드를 직접 작성하지 않고 비즈니스 로직에만 집중할 수 있다.

- 커넥션이나 세션의 생성 및 재사용
- 트랜잭션과의 자동 연결 및 동기화
- 트랜잭션 종료 후 자원 정리
- 발생한 예외를 스프링 예외로 변환하여 처리

### 3-2. 저수준 방식

JDBC나 JPA의 API를 직접 사용하려면 `DataSourceUtils`, `EntityManagerFactoryUtils`, `SessionFactoryUtils` 같은 스프링 유틸리티 클래스를 사용하면 된다.

예를 들어 일반 JDBC에서는 보통 다음과 같이 커넥션을 얻는다.

```java
Connection conn = dataSource.getConnection();
```

하지만 스프링에서는 다음과 같이 호출하는 것이 안전하다.

```java
Connection conn = DataSourceUtils.getConnection(dataSource);
```

이렇게 하면 현재 트랜잭션이 존재할 경우 해당 트랜잭션에 이미 묶여 있는 커넥션을 재사용하고, 없으면 새 커넥션을 만들어 자동으로 트랜잭션에 연결해준다. 그리고 `SQLException`이 발생할 경우 스프링의 `DataAccessException` 하위 클래스로 래핑해주기 때문에, 데이터베이스 종류와 상관없이 일관된 예외 처리가 가능하다.

저수준 방식은 스프링 트랜잭션을 사용하지 않는 경우에도 사용할 수 있어서 범용성이 넓다. 하지만 대부분의 경우에는 `JdbcTemplate` 같은 고수준 방식을 사용하는 것이 훨씬 편리하다.

### 3-3. `TransactionAwareDataSourceProxy`

이 클래스는 외부 라이브러리나 기존 코드에서 오직 `DataSource` 인터페이스만 요구하는 경우에 사용된다. 이런 경우에는 스프링의 `DataSourceUtils` 같은 유틸리티 클래스를 직접 사용할 수 없기 때문에 이 프록시 클래스로 원본 `DataSource`를 감싼다. 이를 통해 트랜잭션이 이미 존재하면 해당 커넥션을 재사용하고, 트랜잭션 종료 시점에 맞춰 커밋이나 롤백이 함께 처리되도록 도와준다.

하지만 새로운 코드를 작성할 때는 일반적으로 `JdbcTemplate` 같은 고수준 API나 유틸리티 클래스를 사용하므로 이 클래스를 직접 사용할 일은 거의 없다.