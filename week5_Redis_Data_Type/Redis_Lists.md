# Redis Lists

Redis 리스트는 문자열 값들로 **연결된 리스트**이고, 스택, 큐 구현, 백그라운드 워커 시스템에서
작업 대기열을 관리할 때 주로 사용된다.

## Basic Commands

- **LPUSH**: 리스트의 시작에 항목 삽입
- **RPUSH**: 리스트의 마지막에 항목 삽입
- **LLEN**: 리스트의 길이 반환
- **LMOVE**: 한 리스트에서 다른 리스트로 원자적으로 항목 이동
- **LRANGE**: 범위안의 요소들을 리스트에서 반환
- **LTRIM**: 리스트를 지정된 범위의 요소들로 잘라서 줄여줌

*리스트에 항목이 없을 경우 null을 반환

### LPUSH, RPUSH

모두 가변인자 명령어로 한 번의 호출로 여러 요소를 집어 넣을 수 있다.

~~~redis
> RPUSH bikes:repairs bike:1 bike:2 bike:3
(integer) 3
> LPUSH bikes:repairs bike:important_bike bike:very_important_bike
> LRANGE bikes: bikes:repairs 0 -1
1) "bike:very_important_bike"
2) "bike:important_bike"
3) "bike:1"
4) "bike:2"
5) "bike:3"
~~~

## Blocking Commands

- **BLPOP**: 리스트의 첫 번째 요소 반환 (리스트가 비어 있을 경우, 요소가 추가될 때까지 대기하거나
정해진 Timeout까지 대기)
- **BLMOVE**: **LMOVE**와 같은 동작 (소스 리스트가 비어있을 경우 새로운 항목이 추가될 때까지 대기)

~~~redis
> RPUSH bikes:repairs bike:1 bike:2 bike:3 bike:4 bike:5
(integer) 5
> LTRIM bikes:repairs 0 2
OK
> LRANGE bikes:repairs 0 -1
1) "bike:1"
2) "bike:2"
3) "bike:3"
~~~

## Redis에서의 List

Redis에서의 리스트는 배열이 아닌 **연결 리스트**로 구현되어 있다. <br>
=> 리스트 안에 수백만 개의 항목이 있더라도, **LPUSH**, **RPUSH** 등 리스트의 시작과 마지막 요소에 대한
작업에 대해서는 상수 시간복잡도 O(1)에 처리된다. 리스트의 길이를 일정한 시간으로 가져올 수 있다.

단점으로는 인덱스를 통해 요소에 접근할 때 리스트의 처음부터 순차적으로 탐색을 해야 하기 때문에 작업량이 접근하는
요소의 인덱스에 비례한다.
- Redis 측에서는 많은 항목이 들어있는 컬렉션의 중간 항목에 접근할 때는 **Sorted Set**을 대체제로 권하고 있다.


## Capped Lists

Redis는 리스트를 고정된 크기의 컬렉션처럼 사용할 수 있게 해준다. **LTRIM** 명령어를 통해 최신의
N개 항목들만 기억하고 오래된 데이터를 제거할 수 있다.

**LRANGE**하고 비슷해 보일 수 있지만, 범위 안의 항목들을 보여주는 것이 아닌 새로운 리스트 값들로 바꾼다.
이때 범위 밖 항목들은 모두 제거한다.

~~~redis
> RPUSH bikes:repairs bike:1 bike:2 bike:3 bike:4 bike:5
(integer) 5
> LTRIM bikes:repairs 0 2
OK
> LRANGE bikes:repairs 0 -1
1) "bike:1"
2) "bike:2"
3) "bike:3"
~~~

**LTRIM 0 2**을 통해 0~2까지의 항목들만 저장하고 나머지는 버린다.

~~~redis
> RPUSH bikes:repairs bike:1 bike:2 bike:3 bike:4 bike:5
(integer) 5
> LTRIM bikes:repairs -3 -1
OK
> LRANGE bikes:repairs 0 -1
1) "bike:3"
2) "bike:4"
3) "bike:5"
~~~

위의 예시와 최신 3개만 유지하게 **RPUSH**와 **LRANGE**로 조합해서 사용할 수 있다.

**LRANGE** 명령어는 O(N)의 시간복잡도를 가진다.

## Blocking operations on Lists

Redis 리스트는 블로킹 연산을 제공한다.

