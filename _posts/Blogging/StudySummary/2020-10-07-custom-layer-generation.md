---
title: 사용자 Layer 생성
layout: post
author: Jo, SeungHyeon
categories: [Blogging, StudySummary]
tags: [Yocto, Layer]
---
## 새로운 Layer 추가

1. 사전에 OpenEmbedded Meta-data Index <http://layers.openembedded.org> 에서 이미 이용 가능한 비슷한 Layer가 있는지 확인하는 것이 좋다.
2. 필요한 Layer를 찾을 수 없으면 다음 단계로 Directory를 생성한다. meta- 로 시작는 것을 추천한다.
3. Layer의 환경설정 file은 모든 Layer에서 필요하고, conf/layer.conf file로 되어 있다. 이 file은 Text editor를 사용해 수작업으로 만들 수도 있지만, 다음과 같이 Poky에서 제공하는 Script로 만들 수 있다.

   - Creating Layer

   ```terminal
   cd ~/poky
   source oe-init-build-env
    bitbake-layers create-layer ../poky/meta-mylayer
   ```

   ![COUT: create-layer]({{ "/assets/img/post/2020-10-07/01_create-layer.png" | relative_url }})
   - Script로 Layer를 생성하기 위해 Layer 우선순위 값 입력하고 샘플 내용 관련된 질문에 답을 입력한다.
   - 예시
   ![meta-mylayer]({{ "/assets/img/post/2020-10-07/02_meta-mylayer.png" | relative_url }})
   ![meta-mylayer]({{ "/assets/img/post/2020-10-07/03_meta-newlayer.png" | relative_url }})
4. 사용자 Layer를 작업하기 위해 다른 Layer를 추가 할 필요가 있을 때, 사용하는 변수
   - LAYERVERSION
      - 하나의 숫자로 Layer의 Version을 표시할 때, 사용하는 선택적 변수다.
      - 이 변수는 LAYERDEPENDS 변수에서 Layer 특정 Version에 의존성을 걸기 위해 사용하고, LAYERVERSION_newlayer = "1" 와 같이 Layer의 이름에 접미사를 붙여 사용한다.
   - LAYERDEPENDS
      - 공백으로 분리
      - Recipe의 의존성을 걸기 위해 Layer목록을 가진다.
      - 선택적으로 otherlayer:2 와 같이 Layer의 이름  끝에 콜론을 추가해 의존성과 같이 Layer Version을 적시 할 수도 있다.
      - 이 변수도 특정 Layer의 이름과 함께, LAYERDEPENDS_newlayer = "otherlayer" 와 같이 Layer의 이름에 접미사를 붙여 사용한다.
   - 의존성을 만족하지 않거나 Version 정보가 맞지 않으면 Error가 발생

## Layer에서 Metadata 추가(Layer 구조의 기반을 만들고 확장하는 방법)

- Layer을 사용하는 목적은 bitbake database에 meta-data를 추가하거나 변경하기 위함
- 많이 추가하는 것은 Application, Library, 혹은 Service server와 같이 Project와 관련된 것
  - 새로운 특징을 추가 하는 것보다 SSH서버를 위한 초기 Network 값 또는 Booting 화면의 예와 같이 사용자가 필요로 하는 기존 설정을 변경하는 것이 더 자주 이뤄 진다.
- 기존의 것을 바꾸기 위해 새로운 Layer, Recipe, Image, .bbappend file 들에서 Meta-data file 여러 종류를 포함할 수 있다.
  - 앞에서 사용한 Script는 두 개의 예제 file과 새로운 Layer를 만들 때 사용됐다. 첫 번째 example_0.1.bb는 Recipe의 예제다. Example_0.1.bbappend는 example_0.1.bb를 포함하고 내용을 수정할 때 사용하는 bbappend의 예제이다.
- meta-yocto-bsp 또는 meta-yocto에 따른 여러 .bbappend 예제가 있고, 많이 사용하는 것들 추후 정리 (12장)

## Image 생성

