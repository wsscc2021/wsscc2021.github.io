---
title: Pyhton - TypeError 줄이기 (Function Annotation, MyPy)
author: D.G.Lee
date: 2021-10-02 16:00:00 +0900
categories: [Python]
tags: [Python]
math: true
mermaid: true
---



## Overview

파이썬을 다루다가 보면 실행 중에 갑작스런 `TypeError`를 접했던 경험이 종종 있었습니다. 이를 몇 번 접하고 나서 Type에 대해 정확하게 이해하고 이러한 오류를 줄일 수 있는 방법을 찾아본 내용을 공유합니다.



## 데이터 타입

데이터 타입은 프로그래밍 언어에서 데이터의 형태를 의미합니다. (int, float, string, list, dictionary ...)

프로그램 상에서 데이터는 메모리에 저장되어 사용됩니다. 데이터 타입은 데이터가 메모리에 저장될 때 확보해야하는 메모리 크기를 결정하고 메모리에 저장되어 있는 2진수를 어떻게 해석할 지에 대해 컴퓨터에게 알려주기 위해 존재합니다. 이를 통해 다양한 데이터 형태를 표현하더라도 메모리 공간을 효율적으로 활용할 수 있습니다.



## 정적 타입 vs 동적 타입

**정적 타입** : **"정적 타입은 개발자가 직접 코드 상에 타입을 지정하며, 컴파일 시 자료형이 결정됩니다."** 이는 타입 에러로 인한 문제점을 조기에 발견하여 프로그램 실행 중 에러가 발생하는 것을 최소화할 수 있습니다. (e.g. Java)

**동적 타입** : **"동적 타입은 개발자가 별도로 타입을 지정하지 않으며, 컴파일 시 자료형이 결정되는 것이 아니라 런타임에 결정됩니다."** 이는 작고 간단한 프로젝트나 스크립트의 개발 속도를 높일 수 있는 요소가 됩니다. (e.g. Python, Javascript)

\>  

컴파일 시 에러가 발생하는 것은 개발의 속도를 늦추는 요소가 될 수 있지만, 런타임 에러를 최소화할 수 있는 장점을 가집니다. 이는 런타임 에러 발생 시 큰 영향을 받을 수 있는 대규모 프로젝트를 진행할 때 긍정적으로 생각할 수 있습니다. 
반면 동적 타입의 경우 빠른 개발이 필요한 프로젝트나 스크립트에서 긍정적으로 생각할 수 있습니다.



## 약타입 vs 강타입

동적 타입 언어는 약타입과 강타입 으로 다시 구분됩니다.

- 약타입 : 자료형이 맞지 않을 시에 프로그래밍 언어 자체에서 암묵적으로 타입을 변환합니다. (e.g. javascript)
- 강타입 : 자료형이 맞지 않을 때 에러가 발생합니다. (암묵적 변환을 지원하지 않습니다.) (e.g. python)

이해를 위해 아래 예시를 살펴 보면, 자바스크립트의 경우 개발자가 integer 타입으로 사용하기 위해 따옴표 없이 표현했더라도 `+` operation을 처리하기 위해서 프로그래밍 언어 자체에서 암묵적으로 String 타입으로 변환하여 프로그램을 실행시킵니다. 반면 파이썬의 경우 `+` operation을 수행할 수 없다며 `TypeError` 를 발생시킵니다.

```javascript
console.log(1+"1");
// 출력: 11
```

