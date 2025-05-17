# Redis Strings

Redis는 기본적으로 Key-Value 구조를 따른 인메모리 기반의 데이터베이스이다.
Redis에서는 키와 밸류 값에 문자열을 기본 자료형으로 삼고, 이를 가지고 여러 연산을
할 수 있도록 지원하고 있다. 

## 기본 명령어

```redis
> SET bike:1 Deimos
OK
> GET bike:1
"Deimos"
```

- **SET**을 통해 문자열 값을 키에 설정
  - 만약 키에 기존 값이 존재하면 기존 값을 대체하여 새 값을 할당
- **GET**을 통해 이를 조회

```redis
> SET bike:1 bike NX
(nil)
> SET bike:1 bike XX
OK
```
- **NX**: 키가 존재하지 않을 경우에만 저장
- **XX**: 키가 존재할 때만 덮어쓰기

```redis
> MSET bike:1 "Deimos" bike:2 "Ares" bike:3 "Vanth"
OK
> MGET bike:1 bike:2 bike:3
1) "Deimos"
2) "Ares"
3) "Vanth"
```

- **MSET**, **MGET**으로 여러 키를 한 번에 조회가능

## 카운터 로서의 문자열

Redis에서는 문자열을 통해 원자적 증가가 가능하다.

```redis
> SET total_crashes 0
OK
> INCR total_crashes
(integer) 
> INCRBY total_crashes 10
(integer) 11
```

**INCR**을 통해 문자열 값을 정수로 해석한 뒤 1씩 증가시키고, 증가된 값을 새 값으로 
설정한다. INCRBY, DECR, DECRBY 등등도 있다.

원자적 증가를 통해, 동시성 문제가 발생해도 경쟁 상태가 발생하지 않는다.

## Limits
- 단일 Redis 문자열을 512MB까지 가능하다.

## 정리
- **SET**: 문자열 값을 저장
- **SETNX**: 키가 존재하지 않을 경우에만 값을 저장. (락 구현에 유용)
- **GET**: 문자열 값을 조회
- **MGET**: 여러 개의 키 값을 한 번에 조회합니다.
- **INCR**: 값을 원자적으로 1 증가
- **INCRBY**: 값을 지정한 수 만큼 원자적으로 증가 
- **INCRBYFLOAT**: 소숫점을 값을 가지는 카운터 조작

### Bitwise Operations

문자열에서 비트 연산이 수행 가능함.

## Performance

대부분의 문자열 명령어는 시간 복잡도 O(1)로 매우 효율적으로 작동한다. 다만,
**SUBSTR**, **GETRANGE**, **SETRANGE** 등은 O(n) 복잡도를 가지므로, 
큰 문자열을 다룰 때 성능 이슈가 발생할 수 있습니다.

### Alternatives

직렬화된 문자열로 구조화된 데이터를 저장하려는 경우, Redis의 해시(Hash) 또는 JSON 자료형을 사용하는 것을 고려할 수 있습니다.

