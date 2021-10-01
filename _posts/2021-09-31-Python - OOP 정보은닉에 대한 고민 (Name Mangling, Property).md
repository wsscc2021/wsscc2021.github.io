---
title: Python - OOP 정보은닉에 대한 고민 (Name Mangling, Property)
author: D.G.Lee
date: 2021-10-01 18:00:00 +0900
categories: [Python]
tags: [Python]
math: true
mermaid: true
---

## Overview

파이썬을 OOP 방식으로 사용하려고 보니 "**정보은닉**을 어떻게 만족시킬 수 있을까??" 라는 고민을 하게 되었습니다. Java에서는 접근제어자를 통해 이를 만족시킬 수 있었으나 파이썬에는 접근제어자가 없었기 때문에 "무언가 내가 모르는 방법이 있지 않을까??" 라는 생각으로 찾아보았습니다.

우선 정보은닉이 무엇인지 이해할 필요가 있습니다. OOP에서는 연관 있는 데이터와 함수를 클래스로 묶고, 구현 내용을 외부에 감추는 것을 캡슐화라고 하며 캡슐화된 클래스에서 중요한 데이터나 기능을 외부에서 접근하지 못하도록 구성하는 것을 정보은닉이라고 합니다. 이를 통해 외부에서 함부로 데이터를 읽거나 변경하지 못하고 개발자가 원하는 범위 안의 동작만 수행하도록 제한하여 보안성을 높이고 에러를 최소화할 수 있습니다.



## Name Mangling

가장 먼저 찾을 수 있던 내용이 Name Mangling입니다. Name Mangling은 Java의 접근제어자와 비교하여 설명하는 경우가 많았기 때문에 "접근제어자와 비슷한 역할인 건가??" 라는 생각으로 살펴보았습니다. (전혀 다른 의미였지만요... ㅎㅎ...)

Name Mangling은 함수나 변수명을 컴파일 단계에서 컴파일러가 일정한 규칙을 가지고 변형하는 것을 의미합니다. (*Name Decoration이라고도 합니다.) 파이썬에서는 `_` 또는 `__`를 변수 또는 함수앞에 접미사로 붙여서 사용할 수 있습니다.

Name Mangling은 다른 범위에 있는 같은 이름 함수나 변수를 구별하기 위한 목적으로 사용됩니다. (예를 들면 하위 클래스에서 상속받은 변수와 같은 이름의 변수를 사용할 때...) 사실 굳이 왜 이렇게 사용해야하는 지 이해가 잘 안되어서 예시를 하나 들어봤습니다.

전기차를 상속을 통해 구현할 때 자동차 등록코드와 전기차 등록코드를 모두 등록해야된다고 가정해봅니다. 적절한 변수명이 `code`라고 했을 때 자동차 등록코드가 전기차 등록코드로 오버라이딩 되어 버립니다. 완벽히 이해하지 않아도 되니 잠깐 살펴봅니다.

```python
class Car:
    def __init__(self, code):
        self.code = code

class ElectricCar(Car):
    def __init__(self, code):
        self.code = code

tesla = ElectricCar("xxxxxxxxxx")
print(tesla.__dict__)
```

```
{'code': 'xxxxxxxxxx'}
```

이럴 때 Name Mangling을 활용하면 오버라이딩을 막을 수 있습니다.

```python
class Car:
    def __init__(self, code):
        self.__code = code

class ElectricCar(Car):
    def __init__(self, car_code, code):
        self._Car__code = car_code
        self.__code = code

tesla = ElectricCar("xxxxxxxxxx","zzzzzzzzzzz")
print(tesla.__dict__)
```

```
{'_Car__code': 'xxxxxxxxxx', '_ElectricCar__code': 'zzzzzzzzzzz'}
```



그리고 처음에 **정보은닉**을 위해 활용된다고 헷갈렸던 포인트(개념)을 정리해봅니다.

```python
class Person:
    def __init__(self,name,age):
        self.name = name
        self.age = age
        self.__secret = "zxj412mazasxio5n321"

person = Person("Lee",26)
print(person.name)
print(person.age)
print(person.__secret) # __secret 속성이 없다는 에러가 출력됩니다.
```

```
Lee
26
Traceback (most recent call last):
  ....
AttributeError: 'Person' object has no attribute '__secret'
```



위 코드를 실행시켜보면 `__secret` 속성이 없다는 에러가 출력됩니다. 이 것은 속성에 접근이 불가능하여 발생하는 에러가 아니라 속성의 이름을 컴파일러에서 바꿨기 때문에 발생하는 에러입니다. 