- Image file은 정해진 방식에서 목적을 가지고 설정된 Package의 집합
- 기존 Image에 필요한 Package를 추가하고 환경설정을 덮어씌운 새로운 Image를 만들 수 있다. 또한 처음부터 새로운 Image를 만들 수도 있다.
  - Image를 필요에 맞게 만들고 작은 수정 했 때 그 코드를 재사용하는 것이 매우 편리
  - 이는 코드 유지 보수를 쉽게 만들고 기능상 차이점은 강조됨
  - 예시
    - core-image-sato image에서 application을 포함하기 원하고 하나의 Image 특징을 제거하기 원한다면 다음 code를 recipes-my/images/my-image-sato.bb에 추가해 image를 생성할 수 있다.

      ```recipe
      require recipes-sato/image/core-image-sato.bb
      IMAGE_FEATURES_remove = "splash"
      CORE_IMAGE_EXTRA_INSTALL += "myapp"
      ```

  - 처음부터 Image를 생성할 필요가 있을 때, 쉽고 유용하게 사용되는 Image 특징의 집합을 제공하는 core-image class를 사용해 그런 작업을 빠르게 할 수 있다. 예를 들면 recipe-my/images/my-image-nano.bb에서 Image는 다음과 같이 구성됨

    ```recipe
    inherit core-image
    IMAGE_FEATURES += "ssh-server-openssh splash"
    CORE_IMAGE_EXTRA_INSTALL += "nano"
    ```

  - Append 연산자(+=)는 새로운 IMAGE_FEATURE 변수가 build/conf/local.conf에 추가될 수 있게 보장하기 위해 사용됨
  - core-image class를 상속받을 때 이미지 생성과 IMAGE_FEATURES 변수를 쉽게 추가하고 코드 중복을 피하게 CORE_IMAGE_EXTRA_INSTALL 변수를 사용해 Image에 추가 Package를 포함한다.
  - IMAGE_INSTALL 변수는 CORE_IMAGE_EXTRA_INSTALL을 추가하고 IMAGE_FEATURES에 관련된 Package를 Root file system에 생성한다.
  - IMAGE_FEATURES 추가가능 features

    allow-empty-password | Dropbear와 OpenSSH에서 빈 호를 사용해 root 계정으로 로그인 할 수 있게 허용한다.
    dpg-pkgs | Image에 설치된 모든 Package의 Debug symbol package를 설치한다.
    debug-tweaks | 개발에 필요한 Package를 설치해 Image를 만든다.
    dev-pkgs | Image에 설치된 모든 Package의 개발 Package (Header와 추가 Library)를 설치한다.
    eclipse-debug | Eclipse IDE에서 원격 Debugging을 지원하게 한다.
    empty-root-password | root 암호를 빈 문자열로 설정한다.
    Hwcodecs | HW 가속되는 Codec을 설치한다.
    nfs-server | NFS server를 설치한다.
    package-management | Package 관리도구를 설치하고 Package 관리 Database를 가진다.
    perf | perf, systemtap, LTTng 같은 Profiling 도구를 설치한다.
    post-install-logging | Target system에서 image를 처음으로 booting할 때 post-install script 실행을 /var/log/postinstall.log file에 저장한다.
    ptest-pkgs | ptest 가 활성화된 모든 Recipe의 ptest Package를 설치한다.
    read-only-rootfs | Root file system이 읽기만 가능하게 Image를 만든다.
    splash | Booting하는 동안 Splash 화면이 보이게 한다.
    ssh-server-dropbear | Dropbear SSH server 를 설치한다.
    ssh-server-openssh | Dropbear보다 많은 기능을 가진 OpenSSH server를 설치한다. 
    staticdev-pkgs | Image에서 설치된 모든 Package의 정적 개발 Package (즉, *.a file들을 포함한 static library)를 설치한다.
    tools-debug | strace와 gdb 같은 debugging도구를 설치한다.
    tools-sdk | Device에서 실행하는 전체 SDK를 설치한다.
    Tools-testapps | Device test 도구를 설치한다. (e.g. touch screen debugging)
    x11 | X server를 설치한다.
    x11-base | 최소 환경을 가진 X server를 설치한다.
    x11-sato | OpenedHand Sato 환경을 설치한다.

## Package Recipe 추가

- Package Recipe는 BitBake가 Application, Kernel module, 또는 Project 에서 제공하는 SW를 Download, 압축풀기, Compile, 설치하는 방법을 제시한다.
- Poky는 Autotools, Cmake, QMake를 기반으로 하는 Project로 가장 흔한 개발 도구를 추상화한 여러 Class를 가지고 있다.
  - Poky에 있는 Class의 목록은 <http://www.yoctoproject.org/docs/2.4/ref-manual/ref-manual.html> 에서 볼 수 있다.