```python
print(1+"1")
# 에러 : TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

\>  

- 약타입
    - 장점 : 런타임 에러를 최소화하여 프로그램이 끝까지 실행되도록 할 수 있습니다.
    - 단점 : 개발자가 원하지 않는 동작을 프로그램에서 실행할 수 있으며, 이러한 동작 오류는 에러가 발생하는 것이 아니기 때문에 찾기가 힘듭니다.
- 강타입:
    - 장점 : 암묵적 변환이 이뤄지지 않기 때문에 디버깅하기 쉬워집니다.
    - 단점 : 런타임 에러가 발생하여 프로그램이 실행 도중 에러를 발생시켜 프로그램이 동작하지 않을 수 있습니다.

실제로 자바스크립트는 동적 타입 중에서도 약타입을 사용하기 때문에 큰 프로젝트에서 단점이 극명하게 드러났습니다. 자바스크립트가 node.js 를 통해 백엔드에서 활용되는 등 다양한 곳에서 활용되고 점차 발전해나가자 이러한 단점을 극복하기 위해 정적 타입을 지원하는 타입스크립트가 개발되었습니다.



## Duck Typing

Duck Typing은 '오리처럼 걷고, 오리처럼 꽥꽥거리면, 그것은 틀림없이 오리다.' 라는 의미를 가지고 있습니다.

이는 개발자가 별도로 타입을 지정하지 않고, 프로그래밍 언어가 데이터의 모습을 보고 판단하여 타입을 결정한다는 의미로 해석할 수 있습니다.

```python
a = 1
b = "1"
print(type(a))
print(type(b))
```

```
<class 'int'>
<class 'str'>
```



## Python - Function Annotation

파이썬은 동적 타입 언어이며 Duck Typing 특징을 가지고 있기 때문에, 데이터 타입에 대한 가독성이 다소 떨어질 수 있습니다. 만약 파이썬으로 다른 개발자와 협업하거나 대규모 프로젝트를 진행할 때에는 가독성을 위해 코드 상에 타입을 명확하게 표현할 수 있는 방법이 필요합니다. 

Function Annotation은 데이터 타입의 힌트를 제공하며, 이를 통해 코드 상에 타입을 명확하게 표현할 수 있습니다. (**타입을 제한하는 것이 아니라 힌트를 주는 것입니다.)

```python
def add(a: int, b: int) -> int: # return type is defined "-> int"
    return a + b

num_a: int = 1
num_b: int = 2
add(num_a,num_b)
```

Function Annotation 기능은 Python3 부터 지원하기 시작했으며 이전에는 Comment를 사용하여 타입을 명시 했었습니다. 

```python
def add(a,b):
    """
        a: int
        b: int
        return type: int 
    """
    return a + b

num_a = 1 # type: int
num_b = 2 # type: int
add(num_a,num_b)
```

저는 기능에 대한 간략한 흐름과 사용하는 이유에 대해 설명하고 있지만 자세한 문법과 사용방법은 공식 홈페이지를 통해 배우는 것이 가장 정확하고 효율적이라고 생각합니다.
- [https://www.python.org/dev/peps/pep-3107/](https://www.python.org/dev/peps/pep-3107/) (Function Annotation)
- [https://docs.python.org/3/library/typing.html](https://docs.python.org/3/library/typing.html) (typing - standard library)




## Pyhthon - MyPy

파이썬은 동적 타입 언어이기 때문에 런타임 타입에러가 발생할 수 있습니다. 런타임 에러는 프로그램이 실행 중에 발생하는 에러로써 운영환경에서 발생할 경우 사용자 경험이 낮아질 수 있습니다. 때문에 프로그램 실행 전에 타입에러를 발견하고 조치할 필요가 있습니다.

MyPy는 정적 타입 체크를 도와주는 파이썬의 Standard Library 입니다. (정적 테스트는 프로그램 실행 전에 코드를 기반으로 테스트를 진행하는 테스트 기법입니다.) 예시를 통해 어떻게 동작하는 간단하게 살펴봅니다.

`MyPy Library 설치 `

```bash
python3 -m pip install mypy
```

`type_python.py`

```python
def add(a: int, b: int) -> int:
    return a + b

num_a: int = 1
num_b: int = 2
add(num_a, num_b)
```

위와 같이 타입 에러가 발생하지 않는 코드를 mypy를 통해 테스트해보면 아래와 같이 성공 메시지를 반환받습니다.

```bash
python3 -m mypy type_python.py
# Success: no issues found in 1 source file
```

만약 타입 에러가 포함된 코드를 테스트하면 아래와 같이 실패 메시지와 에러가 발생하는 시점을 확인할 수 있습니다.

```python
def add(a: int, b: int) -> int:
    return a + b

num_a: int = 1
num_b: str = "a"
add(num_a, num_b)
```

```bash
python3 -m mypy type_python.py
# type_python.py:7: error: Argument 2 to "add" has incompatible type "str"; expected "int"
# Found 1 error in 1 file (checked 1 source file)
```


Function Annotation과 같은 맥락으로 자세한 내용은 공식 홈페이지를 참조합니다.
- [https://github.com/python/mypy](https://github.com/python/mypy) (Github)
- [https://mypy.readthedocs.io/en/stable/index.html#](https://mypy.readthedocs.io/en/stable/index.html#) (ReadHat)


## 결론

파이썬을 사용할 때에는 Function Annotation을 통해 타입을 코드 상에 표현하여 가독성을 높이고 MyPy Library를 사용하여 프로그램을 실행하기 전에 사전에 타입에러를 발견하고 조치하여 런타임 에러를 최소화하는 노력이 필요합니다.