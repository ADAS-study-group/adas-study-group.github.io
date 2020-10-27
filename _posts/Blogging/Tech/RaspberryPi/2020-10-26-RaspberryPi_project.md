---
layout: post
title: 라즈베리파이로 무엇을 할까? 
author: devlee8585
date: 2020-10-26 13:00:00 +0800
categories: [Blogging, Tech, RaspberryPi]
tags: [RaspberryPi, gcp, yocto, adas, adaptive autosar]
---

## 라즈베리파이 프로젝트

---

-  라즈베리파이를 이용해서 Yocto, gcp, ADAS, adaptive autosar를 다 경험해 보면 좋을 것 같아서 제안해 봅니다.
1. 우선 yocto 공부한 것을 사용해 보기위해 Yocto를 사용하여 bitbake 할 때 git을 통해 adaptive autosar 소스를 받아서 빌드하도록 레이어를 만들어 봅니다.
2. 데이터 저장을 위해 Google Cloud Platform을  구현하고
3. adaptive autosar의 샘플코드에다가 Lidar or Radar or carmera 모듈을 사서 연결하고 동작해 봅니다.
4. 최종으로 라즈베리파이에 모터와 라이더 or Radar or camera를 활용하여 Autosar가 적용된 Lane / object detection기능의 자율주행 차를 만들어 봅니다. 
   
---


## Reference Links
아래 경로는 가능성을 확인하는 동영상 위주로 링크입니다.

- [라즈베리파이 Yocto 환경 설정](<https://www.youtube.com/watch?v=V0i0m2pQYyU>)
- [라즈베리파이 + Lidar](<https://www.youtube.com/watch?v=t0ud9HTYjMI>)
- [Autosar + lane/object detection + 라즈베리파이 자율주행 자동차](<https://www.youtube.com/watch?v=QPMSqu_i9hs>)
