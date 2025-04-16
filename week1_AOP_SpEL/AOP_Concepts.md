# 1. Spring AOP Concepts
## AOP
- AOP는 OOP를 보완한다. 클래스가 핵심 단위인 OOP와 달리, Aspect를 핵심 단위로 둔다.
- Aspect는 트랜잭션 관리와 같이, 여러 클래스에 걸쳐 반복되는 **공통 기능(트랜잭션, 로깅, 보안, 캐싱 등)** - 횡단 관심사를 모듈화할 수 있게 만든다.
- 횡단 관심사가 무엇인지 보충 설명을 하면, 위 공통 기능들은 모든 서비스에서 공통적으로 필요하나, OOP 관점에서 볼 때 각 클래스의 주 목적은 아니다. (cross-cutting 하듯이 여러 클래스나 메서드에 공통으로 들어가는 기능이라는 의미)

## AOP는 어떤 문제를 해결하기 위해 등장했을까?
- OOP(객체지향 프로그래밍)는 **공통 관심사(cross-cutting concern)**를 모듈화하기 어렵다는 한계가 있다.

- 모든 서비스에 로그, 트랜잭션, 권한 검사 등 공통적이고 반복되는 코드를 관리하고 유지보수가 어려워진다. 특히, 핵심 비즈니스 로직과 공통 코드가 섞이면 좋지 않다.

    ➡ 반복적이고 횡단적인 관심사를 Aspect로 분리해 코드 중복을 제거하면 응집도 높은 비즈니스 로직을 유지할 수 있다.

## Aspect는 무엇을 포함하나?
- Advice: 언제, 무엇을 실행할지 정의, 실제로 실행할 공통 로직 (`@Before`, `@After`, `@Around` 등)
- Pointcut: Advice가 적용될 위치를 조건으로 표시 (표현식 기반, `execution(* com.example..*Service.*(..))`)


# 2. Spring AOP 구현 방식
- Spring AOP는 프록시 기반의 AOP를 사용한다. (Proxy-based)
- 기본적으로 런타임 프록시를 기반으로 한다. (Not Complie-Time!)


## 세부 구현 방식
1.
2. 


---
ref.
- [docs: AOP Concepts]()
- [docs: Spring AOP Capabilities and Goals]()