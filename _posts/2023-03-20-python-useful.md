---
title: 유용한 Python Function 모음
author: avokey
date: 2023-03-20 18:32:00 -0500
categories: [Dev, Python]
tags: [Python, Pythonic, Programming, Practical]
---
<br>

![Desktop View](/common/python.png){: width="972" height="589" }
_Fig 1. Python_


이번 포스트에선 자주 사용하는 Python Function들을 정리 해보려 합니다.
# zip()

## 동작 방식

기본적인 동작 방식은 `Iterable` 객체를 인자로 받고, `Iterator` 반복자를 반환하는 형태입니다. Iterable한 객체는 순회가 가능해야하고, Iterator은 튜플 형태입니다.

~~~python
>>> numbers = [1, 2, 3]
>>> letters = ["A", "B", "C"]
>>> for pair in zip(numbers, letters):
...     print(pair)
...
(1, 'A')
(2, 'B')
(3, 'C')
~~~

<hr style="height:20px; visibility:hidden;" />

## zip 활용법 1: Unpacking

`zip()` 함수는 가변인자를 받기 때문에 2개 이상 인자를 넘겨 여러 그룹의 원소를 하나씩 출력이 가능합니다.

~~~python
>>> for number, upper, lower in zip("12345", "ABCDE", "abcde"):
...     print(number, upper, lower)
...
1 A a
2 B b
3 C c
4 D d
5 E e
~~~

## zip 활용법 2: Unzip

`zip()` 함수로 엮어 놓은 데이터를 다시 해체(unzip)하고 싶을 때도 `zip()` 함수를 사용할 수 있습니다.

~~~python
# zip
>>> numbers = (1, 2, 3)
>>> letters = ("A", "B", "C")
>>> pairs = list(zip(numbers, letters))
>>> print(pairs)
[(1, 'A'), (2, 'B'), (3, 'C')]

# unzip
>>> numbers, letters = zip(*pairs)
>>> numbers
(1, 2, 3)
>>> letters
('A', 'B', 'C')
~~~

## zip 활용법 3: ToDict

`zip()` 함수를 이용하면 두 개의 리스트나 튜플 부터 쉽게 사전(dictionary)을 만들 수 있습니다. 키를 담고 있는 리스트와 값을 담고 있는 리스트를 `zip()` 함수에 넘긴 후, 그 결과를 다시 `dict()` 함수에 넘기면 됩니다.

~~~python
# Case 1
>>> keys = [1, 2, 3]
>>> values = ["A", "B", "C"]
>>> dict(zip(keys, values))
{1: 'A', 2: 'B', 3: 'C'}

# Case 2
>>> dict(zip(["year", "month", "date"], [2001, 1, 31]))
{'year': 2001, 'month': 1, 'date': 31}
~~~

## zip 활용법 4: Compare

~~~python
string1 = "apple pie available".split()
string2 = "apple have some apple pies".split()

for word1, word2 in zip(string1, string2):
    if word1 == word2:
      print(word1)
>>> "apple"
~~~

<br>