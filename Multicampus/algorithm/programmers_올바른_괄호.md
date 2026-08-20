# 프로그래머스 - 올바른 괄호

[문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/12909)

## 문제 이해

문자열의 괄호가 올바른 순서로 열리고 닫히는지 확인하는 문제다.

프로그래머스 문제에서는 `(`와 `)`만 사용하지만, 같은 방식으로 `[]`와 `{}`까지 확장할 수 있다.

- 여는 괄호는 스택에 넣는다.
- 닫는 괄호는 가장 최근에 열린 괄호와 짝이 맞는지 확인한다.
- 닫는 괄호가 나왔는데 스택이 비어 있으면 올바르지 않다.
- 모든 괄호를 확인한 뒤 스택이 비어 있어야 올바르다.

## `deque`를 이용한 풀이

```python
from collections import deque


def solution(s):
    stack = deque()

    open_brackets = ["(", "[", "{"]

    bracket_pair = {
        ")": "(",
        "]": "[",
        "}": "{"
    }

    for char in s:
        if char in open_brackets:
            stack.append(char)

        else:
            if char in bracket_pair:
                if not stack:
                    return False

                if stack[-1] != bracket_pair[char]:
                    return False

                stack.pop()

    if stack:
        return False

    return True


print(solution("()()"))    # True
print(solution("(())()"))  # True
print(solution(")()("))    # False
print(solution("(()("))    # False
print(solution("([{}])"))  # True
print(solution("([)]"))    # False
```

## 코드 이해하기

### 여는 괄호 저장하기

```python
if char in open_brackets:
    stack.append(char)
```

여는 괄호가 나오면 스택에 저장한다.

```text
"(["를 읽었을 때
stack = ["(", "["]
```

### 닫는 괄호 확인하기

닫는 괄호가 나오면 스택의 마지막 값과 비교한다.

```python
if stack[-1] != bracket_pair[char]:
    return False
```

가장 최근에 열린 괄호가 현재 닫는 괄호와 짝이 맞아야 한다.

```text
"([)]"

")"를 확인할 때
stack의 마지막 값: "["
")"와 짝인 괄호: "("

서로 다르므로 False
```

### stack.pop()의 의미

짝이 맞는 괄호를 확인했으면 스택에서 여는 괄호를 제거한다.

```python
stack.pop()
```

### 마지막에 스택 확인하기

모든 문자를 확인한 뒤에도 스택에 괄호가 남아 있으면 닫히지 않은 괄호가 있다는 뜻이다.

```python
if stack:
    return False

return True
```

## 정리

- 괄호의 순서를 확인하기 위해 스택을 사용한다.
- deque는 append()와 pop()을 이용해 스택처럼 사용할 수 있다.
- 닫는 괄호는 스택의 마지막 괄호와 비교한다.
- 괄호 종류가 여러 개라면 bracket_pair 딕셔너리로 짝을 관리할 수 있다.
- 마지막에 스택이 비어 있어야 올바른 괄호다.