- Compile과 설치 작업을 하는 간단한 Recipe 예제는 다음과 같다.
  ![recipe-example]({{ "/assets/img/post/2020-10-07/04_recipe-example.png" | relative_url }})
  ![recipe-example-from-book]({{ "/assets/img/post/2020-10-07/05_recipe-example-from-book.png" | relative_url }})
  - do_compile과 do_install code 영역은 build와 결과 binary를 ${D}로 언급된 Target directory에 설치 하기 위한 shell script 명령어를 사용한다.
  - Autotools기반 Project의 경우, 다음과 같이 poky/meta/recipecore/dbus-wait/dbus-wait_git.bb의 일부 축약된 Recipe를 보면 알 수 있듯이 autotools Class를 사용해 많은 Code 중복을 피할 수 있다.

    ```recipe
    DESCRIPTION = "A simple tool to wait for a specific signal over DBus"
    ...
    inherit autotools
    ```

  - Class를 상속받는 단순 행위는 다음 작업을 하기 위해 요구되는 모든 Code를 제공한다.
    - Configure script code와 결과물 update
    - libtool script update
    - Configure script 실행
    - make 실행
    - make install 실행
  - 이와 같은 개념은 CMake와 QMake의 경우와 같이 다른 Build 도구에도 적용됨
  - 지원되는 Class의 수는 계속 늘어나고 매 release시에 포함된다.

### recipetool을 사용해 기본 Package Recipe 자동 생성

- recipetool 을 사용하면 Source code file을 기반으로 기본 Recipe를 쉽게 만들 수 있다.
- 압축을 풀고 Source file을 지정할 수 있으면 recipetool은 Recip를 만들고 신규 Recipe file에 모든 file에 모든 build 정보를 자동으로 설정
- autotools로 build하는 Application
  - recipetools을 사용해 기본 Recipe를 생성하면 Build 의존성을 가진 Recipe를 생성한다.
  - 이 Recipe는 autotools Class를 상속하고 License 요구사항과 Checksum을 설정한다.
  - recipetool은 내용의 이해를 돕기 위해 주석을 함께 포함한 Recipe를 생성한다.
    - 모든 주석은 Meta-layer에 Recipe file을 통합할 때 삭제할 수 있다.
  - <https://github.com/OSSystems/bbexample> 에 있는 bbexample을 사용해 Recipe를 생성

    ```terminal
    source oe-init-build-env build
    recipetool create -V 1.0 https://github.com/OSSystems/bbexample
    ```

    ![COUT: recipetool-create]({{ "/assets/img/post/2020-10-07/06_recipetool-create.png" | relative_url }})
    - recipetool은 URL에서 Source code를 Download해 내용을 분석하고 bbexample_git.bb file을 생성한다.
    - Source code 기반으로 아래 Recipe file을 생성한다.
      ![bbexample_git_bb]({{ "/assets/img/post/2020-10-07/07_bbexample_git_bb.png" | relative_url }})
      - recipetool이 기본 Recipe를 생성하더라도 최종 사용할 Recipe로 완벽하지 않다. Compile option, 추가 Meta-data 정보 등을 확인해야 한다.
      - bbexample_git.bb file은 recipetool을 실행한 Directory에 생성되기 때문에 meta-newlayer/recipe-mine/bbexample/bbexample_git.bb처럼 필요한 위치에 file을 복사해서 사용해야 한다.
      - 복사하고 나면 bitbake를 사용해 build할 수 있다.

## 신규 Machine 추가

- Poky에서 사용하기 위해 새로운 Machine을 만드는 것은 간단하다.
  - 본질적으로 작업을 위한 Machine에 필요한 정보를 제공한다.
- Boot loader, kernel, HW 지원 Driver는 BSP layer에서 Board에 통합하기 위해 시작 전에 점검한다.
- Yocto Project는 현재 사용하는 Embedded Architecture의 대부분을 지원
- Machine 정의에서 사용되는 변수

   TARGET_ARCH | ARM과 i586과 같은 Machine Architecture를 설정
   PREFERRED_PROVIDER_virtual/kernel | 특정한 Kernel을 사용하는 경우 기본 Kernel (linux-yocto) 덮어쓴다.
   SERIAL_CONSOLES | Serial console과 속도를 정의
   MACHINE_FEATURES | 필요한 SW stack이 기본 Image에 포함되게 HW특성을 정의
   KERNEL_IMAGETYPE | zImage나 uImage 같은 Kernel image 종류를 선택하기 위해 사용
   IMAGE_FSTYPES | tar.gz, ext4, ubifs 같은 일반적인 File system image 종류를 선택

