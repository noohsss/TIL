# 정렬(Sort)

- 데이터를 일정한 순서로 나열하는 것
- 보통 작은 값부터 나열하는 오름차순을 기준으로 생각한다.

## 1. `sort()` 메서드

```python
lst = [2, 1, 6, 8, 3, 7]

result = lst.sort()

print(result)  # None
print(lst)     # [1, 2, 3, 6, 7, 8]
```

`sort()`는 리스트 자체를 정렬한다.  
따라서 원래 리스트가 변경되고, 반환값은 `None`이다.

## 2. `sorted()` 함수

```python
lst = [2, 1, 6, 8, 3, 7]

result = sorted(lst)

print(result)  # [1, 2, 3, 6, 7, 8]
print(lst)     # [2, 1, 6, 8, 3, 7]
```

`sorted()`는 정렬된 새로운 리스트를 반환한다.  
따라서 원래 리스트는 변경되지 않는다.

## 3. 딕셔너리를 이용한 정렬

```python
lst = [1, 3, 2, 5, 4, 6, 8, 7, 9, 10, 3, 5, 3, 5, 1]

count = {}

for num in lst:
    if count.get(num) is None:
        count[num] = 0

    count[num] += 1

sorted_lst = []

for num in range(1, 11):
    for _ in range(count[num]):
        sorted_lst.append(num)

print(sorted_lst)
```

각 숫자가 몇 번 나오는지 딕셔너리에 저장한다.  
그다음 숫자를 작은 순서대로 확인하면서 개수만큼 리스트에 추가한다.

## 4. 카운팅 정렬

```python
lst = [1, 3, 2, 5, 4, 6, 8, 7, 9, 10, 3, 5, 3, 5, 1]

counting_lst = [0] * 11

for num in lst:
    counting_lst[num] += 1

sorted_lst = []

for num in range(1, 11):
    for _ in range(counting_lst[num]):
        sorted_lst.append(num)

print(sorted_lst)
```

숫자의 개수를 리스트의 인덱스에 저장한다.

예를 들어 `counting_lst[3]`은 숫자 `3`이 나온 횟수이다.

카운팅 정렬은 정렬할 데이터의 범위가 작고 정해져 있을 때 유용하다.  
숫자의 범위를 모르는 경우에는 사용하기 어렵다.

## 5. 버블 정렬

버블 정렬은 서로 붙어 있는 두 숫자를 비교한다.  
왼쪽 숫자가 더 크면 두 숫자의 위치를 바꾼다.

```python
lst = [6, 5, 2, 3, 8, 7, 4, 1, 0]

for j in range(1, len(lst)):
    for i in range(len(lst) - j):
        if lst[i] > lst[i + 1]:
            lst[i], lst[i + 1] = lst[i + 1], lst[i]

print(lst)
```

한 번 반복할 때마다 가장 큰 숫자가 오른쪽 끝으로 이동한다.  
그래서 다음 반복에서는 이미 정렬된 오른쪽 부분을 제외한다.

## 6. 선택 정렬

선택 정렬은 아직 정렬되지 않은 부분에서 가장 작은 값의 위치를 찾는다.  
찾은 값을 현재 위치의 값과 바꾼다.

```python
lst = [6, 5, 4, 2, 8, 7, 3, 1, 0]

for start_index in range(len(lst) - 1):
    min_index = start_index

    for i in range(start_index, len(lst)):
        if lst[i] < lst[min_index]:
            min_index = i

    lst[start_index], lst[min_index] = (
        lst[min_index],
        lst[start_index]
    )

print(lst)
```

첫 번째 반복에서는 전체에서 가장 작은 값을 찾아 맨 앞에 둔다.  
두 번째 반복에서는 첫 번째 값을 제외하고 가장 작은 값을 두 번째에 둔다.  
이 과정을 반복하면 리스트 전체가 정렬된다.

## 정리

- `sort()`는 원래 리스트를 직접 변경한다.
- `sorted()`는 새로운 정렬 리스트를 반환한다.
- 카운팅 정렬은 숫자의 범위가 정해져 있을 때 사용하기 좋다.
- 버블 정렬은 이웃한 값끼리 비교하면서 큰 값을 오른쪽으로 보낸다.
- 선택 정렬은 남은 값 중 가장 작은 값을 찾아 앞쪽에 배치한다.
