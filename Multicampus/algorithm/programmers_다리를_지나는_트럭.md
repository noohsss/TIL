# 프로그래머스 - 다리를 지나는 트럭

[문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/42583)

## 문제 이해

트럭이 순서대로 다리를 건너는 데 걸리는 시간을 구하는 문제다.

- 다리에는 정해진 길이만큼 트럭이 올라갈 수 있다.
- 다리 위 트럭의 무게 합은 최대 하중을 넘을 수 없다.
- 트럭이 다리를 통과하는 데 걸리는 시간은 다리의 길이와 같다.
- 다리 위에 빈칸이 생기면 `0`으로 표현한다.

## `deque`를 이용한 풀이

다리는 먼저 들어온 트럭이 먼저 나가는 구조이므로 큐로 관리한다.

```python
from collections import deque


def solution(bridge_length, weight, truck_weights):
    # 다리 길이만큼 빈칸을 만든다.
    bridge = deque([0] * bridge_length)
    trucks = deque(truck_weights)

    current_weight = 0
    time = 0

    while trucks or current_weight > 0:
        time += 1

        # 다리에서 나가는 트럭을 제거한다.
        out_truck = bridge.popleft()
        current_weight -= out_truck

        if trucks and current_weight + trucks[0] <= weight:
            # 다음 트럭이 올라갈 수 있으면 추가한다.
            truck = trucks.popleft()
            bridge.append(truck)
            current_weight += truck
        else:
            # 트럭이 올라갈 수 없으면 빈칸을 추가한다.
            bridge.append(0)

    return time


print(solution(2, 10, [7, 4, 5, 6]))  # 8
print(solution(100, 100, [10]))        # 101
print(solution(100, 100, [10] * 10))   # 110
```

## 코드 이해하기

### 다리 상태 초기화

```python
bridge = deque([0] * bridge_length)
```

다리의 길이만큼 `0`을 넣어 빈 다리를 만든다.

`0`은 트럭이 없는 빈칸을 의미한다.  
빈칸도 시간에 따라 한 칸씩 이동해야 하므로 다리의 상태에 포함한다.

### 트럭 대기열 만들기

```python
trucks = deque(truck_weights)
```

아직 다리에 올라가지 않은 트럭을 큐에 저장한다.

가장 앞에 있는 트럭부터 다리에 올라가야 하므로 `deque`를 사용한다.

### 다리에서 트럭 내보내기

```python
out_truck = bridge.popleft()
current_weight -= out_truck
```

매 시간마다 다리의 가장 왼쪽 값을 꺼낸다.

- 꺼낸 값이 `0`이면 빈칸이 나간다.
- 꺼낸 값이 트럭의 무게라면 현재 무게에서 뺀다.

### 다음 트럭 올리기

```python
if trucks and current_weight + trucks[0] <= weight:
    truck = trucks.popleft()
    bridge.append(truck)
    current_weight += truck
else:
    bridge.append(0)
```

다음 트럭이 있고, 현재 다리 위 무게에 다음 트럭을 더해도 최대 하중을 넘지 않으면 트럭을 올린다.

트럭이 올라갈 수 없으면 `0`을 넣어 빈칸을 유지한다.

### 반복 조건

```python
while trucks or current_weight > 0:
```

다음 트럭이 남아 있거나, 아직 다리 위에 트럭이 있으면 계속 반복한다.

두 조건이 모두 거짓이 되면 모든 트럭이 다리를 건넌 것이므로 반복을 종료한다.

## 정리

- 트럭이 먼저 들어온 순서대로 나가므로 큐를 사용한다.
- 다리 위의 빈칸은 `0`으로 표현한다.
- `current_weight`로 현재 다리 위 트럭들의 무게 합을 관리한다.
- 다음 트럭이 올라갈 수 없을 때는 `0`을 넣어 다리의 길이를 유지한다.
- 모든 트럭이 대기열과 다리에서 사라질 때까지 시간을 센다.
