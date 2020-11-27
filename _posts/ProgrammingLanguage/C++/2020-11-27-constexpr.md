---
title: constexpr
author: Jay Lee
date: 2020-11-27 00:00:00 +0800
categories: [ProgrammingLanguage, C++]
tags: [ProgrammingLanguage, C++]
image : /assets/img/post/cpp.png
---

## constexpr

### 0. Preface

필자는 constexpr 가 왜 이리도 헷갈리는지 진짜 잠을 설칠 정도였다.
도저히 못참겠어서 이렇게 정리하기로 한다.

constexpr는 Modern cpp 인 c++11 이상에서 지원되고 잇다.
constexpr의 스펙은 STL 버전이 진화하면서 따라 달라지고 있다고 한다.

다양한 강의자료 및 사용처들을 살펴봤는데 다들 사용하는 용도가 다른 것 같아서 그런건지, 이해하고 있는 게 다 다른 것 같다.
constexpr의 원작자의 의도는 분명 무언가가 있었겠지만 지금은 처음과는 많이 다른 것 같다.

다만 모든 사람들이 공통적으로 하는 말은 있다.

"How is it different with const?"

const는 그야말로 constant, 즉 상수이다.
한번 Compile 되면 Runtime 중에 변경이 불가능한 영역의 데이터가 되어 버린다는 뜻이다.

constexpr도 의도는 비슷하다. 프로그래머가 Compile 타임에 변수나 함수가 결정되도록  하는 용도로 사용된다.

하지만 constexpr는 근데 다소 모호하다.
constexpr 로 선언한 변수나 함수는 Compile 중에도 결정될 수 있고 Runtime 중에도 결정 될수 있다. 
[cpp reference c++1x constexpr](https://en.cppreference.com/w/cpp/language/constexpr)

그럼 다음 예제들을 살펴보며 그 모호함에 빠져보기로 하자..

### 1. 일반 예제

``` cpp
#include <iostream>
int fibonacci(int n){
    if (n >= 2) 
        return fibonacci(n-1) + fibonacci(n-2);
    else
        n;
}

int main(){
    std::cout << fibonacci(10) <<'\n';
}
````

위 와 같은 피보나치 수열을 계산하는 코드가 있다고 가정하자.

위 코드는 Compile 후 프로그램을 Run 할 때 해당 값을 계산하여 알 수 있다.

``` sh

$ /usr/bin/time ./fibonacci 
102334155
0.56user 0.00system 0:00.56elapsed 100%CPU (0avgtext+0avgdata 3324maxresident)k
0inputs+0outputs (0major+126minor)pagefaults 0swaps

```

Run 타임에 값이 계산되므로 꽤나 시간이 걸린다.

### 2. Template 

만약 프로그래머가 Runtime 시 효율을 생각한다면 Compile 타임에 피보나치 수열을 계산한 값을 알고 싶다면 어떻게 할까?

먼저 C++ [Template Meta Programming 기법](https://ko.wikipedia.org/wiki/%ED%85%9C%ED%94%8C%EB%A6%BF_%EB%A9%94%ED%83%80%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)을 이용해보자


``` cpp

#include <iostream>

template <int N>
struct fibonacci
{
    static int64_t const value =fibonacci<N-1>::value + fibonacci<N-2>::value;
};
template<>
struct fibonacci<0>
{
    static int64_t const value = 0;
};
template<>
struct fibonacci<1>
{
    static int64_t const value = 1;
};

int main(){
    std::cout << fibonacci<40>::value <<'\n';
}
```

C++ Template 기법을 이용하면 Compile 타임에 그 Compiler 가 내부적으로 Code를 생산하여 작업해준다.

그러므로 위 **fibonacciI<40>::value** 값은 Compile 타임에 정해진다.

``` sh

$ /usr/bin/time ./fibonacci_template 
102334155
0.00user 0.00system 0:00.00elapsed 100%CPU (0avgtext+0avgdata 3388maxresident)k
0inputs+0outputs (0major+125minor)pagefaults 0swaps

```

Compile 타임에 값이 계산되었기 때문에 실행시간이 짧다.

### 3. constexpr

그렇다면 **constexpr** 를 사용하면 어떨까?

``` cpp

#include <iostream>
constexpr int  fibonacci(int n){
    return n>=2 ? fibonacci(n-1) + fibonacci(n-2): n;
}

template<int N>
struct constN{
    constN(){ std::cout << N << '\n';}
};

int main(){
    constN <fibonacci(40)> a; // Compile time
    //std::cout <<fibonacci(40)<<'\n';  //Run time
}

```

``` sh

$ /usr/bin/time ./fibonacci_constexpr 
102334155
0.00user 0.00system 0:00.00elapsed 100%CPU (0avgtext+0avgdata 3412maxresident)k
0inputs+0outputs (0major+125minor)pagefaults 0swaps

```

Compile 타임에 값이 계산되었기 때문에 실행시간이 짧다.

재미있는 것이 **constexpr** 는 두가지 형태 모두 사용가능하다.

위에 같은 코드에 아래 주석을 해제하면 값이 Runtime 에 결정되어 실행시간이 오래걸린다.

``` sh

$ /usr/bin/time ./fibonacci_constexpr 
102334155
0.55user 0.00system 0:00.55elapsed 99%CPU (0avgtext+0avgdata 3376maxresident)k
0inputs+0outputs (0major+125minor)pagefaults 0swaps

```

### 4. Conclusion

constexpr는 위와 같이 Compile 및 Run time에 둘다 사용이 가능하다.
Compile시 constexpr의 요구조건에 맞지 않는다면 바로 Run time 에 계산이 되는 형태이다.

constexpr 를 이용한다면 같은 함수로 Compile time 에 값이 정해져 실행시간에서 효율을 낼 수도 있고, 원한다면  Debugger를 이용하여 Runtime 값을 Break Point에서 찍어볼 수 있다.

결국 필자가 내린 결론은 "constexpr는 프로그래머가 Compile시 값이 결정될 수도 있게 하겠다는 의도를 보여주는 기법" 이다.

