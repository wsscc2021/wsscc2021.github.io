---
title: Python - 임시 파일 다루기 (tempfile)
author: D.G.Lee
date: 2021-10-04 14:00:00 +0900
categories: [Python]
tags: [Python]
math: true
mermaid: true
---

## Overview

저는 아래와 같은 과정(*경험)을 겪으면서 임시파일을 다루는 것에 대한 필요성을 느꼈습니다.

Python 기반의 Flask 어플리케이션을 개발하면서... 로컬에서 Unit Test를 진행하고자 했습니다. → 어플리케이션에는 사용자 데이터를 데이터베이스에 저장하는 로직이 포함되어 있었기 때문에 이를 테스트 하기 위해서 테스트용 데이터베이스가 필요했습니다. → 테스트용 데이터베이스는 테스트가 시작될 때 임시적으로 생성되고 테스트가 끝나면 삭제되어 리소스를 낭비하지 않기를 원했습니다. → 테스트용 임시 데이터베이스를 위한 OS상의 임시파일이 필요했습니다.  

이를 해결하기 위해서 임시파일을 다루는 방법에 대해서 찾아보았고 이번 글에서는 그 내용을 공유하려 합니다.



## tempfile

파이썬에서는 임시파일을 다루기 위해 `tempfile` 이라는 standard library를 제공합니다. `tempfile` 에는 여러 메소드가 있으나 그 중 자주 사용되는 `mkstemp()`, `TemporaryFile()` 메소드를 살펴봅니다.



### `tempfile.mkstemp()` 

`mkstemp()` 메소드는 아래와 같은 특징을 가지고 있습니다.

- 가장 안전한 방식으로 파일을 생성합니다. (파일을 생성한 사용자만 읽을 수 있다.)
- 사용자가 직접 파일을 삭제해야합니다.
- **닫히더라도 사용자가 직접 파일을 삭제하기 전까지 임시 파일이 보존됩니다.**

```python
import tempfile
import os

def read(path):
    with open(path, 'r') as f:
        while True:
            line = f.readline()
            if not line: break
            print(line)

def write(path, text):
    with open(path, 'w') as f:
        f.write(text)

def main():
    print("!!! Start working with file")
    fd, path = tempfile.mkstemp() # 임시 파일을 생성하고 file descriptor, file path(절대경로)를 반환합니다.
    print(f"@ Temporary file: {path} {os.path.isfile(path)}")

    write(path, "Hello World!")
    read(path)

    os.close(fd)    # 파일을 닫습니다.
    os.unlink(path) # 파일을 삭제합니다.
    print("!!! End working with file")
    print(f"@ Temporary file: {path} {os.path.isfile(path)}")

if __name__ == "__main__":
    main()
```



### `tempfile.TemporaryFile()`

`TemproaryFile()` 메소드는 아래와 같은 특징을 가지고 있습니다.

- `mkstemp()` 와 같은 규칙을 사용하여 파일을 안전하게 생성합니다.
- **파일은 닫히는 즉시 삭제됩니다.**

```python
import tempfile

def main():
    print("!!! Start working with file")

    with tempfile.TemporaryFile(mode='w+t') as f:
        f.write('Hello World!')
        f.seek(0)
        while True:
            line = f.readline()
            if not line: break
            print(line)
    
    print("!!! End working with file")

if __name__ == "__main__":
    main()
```



\>  

두 메소드의 특징을 비교해보았을 때, 임시파일을 여러번 재사용해야하는 경우에 `mkstemp()` 를 유용하게 사용할 수 있을 것 같고, 그 외에는 `TemporaryFile()`을 이용하여 임시파일을 사용자의 개입없이 간단하게 삭제할 수 있을 것 같습니다.

정확하고 자세한 내용은 공식 홈페이지를 참조하는 것이 좋습니다.

- [https://docs.python.org/3/library/tempfile.html](https://docs.python.org/3/library/tempfile.html)



## 결론

임시 파일은 테스트 뿐만 아니라 특정 데이터를 다른 머신으로 전달하기전 버퍼로써 활용되기도 하며, 게시글과 같은 컨텐츠를 임시저장할 때도 사용되는 등 다양한 용도로 활용됩니다. 이러한 여러 활용처에서 tempfile 라이브러리를 활용하여 임시파일을 간단하게 생성하고 삭제할 수 있어야합니다.