```python
class Person:
    def __init__(self,name,age):
        self.name = name
        self.age = age
        self.__secret = "zxj412mazasxio5n321"

person = Person("Lee",26)
print(person.name)
print(person.age)
# print(person.__secret) # __secret 속성이 없다는 에러가 출력됩니다.
print(person.__dict__)
```

```
Lee
26
{'name': 'Lee', 'age': 26, '_Person__secret': 'zxj412mazasxio5n321'}
```



실제로 위 코드를 실행시켜보면 `_Person__secret` 이라는 속성이 `person` 객체에 포함되어 있는 것을 확인할 수 있습니다. 파이썬 컴파일러가 컴파일 단계에서 `__secret` 속성의 이름을 `_Person__secret` 으로 변경시킨 것입니다.

그리고 아래 코드를 실행시켜보면 변경된 속성의 이름으로 접근할 수 있는 것을 확인할 수 있습니다. 단순히 속성의 이름만 변경된 것이기 때문에 변경된 속성 이름으로 접근하면 데이터를 읽거나 변경할 수 있는 것입니다. 결론적으로 Name Mangling은 완벽하게 정보은닉을 만족시킬 수 없습니다.

```
class Person:
    def __init__(self,name,age):
        self.name = name
        self.age = age
        self.__secret = "zxj412mazasxio5n321"

person = Person("Lee",26)
print(person.name)
print(person.age)
# print(person.__secret) # __secret 속성이 없다는 에러가 출력됩니다.
# print(person.__dict__)
print(person._Person__secret)
```

```
Lee
26
zxj412mazasxio5n321
```



## Property

그리고 찾아볼 수 있었던 것은 property 데코레이터입니다. property 데코레이터은 getter/setter를 간단하게 생성해주는 파이썬의 기본 데코레이터입니다.

Java에서는 접근 제어자를 통해 데이터에 직접 접근하지 못하도록 보호하고 데이터를 읽거나 수정하는 메소드만 노출시키는 방식으로 정보 은닉을 만족시킵니다. 여기서 데이터를 읽는 메소드를 getter, 데이터를 수정하는 메소드를 setter라고 표현합니다. getter/setter를 활용하면 객체의 데이터를 읽거나 수정할 때 타입체크를 하거나 인증된 사용자인지 확인하는 등의 작업을 수행하여 데이터를 안전하게 보호할 수 있습니다.

property 데코레이터 사용하지 않을 경우

```
class Person:
    def __init__(self, name, age):
        self.name = name
        self.set_age(age)
    
    def get_age(self):
        return self.__age
    
    def set_age(self, age):
        if type(age) is not int:
            raise TypeError("age must be int type")
        else:
            self.__age = age

person = Person("Lee",24)
print(person.get_age())
person.set_age(50)
print(person.get_age())
```

```
24
50
```

property 데코레이터 사용

```
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    @property
    def age(self):
        return self.__age
    
    @age.setter
    def age(self, age):
        if type(age) is not int:
            raise TypeError("age must be int type")
        else:
            self.__age = age

person = Person("Lee",24)
print(person.age)
person.age = 50
print(person.age)
```

```
24
50
```



하지만 property 데코레이터도 결국 Name Mangling을 사용하는 방식이기 때문에 Name Mangling 원리로 변경된 변수명으로 데이터에 직접 접근할 수 있습니다. 결국 property를 사용하더라도 완벽하게 정보 은닉을 만족할 수 없습니다.

```
class Person:
    def __init__(self, name, age):
        self.name = name
        self.set_age(age)
        self.auth = True
    
    def get_age(self):
        if self.auth: # 신뢰하는 사람만 데이터에 접근할 수 있습니다. (*가정)
            return self.__age
        else:
            raise AttributeError("Access Denied")
    
    def set_age(self, age):
        if type(age) is not int: # 나이는 int type이어야 합니다.
            raise TypeError("Must be int type for 'age'")
        else:
            self.__age = age

person = Person("Lee",24)
print(person.get_age())
person._Person__age = "HHHHH"
print(person._Person__age)
```

```
24
HHHHH
```



## 결론

제가 찾아본 한에서 파이썬은 완벽하게 정보은닉을 만족할 수 없습니다. 하지만 Name Mangling과 Property 데코레이터를 적절히 활용하여 최대한 외부에서 간섭하지 못하도록 노력하는 것이 필요합니다.