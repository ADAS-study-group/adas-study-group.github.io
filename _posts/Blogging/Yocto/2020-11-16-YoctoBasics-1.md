---
title: "Yocto 빌드 시스템"
author : Jay Lee
date: 2020-11-16 10:00:00 +0800
categories: [Blogging, Yocto]
tags: [Yocto, Bitbake, OpenEmbedded, Linux]
image : /assets/img/post/2020-11-16/yocto.png

---

## 1. 들어가며

본 문서에서는 필자가 Yocto Project의 빌드시스템 기반으로 빌드 및 배포하기 위해  스터디한 부분들 중 중요 개념들을 정리한다.

Yocto Project 의 공식홈페이지에 Mega Manual 이라고 써있을 정도로 무지막지한 메뉴얼 양을 자랑하기 때문에 이를 전부 다 깨우치는 것은 필자의 레벨에서는 의미가 크게 없다고 생각하기 때문에, 중요 부분들만 그림을 섞어 설명하도록 한다.

친절하게 링크를 걸어둘터이니 볼테면 봐보시라 --> [mega-manual](https://www.yoctoproject.org/docs/current/mega-manual/mega-manual.html)

구글링을 통한 필자의 주관적 분석이 많이 섞여있으므로 더 객관적인 분석을 원한다면 위 링크를 참조 바란다.

## 2. 재미로 보는 역사

Yocto Project는  **OpenEmbedded Project** 로 부터  그 근원이 있다. 

<center>[그림 설명] 고인물의 상징  [출처 : 구글 이미지검색]</center>
![Desktop View]({{ "/assets/img/post/2020-11-16/oe.png" | relative_url }})


OpenEmbedded  Project는 필자에게 전자사전으로 익숙한 **Sharp** 가 **Open Source license**에 따라 해당 제품의 ROM Image를 공개하는데부터 시작한다고 전해진다. 

(라떼는 잘나갔읍니다..)

<center>[그림 설명] ~~90년대 생은 잘 모를수도 있다~~  [출처 : 구글 이미지검색]</center>
![Desktop View]({{ "/assets/img/post/2020-11-16/sharp.jpeg" | relative_url }})

이를 기반으로 2002년 OpenZaurus Project가 시작되었다. (Zaurus는 PDA이다)

<center>[그림설명] OpenZaurus와 Sharp사의 Zaurus PDA [출처: 구글 이미지검색]</center>

![Desktop View]({{ "/assets/img/post/2020-11-16/Openzaurus-logo.png" | relative_url }})
![Desktop View]({{ "/assets/img/post/2020-11-16/Sharp_Zaurus.jpg" | relative_url }})

시간이 발전함에 따라 우리에게 Ubuntu로 친숙한 Debian계열 기반의 패키지 관리 및 빌드방식을 채용한 OpenEmbedded Project 로 통합되어 발전되어왔다 한다.

OpenEmbedded 는 shell이나 python 같은 script로 작성된 빌드 툴인 **Bitbake** 와 그의 빌드대상 명세인** metadata(recipe)**로 구분된다.

(Bitbake 도 나중엔 너무 커져 2004년부터는 make 처럼 별도의 프로젝트로 분기되었다)

OpenEmbedded Project는 전세계 개발자들에게 큰 인기를 얻어 만개에 가까운 recipe와 수백개의 machine를 지원하게 되었고, 그 성장세 만큼 관리가 어려워 지게 되어 이를 개선하기위한 여러 시도들이 시작된다.



그 중 하나가 Embedded 스타트업 OpenedHand 가 2003년 시작한 Poky Linux Project 인데, 이 프로젝트에서는 수백개의 특정 필수 recipe들만들 선별하였고, QEMU 를 통한 가상환경 빌드 및 SDK 빌드 지원을 해주면서 급 인기를 끌기 시작했다.~~(OpenedHand 는 2008년 Intel에 M&A 에 됐다. 아무리 찾아봐도 얼마에 팔렸는지는.. )~~

<center>[그림설명] Intel 의 스타트업 OpenedHand 인수홍보. 왠지모르게 행복해보인다.</center>
![Desktop View]({{ "/assets/img/post/2020-11-16/openedhand.png" | relative_url }})
![Desktop View]({{ "/assets/img/post/2020-11-16/oh.jpg" | relative_url }})


시간은 흘러 2010년 Linux Foundation WG이 **Yocto Project** 를 발표한다. 
Poky Linux 기반으로 embedded linux 배포판을 만드는 것이 프로젝트의 핵심이다.

2011년 부터는 너무 커버린 OpenEmbedded Project를 **OpenEmbedded-Core(OE-Core)**로 **Poky linux** 에서 분리하였다. (이전 OpenEmbedded 는 **OE-Classic**으로 불리운다)

OE-Core 는 arm, x86 와 같은 주요 아키텍쳐지원에 그 중점을 두고, QEMU 지원에, X window에서 구동하는 Sato 기반 GUI 테스트 툴까지가 지원하는 범주이다.

<center>[출처: Yocto Project Homepage]</center>
![Desktop View]({{ "/assets/img/post/2020-11-16/yoctolayer.png" | relative_url }})

이렇게 오랜기간 덩치를 줄이고 쪼개고를 반복하여 위와같이 지금의 Layered Architecture를 가지게 되었고, 기존의 Push모델이 아닌 Pull Model로서 프로젝트가 발산할 가능성을 배제할 수 있게 되었다한다.


## 3. Cross-build 개요

자 위 역사부분에서 그 흐름을 이해했고 간단한 임베디드 개발이나 크로스 빌드를 해본 독자라면 3장 부터 이해가 빠를 것이다.

위 역사에서 말했듯이 Yocto Project는 Poky Linux 기반으로 빌드 툴로서 Bitbake와 그 명세로 metadata(recipe)를 사용하고, Bitbake는 OpenEmbedded프로젝트의 핵심 기능이다.


그러므로 우리는 이 Bitbake 에 대해 집중 분석할 필요가 있다.

하지만...

오랜 역사를 자랑하는 Bitbake의 모든 것을 알기엔 무리가 있고.. 
간략하게 도식화된 그림들을 통해 빌드하기 위해 필요한 중요 기능들만을 알아보기로하자.

### 3.1 유저 설정

역시 가장 처음은 역시 빌드환경 구축이겠다. 

크로스 빌드를 해본 사람이라면 이 부분이 가장 귀찮고 시간도 많이 소비하지만, 가장 중요한 영역이라는 것을 안다.

아래 그림을 보자


![Desktop View]({{ "/assets/img/post/2020-11-16/bitbake1.png" | relative_url }})
<center>[출처: https://www.yoctoproject.org/docs/3.1/overview-manual]</center>

Poky Project에서는 이 부분을 자동화 가능하도록 스크립트를 제공한다. 그게 바로 oe-init-build-env이다.

스크립트를 실행하면 Build Directory가 생성되고 여기서 모든 빌드 관련 작업이 이루어진다.

생성된 Build directory에 conf directory에는 **User Configuration**을 첨삭가능 하게 해두었고, 이는 bitbake command line 으로도 수정이 가능하다.

첨삭하는 정보들은 먼저 방대한 metadata(recipe)들중 어떤것을 해당 빌드시스템에서 사용할 것인지와 타겟머신 설정, 빌드를 위해 필요한 Package들의 Download 경로, Cache 경로 등이 있다.

### 3.2 소스 준비

![Desktop View]({{ "/assets/img/post/2020-11-16/source-fetching.png" | relative_url }})
[출처: https://www.yoctoproject.org/docs/3.1/overview-manual]

 기반이 다져졌으면 이제 우리만의 Software를 가져다가 Yocto Project 구조에 붙여야한다.

이 부분을 행하는 recipe속 함수는 do_fetch와 do_unpack이다. 이 두 함수를 실제 원본 소스를 Build Directory 내에 Working Directory를 생성하여 그 안에 복사한다.

이 방법 같은 Source로 부터 다양한 Architecture와 OS를 지원하기 위해 고안된 구조이다.

### 3.3 설정 & 컴파일 & 스테이징

![Desktop View]({{ "/assets/img/post/2020-11-16/configuration-compile-autoreconf.png" | relative_url }})


[출처: https://www.yoctoproject.org/docs/3.1/overview-manual]

Software의 원본 소스를 가져오는 작업까지 마쳤다면 그 다음 과정은 컴파일과 설치 과정이다.

이 과정은 독자가 CMake나 Autotool 같은 빌드 툴을 안다면 좀 더 친숙할 것이다.

먼저 **do_prepare_recipe_sysroot** 함수가 크로스 빌드를 위한 두개의 sysroot를 Working Directory에 위치시킨다. (target **sysroot** 와 **sysroot-native** 두가지)

**do_configure** 를 통하여 해당 **원본 소스(S)**에서 Compile을 위해 필요로 하는 빌드 설정 파일들을 B**uild Directory(B)**에추출한다(일반적 cmake의 과정에 해당한다).

**do_compile** 과정은 **Build Directory**에서 컴파일을 진행한다(일반적인 make과정에 해당한다).

**do_install** 과정은 이렇게 컴파일된 파일들을 설치 **대상 목적지(D)**에 위치시킨다.

### 3.4 패키지 분류


![Desktop View]({{ "/assets/img/post/2020-11-16/analysis-for-package-splitting.png" | relative_url }})

[출처: https://www.yoctoproject.org/docs/3.1/overview-manual]

소스들을 컴파일 및 인스톨까지 잘 마무리 됐다면 이제 이를 어떻게 포장해서 배포할지가 남았다. 

Yocto Project에서는 3가지 형태의 배포 형태를 지원하는데 이는 **rpm** ,**deb**,**ipk**  이다.

**do_package, do_packagedata** 두 함수는 설치 목적지인 D에 있는 파일들을 쪼개고 분류하여 포장한다.

Package들을 쪼개는 것은 우리가 Ubuntu 에서 python을 apt 패키지로 설치할때  python, python-dev, python-3.6 등 다양한 방식으로 존재하는것을 빗대어 이해하면 좋다.

### 3.5 이미지 생성

![Desktop View]({{ "/assets/img/post/2020-11-16/image-generation.png" | relative_url }})

[출처: https://www.yoctoproject.org/docs/3.1/overview-manual]

패키지들도 잘 만들어졌다면 우린 이제 이를 Bitbake를 이용하여 이미지의 rootfs (Root file system)에 넣을 수 있다.

먼저 do_rootfs 함수는 위 과정에서 만들어진 패키지를 설치한 Image의 Rootfs을 생성한다.

이 과정 안에는 몇몇 중요 변수들이 있는데 매우 중요하기 때문에 짚고 넘어가도록 하자.

* IMAGE_INSTALL : 우리가 만든 Package모음(Package Feeds area)에서 이미지에  포함시킬 패키지를 리스트업 한다
* PACKAGE_EXCLUDE : 설치되지 말아야할 것들을 리스트업 한다
* PACKAGE_CLASSES : 사용할 패키지의 종류(rpm, deb, ipk)를 선택한다
* PACKAGE_INSTALL : 이미지에 설치될 최종 패키지 리스트이다

패키지 설치를 완료하고 나서는 일종의 후처리 작업(Post-process) 을 할 수 있다.

후 처리 작업에서는 manifest file을 만들어내기도하고 특정 스크립트들을 돌리는데, 목적은 대부분이 테스트 용도이다.

Manifest file은 Poky에서 지원하는 Qemu와 같은 가상환경 뿐아니라 실제 Target 환경에서 테스트 자동화를 위해 쓰인다. (자세한 사항은 testimage*.bbclasss나 testsdk.class 참조)

배포를 위한 V-cycle 중 통합테스트(Integration Test)나 더 나아가 시스템테스트(System Test) 구축에 유용할 것이다.

후처리 작업도 끝났다면 이제 우리는 드디어 Target에 올릴 이미지를 생성할 준비가 되었다.

do_image 함수가 그 역할을 하는데, File System(ext4, fat32 등)마다 다른식으로 할수 있도록 내부적으로 do_image_* 를 제공한다.

### 3.6 SDK 생성

<center> [설명: 데브옵스의 영역???..???? 전부다인가] [출처: 패스트캠퍼스]</center>
![Desktop View]({{ "/assets/img/post/2020-11-16/devops.png" | relative_url }})


그냥 당연히 되야한다고 생각하는 이들이 대부분일 것이고, 실제로 위 언급한 대부분의 과정은 DevOps의 영역이다.

기능 개발자에게는 크로스 컴파일를 위해 이미지 생성까지 한다는 것은 엄청난 시간 낭비고 자원 낭비이므로 DevOps는 SDK형태로 그들이 쉽게 컴파일 할 수 있는 환경을 만들어 줘야한다.

이를 구축하는 것은 매우 귀찮은 일이지만 생산성 측면에서 생각해보면 필수적이다. 때문에 위 역사에서 언급했듯이 Poky Linux 프로젝트에서도 이런점을 고려하여 기능으로 제공한다.

![Desktop View]({{ "/assets/img/post/2020-11-16/sdk-generation.png" | relative_url }})

[출처: https://www.yoctoproject.org/docs/3.1/overview-manual]



상위 과정들에서 패키지화가 잘되었다고 가정하자.

Yocto Project를 이용한다면 이러한 패키지들을 do_populate_sdk 혹은 do_populate_sdk_ext를 이용하여 SDK로 쉽게 만들 수 있다.

SDK설치파일은 통상적으로 위 그림과 같이 /build/tmp/depoly/ *.sh 로 만들어 지고, 

해당 파일만 있다면 개발자는  Cross-build 환경을 쉽게 가져갈 수 있다.

![Desktop View]({{ "/assets/img/post/2020-11-16/cross-development-toolchains.png" | relative_url }})


### 4. 마무리하며

2020년 말 기준으로 자동차의 시스템 플랫폼은 레벨은 자동차 OEM들 뿐아니라 전세계 IT공룡들, 전자, 반도체 회사들 이 득달같이 달려드는 뜨거운 감자이다.

차량뿐아니라 IoT의 세상도 Linux 기반의 Kernel 을 가진다는 것을 감안하면 Yocto 와 같은 Open Source Project기반 통합 빌드환경을 제공하는 에코 시스템은 더욱어 각광 받을 것으로 보인다.

실제로 Linkedin으로 살펴보면 VW Group, BMW Group, Hyundai-Motors Group, Toyota Group 뿐아니라 차량반도체 칩제소사들을 포함한 Amazon, Facebook 등 IT회사들에서도 Yocto Project에 익숙한 Senior 급의 Build Architect 혹은 Build Engineer 를 뽑는다.

아무래도 Yocto 프로젝트가 다루는 영역이 Kernel부터 Application 까지 묶어서 보기 때문에 능숙히 다룰줄 알아야하는 컴퓨터 언어만해도 최소 3개이상이어야 하기 때문에 엄청난 양의 스터디를 요구한다.

본인 역시 Yocto 빌드 시스템을 공부하며 참 힘들었는데 필자처럼 맨땅에서 공부하는 이들에게 이 기고글이 많은 도움이 되었길 바라며 글을 마무리 한다.