- meta-yocto-bsp/conf/machine의 Poky source code에서 Machine file들의 예제를 볼 수 있다.
- 새로운 Machine을 만들 때, 특히 MACHINE_FEATURES 에서 지원하는 기술을 잘 살펴 봐야 한다.
  - 이런 방법으로 특성을 지원하기 위해 필요한 SW가 Image에 설치된다.
  - MACHINE_FEATURES에 대해 이용 가능한 값

    acpi | HW가 ACPI(x86/x86_64에 대해서만)를 구비
    alsa | HW가 ALSA Audio Driver를 구비
    apm | HW가 APM(또는 APM emulation)을 사용
    bluetooth | HW가 통합된 Bluetooth를 구비
    efi | EFI로 Booting 할 수 있게 지원
    ext2 | HW가 HDD 또는 Micro drive를 구비
    IRDA | HW가 IrDA를 지원
    keyboard | HW가 Keyboard를 구비
    pcbios | BIOS로 Booting할 수 있게 지원
    pci | HW가 PCI bus를 구비
    pcmcia | HW가 PCMCIA 또는 CompactFlash Socket을 구비
    phone | Mobile phone(음성)을 지원
    qvga | QVGA(320x240) Display를 지원
    rtc | Real time clock을 지원
    screen | HW가 화면을 구비
    serial | HW가 Serial(일반적으로 RS232)을 지원
    touchscreen | HW가 Touch screen을 구비
    usbgadget | HW가 USB gadget driver와 호환
    usbhost | HW가 USB host와 호환
    vflat | FAT file system을 지원
    wifi | HW가 통합된 wifi를 구비

- Machine에 Image 배치
  - BSP Layer 개발이 끝날 때 까지 자주 잊는 것 중 하나는 Machine에 바로 사용할 수 있는 Image를 생성하는 것
  - 사용하는 Image의 종류는 Processor, Board 에 있는 주변장치, Project 제약 등과 같이 여러 경우에 따라 달라진다.
    - 저장소에 직접 사용하기 위해 가장 많이 사용하는 Image 종류는 Partition된 Image
  - Yocto Project는 이런 Image를 생성할 때 유연하게 사용할 수 있는 wic도구가 있다.
    - Target image layer을 정의하는데 많이 사용하는 언어로 작성된 Template file(wks)을 기반으로 Partition된 Image를 생성할 수 있다.
    - 이것은 <http://www.yoctoproject.org/docs/2.4/ref-manual/ref-manual.html#openebedded-kickstart-wks-reference> 를 참고해 작성
    - wks file은 Meta-data의 wic directory에 있다.
      - 다른 Image layout을 지정할 수 있도록 Directory에 여러 개의 file 갖고 있는 것이 일반적이다.
      - Machine과 일치하는 Layout을 선택하는 것이 중요
      - 예시
        - 두 개의 Partition으로 SD card에 SPL과 U-Boot를 사용하는 i.MX 기반 Machine을 고려할 때 하나는 Boot file용이고 다른 하나는 Root file system용이다.
      - 각 wks file은 다음과 같다.
      ![wks_file]({{ "/assets/img/post/2020-10-07/08_wks_file.png" | relative_url }})
      - wic 기반 Image를 생성 할 수 있게 하려면 IMAGE_FSTYPES에 추가해야 한다.
      - WKS_FILE 변수에 wks file을 설정해 사용할 수 있다.

## Custom 배포

- 배포의 생성은 단순함과 복잡함이 섞여 있다. 배포 file을 만드는 절차는 매우 쉽다.
- 배포 환경설정은 Poky가 사용하는 방식에 큰 영향을 주고, 사용한 Option에 따라 이전에 Build된 Binary에 호환성을 갖지 않는 데 영향을 줄 수 있다.
- 배포는 Tool chain version, Graphic Backend, OpenGL 지원 등과 같은 전역 Option을 정의한다.
  - Poky에서 제공하는 기본 설정을 사용하면 간단한 배포를 만들 수 있지만, 사용자의 요구 사항을 모두 만족시킬 수는 없다.
- 일반적으로 Poky의 작은 Option 집합을 의도적으로 변화 시킨다.
  - E.g. Framebuffer를 사용하게 X11 지원을 제거
- Poky 배포의 재사용과 필요한 변수를 덮어 쓰는 것을 쉽게 수행할 수 있다.
  - E.g. Sample 배포는 다음 \<layer\>/conf/distro/mydistro.conf file처럼 표현한다.
   ![mydistro_conf]({{ "/assets/img/post/2020-10-07/09_mydistro_conf.png" | relative_url }})
