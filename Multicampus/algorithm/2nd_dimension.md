# 2차원 리스트

- 리스트 안에 리스트가 들어 있는 형태
- 행(row)과 열(column)의 인덱스로 값에 접근한다.

```python
mat = [
    [0, 1, 2, 3],
    [4, 5, 6, 7],
    [8, 9, 10, 11],
    [12, 13, 14, 15]
]
```

바깥쪽 리스트는 행(row), 안쪽 리스트는 열(column)이라고 생각하면 된다.

## 1. 2차원 리스트의 값 가져오기

`mat[행][열]`의 형태로 값을 가져온다.

```python
print(mat[1][2])  # 6
```

`mat[1]`은 두 번째 행 전체를 의미한다.

```python
row = mat[1]
print(row)     # [4, 5, 6, 7]
print(row[2])  # 6
```

인덱스는 0부터 시작한다.

## 2. 가로 방향으로 출력하기

행을 먼저 고정하고, 열을 움직인다.

```python
n = 4

for i in range(n):
    for j in range(n):
        print(mat[i][j])
```

출력 순서는 다음과 같다.

```text
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
```

- 바깥쪽 반복문: 행을 움직인다.
- 안쪽 반복문: 현재 행에서 열을 움직인다.

## 3. 세로 방향으로 출력하기

열을 먼저 고정하고, 행을 움직인다.

```python
n = 4

for j in range(n):
    for i in range(n):
        print(mat[i][j], end=" ")

print()
```

출력 순서는 다음과 같다.

```text
0 4 8 12 1 5 9 13 2 6 10 14 3 7 11 15
```

가로 방향과 반대로 반복문의 순서를 바꾸면 세로 방향으로 탐색할 수 있다.

## 4. 세로 방향으로 더하기

각 열의 값을 더해서 새로운 리스트를 만들 수 있다.

```python
n = 4
total_lst = []

for j in range(n):
    total = 0

    for i in range(n):
        total += mat[i][j]

    total_lst.append(total)

print(total_lst)
```

결과:

```text
[24, 28, 32, 36]
```

열 하나를 고정하고 행을 움직이면서 값을 더한다.

## 5. `n * m` 크기의 2차원 리스트 만들기

0부터 시작해서 순서대로 숫자가 들어 있는 `n * m` 크기의 리스트를 만들 수 있다.

```python
n = 4
m = 5

num = 0
mat = []

for i in range(n):
    row = []

    for j in range(m):
        row.append(num)
        num += 1

    mat.append(row)

print(mat)
```

결과:

```python
[
    [0, 1, 2, 3, 4],
    [5, 6, 7, 8, 9],
    [10, 11, 12, 13, 14],
    [15, 16, 17, 18, 19]
]
```

행과 열의 크기가 다를 수 있기 때문에 숫자를 계산할 때는 `m * i + j`를 사용해야 한다.

```python
n = 4
m = 5

mat = [
    [m * i + j for j in range(m)]
    for i in range(n)
]

print(mat)
```

## 6. 0으로 채워진 빈 grid 만들기

알고리즘 문제에서는 0으로 채워진 2차원 리스트를 자주 사용한다.

```python
n = 4
m = 5

mat = [[0] * m for _ in range(n)]

print(mat)
```

결과:

```python
[
    [0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0]
]
```

각 행마다 새로운 리스트가 만들어진다.

## 7. 2차원 리스트를 만들 때 주의할 점

다음과 같이 작성하면 안 된다.

```python
lst = [0] * m
mat = []

for _ in range(n):
    mat.append(lst)
```

이렇게 하면 모든 행이 같은 리스트를 가리키게 된다.

```python
mat[0][0] = 100
print(mat)
```

첫 번째 행만 바꾸려고 했지만 모든 행의 첫 번째 값이 함께 바뀐다.

## 8. 올바르게 2차원 리스트 만들기

반복할 때마다 새로운 리스트를 만들어야 한다.

```python
mat = []

for _ in range(n):
    row = [0] * m
    mat.append(row)

mat[0][0] = 100
print(mat)
```

이제 첫 번째 행의 첫 번째 값만 변경된다.

## 9. 리스트의 참조 관계

리스트를 대입하면 값이 복사되는 것이 아니라 같은 리스트를 가리키게 된다.

```python
value = [1, 2, 3, 4]
value2 = [1, 2, 3, 4]
value3 = value

value[0] = 100

print(value)   # [100, 2, 3, 4]
print(value2)  # [1, 2, 3, 4]
print(value3)  # [100, 2, 3, 4]
```

`value3 = value`로 작성했기 때문에 `value`와 `value3`은 같은 리스트를 가리킨다.

## 정리

- 2차원 리스트의 값은 `mat[행][열]`로 접근한다.
- 가로 탐색은 행을 고정하고 열을 움직인다.
- 세로 탐색은 열을 고정하고 행을 움직인다.
- 2차원 리스트를 만들 때는 각 행을 새로 만들어야 한다.
- `[[0] * m for _ in range(n)]` 형태를 자주 사용한다.
- `mat.append(lst)`를 반복하면 모든 행이 같은 리스트를 가리킬 수 있다.
- 리스트를 대입하면 복사되지 않고 같은 객체를 참조할 수 있다.
