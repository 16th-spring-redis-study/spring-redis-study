# 1. Spring AOP 동작 방식
## 주요 개념
- Aspect: 공통 관심사(Advice + Pointcut 조합)
- Advice: 실행 시점에 따라 실행되는 코드 (`@Before`, `@After`, `@Around` 등)
- Join Point: Advice가 적용될 수 있는 지점 (메서드 실행 등)
- Pointcut: JoinPoint 중 실제로 Advice가 적용될 조건을 명시
- Weaving: Advice를 실제 코드에 적용하는 과정 (Spring은 런타임에 프록시로 위빙)


# 2. Proxying Mechanisms


# 3. Spring AOP 방식 별 주의 사항


---
ref.
- [docs: Proxying Mechanisms]()
- [docs: AspectJ Proxies]()
- []()