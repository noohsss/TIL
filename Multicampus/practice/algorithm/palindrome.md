# 회문(Palindrome)

- 거꾸로 읽어도 원래 단어와 같은 단어
- 문자열을 뒤집어서 비교하거나, 양쪽 글자를 직접 비교해 확인할 수 있다.

## 1. 슬라이싱으로 뒤집기

```python
def is_palindrome(word):
    reversed_word = word[::-1]
    return word == reversed_word
```

`[::-1]`을 사용하면 문자열을 쉽게 뒤집을 수 있다.  
가장 간단하고 이해하기 쉬운 방법이다.

## 2. 반복문으로 직접 뒤집기

```python
def is_palindrome(word):
    reversed_word = ""

    for index in range(len(word) - 1, -1, -1):
        reversed_word += word[index]

    return word == reversed_word
```

문자열의 마지막 인덱스부터 시작해서 글자를 하나씩 추가한다.  
슬라이싱 없이 반복문으로 문자열을 뒤집는 방법이다.

## 3. 문자를 앞에 붙여 뒤집기

```python
def is_palindrome(word):
    reversed_word = ""

    for char in word:
        reversed_word = char + reversed_word

    return word == reversed_word
```

문자를 기존 문자열의 앞에 붙이면 결과적으로 문자열이 거꾸로 만들어진다.

## 4. 양쪽 글자를 직접 비교하기

```python
def is_palindrome(word):
    for start in range(len(word) // 2):
        end = len(word) - start - 1

        if word[start] != word[end]:
            return False

    return True
```

앞에서부터의 글자와 뒤에서부터의 글자를 비교한다.  
양쪽을 비교하기 때문에 문자열의 절반만 확인해도 된다.

## 5. 투 포인터 사용하기

```python
def is_palindrome(word):
    left = 0
    right = len(word) - 1

    while left < right:
        if word[left] != word[right]:
            return False

        left += 1
        right -= 1

    return True
```

`left`와 `right` 포인터를 양쪽 끝에 두고 가운데로 이동시키면서 비교한다.  
문자열을 새로 뒤집지 않고 확인할 수 있다는 점이 특징이다.

## 정리

처음에는 문자열을 뒤집어서 비교하는 방법이 가장 쉽다.  
반복문을 사용하면 문자열을 직접 뒤집는 과정을 이해할 수 있다.  
문자열을 뒤집지 않고 양쪽 글자를 비교하는 방법은 메모리를 더 효율적으로 사용할 수 있다.
