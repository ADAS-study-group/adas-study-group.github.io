---
title: 외부 레이어
layout: post
author: Jo, SeungHyeon
categories: [Blogging, StudySummary]
tags: [Yocto, Layer]
---
## Layer를 이용한 유연성 확보

- Poky는 방대한 양의 Meta-data를 가진다.
  - Machine configuration file,
  - class,
  - 간단한 application
  - 화면을 구성하는 Stack들
  - Framework
- Bitbake는 다양한 곳에서 Meta-data를 Load 할 수 있다.
- 다양한 Meta-data의 집합을 Meta-layer라고 한다.
- Layer 사용의 장점
  - 사용자가 Project에 필요한 Meta-data 집합만 선택할 수 있음
    - 논리적 단위로 Meta-data를 분리할 수 있기 때문
    - 이게 가장 큰 장점
    - Meta-data layer를 사용하면 Code 재사용성이 높아짐
      - 여러 Team, Community, Vendor간의 작업을 공유
        - 다양한 Entity를 같은 Meta-data에서 함께 작업함으로써 Yocto Project Community의 Code Quality를 높일 수 있다.
  - 여러가지 목적에 따라 시스템을 설정할 수 있다.
    - 기능을 활성/비활성화 하거나
    - Architecture별로 최적화하도록 Build flag를 변경하는 것
- 예시
  - Custom project 환경을 만들 때 Recipe와 Configuration file을 변경하거나, Poky layer에 file을 수정하는 대신, 다른 Layer에서 Meta-data를 만들어야 한다.
  - 더 많이 구조화 될수록 다음 Project에서 Layer 재사용성이 높아진다.
  - 이러한 이유로 Poky source code를 보면 여러개의 Layer가 있다.
  - 다음 명령어를 통해 Layer를 확인 할 수 있고, 기본적으로 3개가 존재한다.

    ```terminal
    bitbake-layers show-layers
    ```

    ![COUT: bitbake-layers show-layers]({{ "/assets/img/post/2020-10-06/01_bitbake-layers_show-layers.png" | relative_url }})
    - 예시를 통해 알 수 있는 Layer의 중요한 특징들

        이름 | 보통 meta 접두사로 시작한다
        경로 | 새로운 Layer를 Project에 추가하고 싶을 때, BBPATH에 layer의 위치를 이어 붙여야 한다.
        우선순위 | Bitbake에서 사용하는 값으로, 어떤 Recipe를 이용하고 어떤 .bbappend를 먼저 적용시킬지 결정하는 데 사용

      - 두 개의 같은 Recipe(.bb)를 갖고 있다면, 우선순위가 높은 Recipe를 이용한다.
      - .bbappend file의 경우, 모든 file이 적용되고, 우선순위가 높은 Recipe가 낮은 Recipe보다 먼저 적용된다.
    - Poky는 3개의 각기다른 종류의 Layer로 구조화 되어 있다.
      - meta-layer
        - OpenEmbedded-Core Meta-data 포함
      - Recipe, Class, QEMU Machine 환경설정 file을 갖고있는 SW layer
        - SW layer
        - 어떤 Architecture에서든 이용될 수 있는 Application과 환경설정 file을 갖고 있다.
        - 예시
          - meta-java
            - Java runtime과 SDK 지원
          - meta-qt5
            - Qt5 지원
          - meta-browser
            - Firefox와 Chrome 지원
    - meta-yocto-bsp layer
      - Reference BSP layer
      - Package들을 해당 Machine에 적용하기 위한 Machine 환경설정 file들과 Recipe를 가지고 있다.
      - 예제로 많이 이용되기도 한다.
    - meta-poky layer
      - Distribution layer
      - Poky가 기본으로 사용하는 배포 환경설정 file을 가지고 있다.
      - 기본 배포본은 poky.conf 이고 Test 용으로 많이 이용된다.
        - 제품에 따라 필요한 수정사항을 build/conf/local.conf 에 적용해야할 필요가 있을 수 있다.
          - 개발단계에서 Test목적으로 사용가능하다.
          - local.conf file에 의존해서 Package version, provider, system configuration을 적용하는 것은 바람직하지 않다.
        - 지속가능하고 가장 적합한 방법 (어떤 Build도 재현가능하게 만드는 설정 파일)
          - 배포 file과 배포 layer를 각각 생성
          - 배포 file을 배포 layer에 넣는 것
        - 하나의 배포판 layer가 있으면 모든 배포판에 적용할 수 있고, 새로운 배포판 마다 Layer를 만들 수도 있다.
          - 배포판 layer에서의 환경설정은 build/conf/local.conf를 덮어쓴다.
      - 예제로 많이 이용되기도 한다.

