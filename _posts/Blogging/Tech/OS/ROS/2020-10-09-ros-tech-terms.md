---
layout: post
title: ROS Technical Terms
categories: [Blogging, Tech, OS, ROS]
tags: [ros, term, jargon]
---
## Technical Terms

Term | Description
-----| -------------
[ROS](#ros) (Robot Operating System) | Robot SW를 작성하기 위한 Framework or Platform
[Master](#master) | Node 사이 연결과 Message 통신을 위한 Name server 역할
[Node](#node) | ROS에서 실행되는 최소 단위의 Processor를 지칭
XML-RPC (XML - Remote Procedure Call) | RPC Protocol의 일종으로서, Encoding 형식에는 XML, 전송 방식에서는 HTTP Protocol을 사용
Message | Node 간 Data를 주고 받을 때 사용하는 Medium으로, int, float, bool, point 등 변수 형태이며, 간단한 Nested 구조나, Message Array 등을 사용가능
[Topic](#topic) | Publisher - Subscriber 관계에서 사용하는 Medium
[Service](#service) | 요청과 응답을 사용하는 동기 방식의 Message교환 방법의 일종
[Action](#action) | Service처럼 양방향 통신이 필요하지만, 요청 처리 후 응답까지 오랜 시간이 걸리고, 처리 상태가 필요한 경우 사용됨
Bag | ROS에서 주고받는 Message Data를 저장할 때 사용하는 File format, Bag을 재생하여 이전 상황을 그대로 재현 가능(같은 실험을 수행하지 않아도 Sensor값을 반복하여 사용가능)
[Catkin](#catkin) | ROS1의 Build package로서, ROS에 맞게 Customize된 CMake macro 집합이라 할 수 있음
[Meta-OS](#meta-os) (Meta-Operating System) | Application과 Parallel Computing 자원간의 가상화 Layer (비공식 용어)

## Detailed description

### ROS

- 일반 OS에서 제공하는 HW 추상화, 저수준 기기제어, 빈번히 사용되는 기능들이 구현되어 있음
- Process간 Message 전달 Package 관리 기능 등을 제공
- 여러 Computer system 에서 작동하는 Code를 Fetch, Build, Implementation, Run하기 위한 Tools & Library를 제공
- 일종의 MOM (Message-Oriented Middleware)라고 할 수 있음

### Master

- roscore를 통해 실행
- Master를 실행하면 각 Node의 이름을 등록하고 필요에 따라 정보를 받을 수 있다.
- Master를 제외한 Node간에는 접속, Message 통신이 불가하다.
- HTTP 기반의 XMLRPC Protocol을 사용하여 통신한다.
- Node는 필요할 때만 Master에 접속하여 정보를 등록하거나 다른 Node의 정보를 요청한다.
- Server 주소와 Port로는 ROS_MASTER_URI 변수에 기재된 URI 주소와 Port를 사용한다.
- 기본값은 ${Current Local IP}:11311이다.

### Node

- ROS에서는 하나의 목적에 하나의 Node 작성을 권장
  - 재사용이 쉽게 구성하여 개발하는 것이 권장됨
- Mirco-service Architecture 개념
- Mobile Robot의 경우, 아래와 같이 Node를 세분화 하여 작성
  - Sensor drive
  - Sensor data를 이용한 변환
  - 장애물 판단
  - Motor 구동
  - Encoder 입력
  - Navigation
- Node 구동시 Master에 다음 항목들이 등록됨
  - 이름,
  - Publisher
  - Subscriber
  - Service Server
  - Service Client가 사용하는 Topic 및 Service 이름
  - Message 형태
  - URI주소와 Port
- Master에 등록된 이러한 정보를 기반으로 Node 간 통신이 가능
  - URI 주소는 ROS_HOSTNAME 환경변수 값으로 지정
  - Port는 임의의 고유값으로 설정
- Node - Master 간 통신에서 XMLRPC를 사용
- Node - Node 간,
  - 접속 요청과 응답은 XMLRPC
  - Message 통신은 TCP/IP 기반의 TCPROS 사용

### Topic

- Publisher가 Master에 Topic을 등록한 뒤, Message를 일방 출력하면 Master에 등록된 Subscriber가 Data를 수신하는 형태
- Topic은 Publisher가 Subscriber에게 일방적으로 Data를 전달하는 단방향 통신이며, 비동기 적으로 연속적인 Message를 전달하는 통신방법이기 때문에 Sensor Data 전송에 적합

### Service

- Client의 요청이 있을 때 응답하는 Server를 가진다.
- 일회성 Message 통신으로 요청과 응답이 완료되면 두 Node간 접속이 끊긴다.

### Action

- 요청과 응답에 해당하는 Gool과 Result가 있으며, 중간 결과에 해당하는 Feedback이 추가됨
- Action Client가 Goal을 Action Server에 전달하면, Action Server가 수행 상태에 따라 Feedback과 Result를 반환

### Catkin

- CMakeLists.txt를 이용해서 사용
- rosbuild -> catkin (ROS1) -> ament -> colcon (ROS2)

### Meta OS

- Parallel Computing Resource를 활용하여, Scheduling, Load, Monitoring, Error Handling 등을 실행하는 System
- 즉, Windows, Linux, Android와 같은 전통적인 OS와 같은 개념이라기 보다, 이 OS들을 이용함
  - 기존 OS의 다음과 같은 기능 사용
    - Process Management System
    - File System
    - User Interface
    - Program Utilities (Compiler, Thread model)
  - 추가적으로 다음과 같은 Robot Application 개발을 위한 필수 기능들을 Library 형태로 제공
    - 다수의 이기종 HW간의 Data communication
    - Scheduling
    - Error handling
  - 이러한 Robot SW Framework 를 기반으로 다양한 목적의 Application을 개발, 관리, 제공
  - User들이 개발한 Package를 유통하는 Ecosystem 을 갖추고 있다.

    ![Image of Meta-OS]({{ "/assets/img/post/2020-10-09/robotis-lec-1_2.png" | relative_url }})
