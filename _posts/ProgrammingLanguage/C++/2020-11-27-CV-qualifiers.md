---
title: CV-qualifiers
author: Jay Lee
date: 2020-11-27 00:00:00 +0800
categories: [ProgrammingLanguage, C++]
tags: [ProgrammingLanguage, C++]
image : /assets/img/post/cpp.png
---


## CV-qualifiers

### 0. Preface

const 는 Contatnt 즉 상수를 표현하기 위한 기법이고 volatile 은 휘발성(?) 타입이라는 것을 표현하기위한 기법이다.
STL에서 cont는 volatile 과 함께 cv qualifiers 로 정의한다. [Link:](https://en.cppreference.com/w/cpp/language/cv)

### 1. Notation

C++ 에서 cv qualifiers 와 같은 type 한정자(?)는 type의 왼쪽 및 오른쪽 양쪽 다에 올 수 있다.

예를 들어보자

``` cpp
    const int i = 100;
    int const i = 100;
```

여러 프로그래밍 언어를 하는 독자들은 다소 헷갈릴 여지가 있는 것이, const 키워드는 보통 타입의 왼쪽에 표기되기 때문이다.

C와 C++ 에서는 위 코드에서 둘다 맞고 동일한 표현이다.

> 다만 const는 오른쪽에서 왼쪽으로 수식해야하고, 왼쪽에 수식대상이 없을때만 왼쪽에서 오른쪽으로 수식한다.

왜 이런 기법을 만들었을까? 

#### 가독성 측면의 강점

위 예제에서는 동일했지만 C/C++에서 Pointer와 사용시엔 다소 다르다.

``` cpp
    const char *const s = "aaa";
    char const *const s = "aaa";
```

위 두 표현도 둘다 같은 표현이지만 const 표현이 두번 혼재되어있고, 
둘중 윗 예제에선 하나의 const 는 왼쪽에서 오른쪽을 하나의 const는 오른쪽에서 왼쪽을 수식한다.

때문에 식이 복잡하다면 항상 오른쪽에서 왼쪽으로 const 를 표기하도록 하는 Convention을 지키는 것이 가독성에 유리할 것이다.