## Layer의 Source code에 대한 고찰

- Layer의 일반적인 Directory 구조

  ```terminal
  meta-*/
    +-- classes/
    +-- conf/
    +-- files/
    +-- lib/
    +-- recipes-*/
    +-- COPYING.MIT
    +-- README  
  ```

  - Layer 이름
    - meta-로 시작하지만 권장되는 규칙일 뿐
    - COPYING.MIT
      - License 와 사용자를 위한 Guide 파일  
    - README
      - 파일에는 사용자가 알아야 할 추가 의존성이나 정보가 있다.
    - classes/
      - Layer에 관련된 class file (.bbclass)을 포함해야 한다. (optional)
    - conf/
      - 설정 file (.conf)을 제공해야 한다. layer.conf file(11장)이 있다. (mandatory)
        - meta-yocto-bsp layer의 conf/ 구조
        ![COUT: meta-yocto-bsp layer conf/]({{ "/assets/img/post/2020-10-06/02_meta-yocto-bsp_layer_conf.png" | relative_url }})
        - meta-poky layer의 conf/ 구조
        ![COUT: meta-poky layer conf/]({{ "/assets/img/post/2020-10-06/03_meta-yocto-poky.png" | relative_url }})
  - recipes-*/
    - 범주에 따라 분류 Recipe들의 집합으로, Recipe 범주에 속하는 Directory와 그 내부에 .bb와 .bbappend로 끝나는 실제 Recipe가 있다.
    - 예시) recipes-core, recipes-bsp, recipes-multimedia, recipes-kernel
    ![COUT: recipes-multimedia]({{ "/assets/img/post/2020-10-06/04_recipes-multimedia.png" | relative_url }})

## Meta-layer 추가

- Yocto Project, OpenEmbedded, Community, Vendor들에서 제공되는 수백 개의 Meta-layer가 있고, 개발할 때 필요한 부분은 Project source directory에서 수동으로 받아와야 한다.
  - <http://git.yoctoproject.org>
  - <http://layers.openembedded.org>
- 수동으로 build/conf/bblayers.conf를 수정하는 대신 아래와 같이, bitbake-layers 도구를 이용해 추가 할 수 있다.
  - Adding layers to Poky/build/bblayers.conf file (meta-openembedded/meta-oe 는 bitbake database에 포함되어 있음)

  ```terminal
  bitbake-layers add-layer ../poky/meta-mylayer
  bitbake-layers add-layer ../poky/meta-openembedded/meta-oe
  ```

## Yocto Project layer eco-system

- Layer를 쉽게 만들 수 있기 때문에 수많은 Layer가 이미 만들어져 사용가능하다. 이를 쉽게 찾도록 하기위해 OpenEmbedded Community는 대부분의 Layer을 찾을 수 있는 Index를 개발했다.
  - RasberryPi3를 사용한다고 가정하면,
    - <http://layers.openembedded.org> 의 Machines Tab에서 찾으면 다음 화면을 볼 수 있다.
      ![COUT: Machines Tab]({{ "/assets/img/post/2020-10-06/05_Machines_Tab.png" | relative_url }})