- 생성된 배포를 사용하기 위해 build/conf/local.conf에 다음 code 한 부분만 추가하면 된다.
  - DISTRO = "mydistro"
- DISTRO_FEATURES 변수는 Recipe를 설정하고 Package를 Image에 설치하는 방법에 영향을 미친다.
  - E.g. 어떤 Machine과 Image에서 Sound를 사용하기 원하면 alsa 기능이 지원돼야 한다.
  - DISTRO_FEATURE에서 지원하는 기능들

    alsa | ALSA를 지원 (지원하면 OSS 호환 Kernel module이 설치돼 있다.)
    api-documentation | Recipe를 Build하는 동안 API 문서를 생성
    bluetooth | Bluetooth를 지원(통합된 Bluetooth만 해당)
    Bluz5 | 핵심 Bluetooth layer와 Protocol 지원을 제공하는 BlueZ version 5를 포함
    cramsfs | CramFS를 지원
    directfb | DirectFB를 지원
    ext2 | File을 저장하게 내장 Harddisk / Micro driver를 가진 Device를 지원하기 위한 도구를 포함(Flash만 지원하는 Device 대신)
    ipsec | IPSec를 지원
    Ipv6 | IPv6를 지원
    irda | IrDA를 지원
    keyboard | Keyboard를 지원(e.g. Keymap이 booting하는 동안 Loading된다.)
    ldconfig | Target에서 ldconfig와 ld.so.conf를 지원
    nfs | NFS client를 지원한다. (Device에서 NFS를 Mount 하기 위해)
    opengl | 2차원, 3차원 Graphic을 Rendering하기 위해 사용되는 Cross language, Multi-platform application programming interface를 지원하는 OpenGL을 지원 
    pci | PCI bus를 지원
    pcmcia | PCMCIA/CompactFlash를 지원
    ppp | PPP Dial up을 지원
    ptest | 개별 Recipe에서 지원하는 Package test를 Build
    smbfs | SMB Network client를 지원 (Device에서 Microsoft window를 SAMBA로 Mount하기 위해)
    systemmd | Service를 병렬로 시작하는 init을 완벽히 대체하며, Shell overhead 줄이고 다른 특징들을 갖는 초기화 관리자를 지원한다. 이 초기 관리자는 많은 배포판에서 사용한다.
    usbgadget | USB gadget device를 지원(USB networking / serial / 저장장치를 위해)
    usbhost | USB host를 지원(외부 keyboard, Mouse, 저장장치, Network 등을 연결할 수 있게)
    wayland | Wayland Display server protocol과 Library를 지원
    wifi | Wifi를 지원 (통합된 wifi만 해당)
    x111 | X server와 Library를 포함

## MACHINE_FEATURES와 DISTRO_FEATURES 비교

- DISTRO_FEATURES와 MACHINE_FEATURE 모두 최종 System에서 사용 수 있는 기능을 제공
- Machine이 기능을 지원할 때 이 최종 System에 의해 지원되는 것을 시사하지는 않는다.
- 사용하는 배포판에서 해당 기능을 위한 근본적인 기반을 제공해야 하기 때문
- Machine이 Wifi를 지원하지만, 배포판이 지원하지 않으면 OS에서 사용하는 Application은 Wifi 지원 기능이 비활성화된 상태로 Build될 것
  - 그 결과 Wifi 없는 System이 될 것
- 배포판에서 Wifi가 지원되고 Machine이 지원하지 않으면 OS와 Module이 Wifi가 활성화 되어 있더라도 Wifi에 필요한 Module과 Application은 이 Machine이 Build된 Image에 설치되지 않을 것

## 변수의 범위

- BitBake Meta-data는 수천 가지 변수가 있지만 이 변수는 자신이 정의된 곳에 영향을 미치고 해당 영역에서 이용 가능
- 기본적으로 변수에는 다음과 같은 두 가지 종류가 있다.
  - 환경설정 파일에 정의된 변수는 모든 Recipe에서 전역 변수
    - 주된 환경설정 File의 Parsing 순서는 다음과 같다.
      - build/conf/local.conf
      - \<layer\>/conf/machines/\<machine\>.conf
      - \<layer\>/conf/distro/\<distro\>.conf
  - Recipe file에 정의된 변수는 그 작업이 진행되는 동안 특정 Recipe에서만 사용할 수 있는 지역 변수