프로세스간 통신을 생산자-소비자 패턴으로 예시를 들면
- 리스트에 항목들을 추가하고 싶은 경우, 생상자는 **LPUSH**를 호출
- 리스트에서 항목을 추출하거나 처리할때는 소비자가 **RPOP**을 호출

하지만, 리스트에 처리할 항목이 아무것도 없을 경우가 있다. 이때는 **RPOP**은 NULL을 반환한다. 
이러한 경우에는 소비자는 일정 시간동안 대기를 하게 되고, **RPOP**을 다시 시도한다. 
이것을 **polling**이라고 부르며, 다음과 같은 단점들 때문에 이러한 경우에는 좋지 않은 방법이다:

1. Redis와 클라이언트에 필요없는 명령을 강제한다. (리스트가 비어있을 경우는 단지 실질적인 작업없이 NULL만 반환한다.)
2. 작업자가 NULL을 받음으로써 항목들을 처리하는데 지연이 생긴다. 이 경우 RPOP 호출간 인터벌을 줄일 수 있지만, 오히려 1번 문제가 심각해지는 단점이 있다.

~~~redis
> RPUSH bikes:reparis bike:1 bike:2
(integer) 2
> BRPOP bikes:repairs 1
1) "bikes:repairs"
2) "bikes:2"
> BRPOP bikes:repairs 1
1) "bikes:repairs"
2) "bike:1"
> BRPOP bikes:repairs 1
(nil)
(2.01s)
~~~

이것은 bikes:repairs에 요소가 들어올 때까지 대기하고, 만약 1초 동안 가능한 요소가 없으면 반환하라는 뜻이다.

타임아웃 값을 0으로 설정하면 새로운 요소가 들어올 때까지 무한정 기다린다. 또한, 하나의 리스트뿐만 아니라 여러 리스트들이 동시에 대기하게 지정할 수 있으며, 
가장 먼저 요소가 추가된 리스트에서 알림을 받을 수 있다.

주의점

1. 클라이언트는 순서대로 처리된다: 가장 먼저 대기하던 클라이언트가 리스트에 요소가 추가됐을때, 먼저 처리되고 그 다음 순서대로 처리
2. RPOP과 다르게 반환 값이 다르다: 두 개 요소를 가지는 배열이며, 요소가 추가된 키의 이름과 추출된 값이다.
3. 만약 타임아웃에 도달하면, NULL을 반환한다.

## Automatic creation and removal of keys

Redis 리스트는 요소를 추가하기 전에 빈 리스트를 생성하거나, 더 이상 요소가 없는 리스트를 직접 삭제할 필요가 없다. 
이런 작업은 Redis가 자동으로 빈 리스트가 존재하면 키들을 삭제하고, 키가 존재하지 않으면, 키가 존재하지 않는 리스트에 요소를 추가할 때 
자동으로 빈 리스트를 생성해준다. 그리고 다음의 자료형들에도 적용된다. (Streams, Sets, Sorted Sets, Hashes)

1. 집합형 데이터 타입에 요소를 추가할 때, 해당 키가 존재하지 않으면, 요소를 추가하기 전에 자동으로 빈 집합형 데이터 타입을 추가한다.
2. 집합형 데이터 타입에서 요소들을 제거할 때, 값이 비게 되면 해당 키는 자동으로 삭제된다. 단, Stream 자료형에서는 이 규칙이 적용되지 않는다.
3. LLEN(리스트의 길이 반환)과 같이 read-only 명령어를 호출하는 것이나, 요소를 제거하는 쓰기 명령어를 비어 있는 키에 대해 호출하면, 해당 명령어가 기대하는 빈 집합형 데이터가 존재하는 것과 동일한 결과를 반환한다.

## Limits

Redis 리스트의 최대 요소 개수는 2^32 - 1 (4,294,967,295).

## Performance

리스트의 첫 번째과 마지막 요소에 접근하는 작업은 O(1)으로 이루어지지만, 리스트 내부의
요소들을 조작하는 명령어들은 O(n)으로 이루어진다. (**LINDEX**, **LINSERT**, **LSET**)
특히 리스트의 크기가 클 경우 성능에 영향을 줄 수 있으니 주의해야 한다.

## Alternatives

불확정적인 이벤트들을 처리해야 할 경우에는 리스트 대신 **Redis Streams**를 고려해보라고 한다.

