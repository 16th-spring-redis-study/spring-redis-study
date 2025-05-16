# Redis Sorted Set
- Sorted Set은 고유한 문자열(member)들의 집합으로, 각 유니크한 문자열에 연관된 score가 있어서 score를 기준으로 정렬된다.
- 여러 문자열이 같은 score를 가질 경우, 문자열 사전식으로 정렬된다.


## Sorted Set의 특징
### 1. Sorted Set은 Set과 Hash의 특징을 모두 가진다.
- Sorted Set은 고유한 문자열 요소들로 구성되며 중복이 없다는 점에서 일반적인 Set의 개념과 유사하다.
- 또한, 각 요소가 score에 매핑되어 있어 Hash 자료 구조와 같이 KV 쌍 구조로 볼 수 있다.
- 정렬된 상태를 유지하고 싶으면서도 빠른 조회가 필요한 경우에 사용한다.

### 2. 정렬 방식의 특징
- 요소들이 요청 시 정렬되는게 아니라, 자료구조 자체가 이미 정렬된 상태로 데이터를 유지한다.
- 여러 문자열이 같은 score를 가질 때, 문자열의 사전식으로 정렬되는데 이 때 문자열은 유니크한 것만 키로 허용되므로 자료 구조 안에 서로 같은 문자열 키가 존재할 수 없다.


## Redis Sorted Set의 핵심 구조
- 정렬과 빠른 조회를 동시에 가능하게 하기 위해 내부적으로 2가지 구조를 동시에 사용한다. (HashTable + Skip List)
- 이중 구조라서 복잡하거나 무거울거라고 생각


| 구조             | 역할                        | 시간 복잡도                 |
| -------------- | ------------------------- | ---------------------- |
| **Hash Table** | 값 → 점수 (value → score) 매핑 | O(1) 조회                |
| **Skip List**  | 점수 → 값 정렬 구조 유지           | O(log N) 삽입, 삭제, 범위 조회 |

> `Skip List`가 트리와 유사한 구조를 유지하여 트리와 유사한 수준인 O(log N)으로 탐색할 수 있다.



## Sorted Set의 대표적인 활용 사례
- LeaderBoards
- RateLimiters
- TimeLines
- LRU


---
ref.
- [Redis docs: Redis Sorted Set](https://redis.io/docs/latest/develop/data-types/sorted-sets/)
- [Redis Skip List](https://medium.com/@chnwsw01/redis-skiplist-20e35831d9e7)