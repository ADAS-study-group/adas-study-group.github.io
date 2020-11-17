---
title: Study summary 기록 2020-09-15
author: Leehyunho94
date: 2020-09-15 14:10:00 +0800
categories: [Blogging, StudySummary]
tags: [Writing, Yocto]
---
## 07 비트베이크 메타데이터 참고 자료

### 메타데이터 사용

메타 데이터는 다음과 같이 3개의 주요 영역으로 분류

Configuration(.conf) : 환경설정 파일은 전역으로 영향을 미치는 파일로 classes와 레시피의 동작을 위한 정보를 제공.

Classes(.bbclass ) : 전체 시스템에서 이용할 수 있고 쉬운 유지 보수와 코드 중복을 피하기 위해 레시피에서 상속

Recipes(.bb .bbappend) : 테스크가 실행되도록 정의하고 비트베이크가 필요한 태스크 체인을 생성할 수 있게 필요한 정보를 제공, 가장 많이 사용하는 메타데이터의 종류이고, 모든작업에 필요

- 레시피와  클래스는 파이썬과 셀스크립트 코드 두개로 작성가능


### 메타데이터 작업

비트 베이크의 환경 변수 옵션을 사용해서 각 변수의 값을 확인가능

- $: bitbake -e <recipe> | grep <variable>

#### 메타데이터 변수

- #####기본 변수 설정

 BasicVar = "helloworld" 

위 와 같이 선언하면 변수 BasicVar의 값은 helloworld 

--------------------

- ##### 변수 확장

 A = "aval"

 B = "pre${A}post"

A 의 값은 aval , B의값은 preavalpost

이런식으로 결과값이 나오고 주의할 점은 실제 사용될 시점에 변수 확장이 실행됨의로 주의

 A = "aval"

 B = "pre${A}post"

 A = "change"

A 의 값은 change , B 의 값은 prechangepost

위 예시는 비트베이크 결과 값이 나중에 적용되는 것을 보여줌

----------------------------------------------------

- ##### 변수 기본 값 설정 , ?= 사용

?= 는 기본적으로 이미 값이 설정되어있지 않는 변수에 사용하면 새로운 값을 할당

 A ?= "aval"

이전 값 설정을 갖고 있지 않음으로 A 의 값은 aval

하나의 변수에 여러 ?= 할당이 있다면 첫번째로 할당된 것이 사용

 A ?= "aval" 

 A ?= "change"

A의 값은? aval이다.

A에 만약 이전 값이 설정되어있따면 이전 값이 사용됨

 A = "before"

 A ?= "aval"

 A ?= "after"

A의 값은? before 이다.

----------------------------------------

- ##### ??= 를 사용한 기본 값 설정

위 ?= 와 똑같고 차이점은 파싱 절차가 끝날때까지 적용되지 않는다는점

 A ??= "aval"

 A ??= "change"

A의 값은 change , ?=의 경우에는 aval이었음

여기서 ?=과 마찬가지로

A에 만약 이전 값이 설정되어있따면 이전 값이 사용됨

 A = "before"

 A ??= "aval"

 A ??= "after"

A의 값은? before 이다.

----------------------------------

- ##### := 즉시 변수 적용

변수를 바로 적용해야 할 필요성이 있을 때 사용
 A = "now"
  
 B := "${A} TEST"
  
 C = "${A} TEST"
  
 A = "after"
        
B의 값은? now TEST , C의 값은? after TEST 
이럴경우, B에는 "now TEST"가, C에는 "after TEST"가 들어가 있음

----------------------------------------
  
- ##### 앞뒤 추가 += , =+

String + 연산 , += 와 =+ 2가지를 제공하고 차이점이 있음

 A1 = "ABC"
  
 A2 = "ABC"
  
 A1 += "DEF"
  
 A2 =+ "DEF"
  
A1의 값은 ABC DEF , A2의 값은 DEF ABC
  
결과값을 자세히 보면 각각의 호출 사이에 여분의 공백을 포함 , 이러한 공백을 없애려면 .= 와 =. 두가지를 사용
  
 A1 = "ABC"
 
 A2 = "ABC"
 
 A1 .= "DEF"
 
 A2 =. "DEF"
 
A1의 값은 ABCDEF , A2의 값은 DEFABC

+=,=+ 는 리스트에 아이템을 추가하기 위해 사용

.=,=. 는 문자열을 연결할 때 사용

-------------------------------------------------

- ##### 오버라이드 문법 연산자

문자열을 연결하는 방법은 위 방법말고 오버라이드 방식 문법을 사용한 append와 prepend 연산자도 사용할 수 있다.

 A = "ABC"
 
 A_append = "DEF"
 
 B = "ABC"
 
 B_prepend = "DEF"
 
A의 값은 ABCDEF , B의 값은 DEFABC

여기서 위 연산자들은 공백을 추가하지않는다.

But , .=,=.와 같아보이지만 미묘한 차이가 존재한다.

예를들면, 확장 연산자를 사용하면 변수의 확장은 연산이 실행되기 전에 진행된다.

 A ?= "ABC"

 A .= "more"

 B ?= "ABC"

 B_prepend = "more"

A의 값은 more , B의 값은 moreABC

------------------------------------------------

- ##### 변수의 값을 제거하는 연산자 , remove

remove 연산자는 공백으로 구분된 문자열 목록을 변수 값으로 갖기 때문에 목록에 있는 하나 이상의 문자열을 제거

 A = "이 현 호"
 
 A_remove = "이 호"

A의 값은 현

-----------------------------------

- ##### 조건을 가진 메타데이터 설정

비트베이크는 조건을 가진 메타데이터 설정을 쉬운 방법으로 제공

OVERRIDES를 호출해 메커니즘을 만듬

 OVERRIDES = "as:bs:cs"

 TEST = "defaultval"

 TEST_bs = "bsval"

 TEST_os = "osval"

 TEST의 값은 bsval
 
-------------------------------

- ##### 조건적 추가

조건 변수에 대한 Appending(뒤에 이어 붙이기) / Depending(앞에 이어 붙이기)

 DEPENDS = "A B"

 OVERIDES= "machine:local"

 DEPENDS_append_machine = "C"
 
DEPENDS의 값은 A B C

--------------------------------------

#### 파일 포함
    
비트베이크는 파일을 포함하기 위한 2가지 방법을 제공

1. include

비트베이크는 그 위치에 있는 파일을 추가하는 것을 시도하고. 추가 라인에서 특정 경로가 상대 경로이면 비트베이크는 bbpath내에서 찾을 수 있는 첫번째 것을 포함시킨다.

2. Required 

required 파일을 찾을 수 없으면 ParseError발생. 이외에는 include와 동일

--------------------------

#### 파이썬 변수 설정

비트베이크는 변수 설정 시 쉽게 파이썬 코드를 사용 가능

VARIABLE = "${<python-command>}"

ex)

DATE = DATE = "${(@time.strftime(1 %Y%m%d1 ,time.gmtime())}"

DATE의 값은 오늘의 날짜.

#### 실행 가능한 메타데이터 정의

메타데이터 레시피와 클래스 파일들은 예시와 같은 셀 스크립트를 이용해 만들 수 있음.

ex)

* 셀 스크립트

do_mytask () {\
echo "Hello, world!"\
}

* 파이썬 코드임을 표시해서 비트베이크가 파이썬 코드임을 알고 이에 맞게 실행하도록 python 표시

python do_prlntdate () { \
import time\
print time.strftlme('%Y%m%d', time.gmtime())\
}

-------------------------------------------------

### 전역 네임스페이스에서 파이썬 함수 정의

변수나 또 다른 사용을 위한 값을 만들어내기 위해 파이썬 함수를 사용할 필요가 있다.

ex)

def get_depends(d):\
  if d.getVar('SOMECONDITION'):\
      return "dependencywlthcond \
  else:\
      return "dependency"\
SOMECONDITION = T\
DEPENDS = "${@get_depends(d)}"

DEPENDS의 값은? dependencywlthcond

모든 메타데이터 사이에 규약은 비트베이크의 데이터베이스를 가리키는 데 d 변수를 사용 한다. 그리고 d 변수는 함수의 마지막 파라미터로도 자주 사용한다. 앞의 예에서 데이터베이스는 SOMECONDITION 변수 값을 데이터베이스에 요청하고 그 것에 의존성 있는 값을 리턴한다.
이 예제에서 DEPENDS의 결과는 dependencywithcond다.

------------------------------------------------------

### 상속 시스템

inherit 지시어는 레시피(.bb)에서 필요한 기능의 클래스를 사용하는 방법이다. 이것 은 상속의 기본적인 형식이다. 예를 들어 autoconf나 automake에 관련된 작업을 쉽 게 추상화할 수 있고，그것을 레시피에서 사용하기 위해 .bbclass에 넣을 수 있다. 주어진 .bbclass는 BBPATH에서 상속받은 파일 이름인 classes/filename.bbclass로 찾 을 수 있다. autoconf 또는 automake를 사용하는 레시피에서 다음과 같이 사용할 수 있다.
   
   
ex)
Inherit autotools

----------------------------
