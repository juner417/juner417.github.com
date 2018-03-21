---
layout: post
title: "OSI 7 Layer"
date: 2018-03-21 18:00:00
categories: blog
author: juner
tags: [network,cs]
comments: true
---

# Network 기초 - OSI 7 Layer
## 참고자료
- [그림으로 배우는 네트워크 구조](http://www.yes24.com/24/Goods/36552194?Acode=101)
- [후니의 쉽게 쓴 시스코 네트워킹](http://www.yes24.com/24/Goods/4747319?Acode=101)

## OSI 7Layer
 - 목적 : 표준, 학습도구
 - [영문위키](https://en.wikipedia.org/wiki/OSI_model)
 - [한글위키](https://ko.wikipedia.org/wiki/OSI_%EB%AA%A8%ED%98%95) 

| layer | name | 대표 protocol |   
|---|---|---|
|L7| application |http, https, pop, imap, DHCP, DNS, FTP|  
|L6|presentation |jpeg, mpeg, ascii|
|L5|session |ssl/tls|           
|L4|transport|TCP/UDP|  
|L3|network |IP. ICMP|  
|L2|data link |mac|  
|L1|physical| ethernet|  

### L7 application layer
 - 인터넷 서비스를 제공하는 레이어
 - 사용자가 서비스에 접근할수 있도록 하는 프로토콜
 - 사용자 인터페이스, 전자 우편등 서비스를 제공함

### L6 presentation layer
 - 사용자 시스템에서 데이터의 형식상 차이를 다루는 레이어
 - 입력/출력되는 데이터를 형식에 맞게 표현해 준다.(인코딩)
 - 필요한 번역을 수행하여 두장치가 일관되게 전송데이터를 서로 이해할수 있도록 한다. 
 - 제어 코드나, 문자 및 그래픽등의 확장자를 생각하면 이해하기 쉬움

### L5 session layer
 - 양 끝단의 응용프로세스가 통신을 관리하기 위한 제어 레이어
 - TCP/IP 세션을 만들고 없애는 책임을 진다. 
 - 통신 장치간의 상호작용을 설정하고(체크포인팅/유휴/종료) 유지하며, 동기화, 종료한다.

### L4 transport layer
 - 양 끝단(end-to-end)의 사용자 들이 신뢰성 있는 데이터를 주고 받을수 있도록 하는 레이어
 - 패킷(packet)들의 전송이 유효한지 확인하고 실패한 패킷을 다시 보내는 등 신뢰성 있는 통신을 보장한다.  
 - 오류검출 및 복구와 흐름제어, 중복검사 등을 수행한다.
 - 시퀀스 넘버 기반의 오류제어 방식을 사용한다. 

### L3 network layer
 - 여러 네트워크 링크에서 패킷(packet)을 발신지로부터 목적지까지 전달하는 레이어
 - 여러개의 네트워크를 거칠때마다 경로를 찾아주는 역할을 한다. 
 - 라우팅, 흐름제어 세그멘테이션, 오류제어, 인터네트워킹 등을 수행한다. 
 - 데이터를 연결하는 다른 네트워크를 통해 전달함으로써 인터넷이 가능하게 만드는 계층
 - 이 계층은 각 패킷이 시작지점부터 최종 목적지 까지 성공적이고 효과적으로 전달되도록 한다.  

### L2 datalink layer
 - 포인트 투 포인트간 신뢰성 있는 전송을 보장하기 위한 레이어
 - 오류없이 한 장치에서 다른 장치로 프레임(frame)을 전달하는 역할을 한다. 
 - 스위치 같은 장비의 경우 MAC 주소를 이용하여 정확한 장치로 데이터 전달, 노드 대 노드간 전달을 감독한다. 
 - 주소 체계는 계층이 없는 단일 구조(L3 IP는 계층 구조)

### L1 pysical layer
 - 물리적 장치를 이용하여 비트(bits)흐름을 전송하고 조정하기위한 레이어 
 - 네트워크의 기본 네트워크 하드웨어 전송 기술을 이룬다. 

## TCP/IP 4Layer(인터넷 모델)
| layer | name | 대표 protocol |   
|---|---|---|
|L4 | application | tcp/ip기반의 응용프로그램  http, ftp, telnet, dns, smtp |
|L3 | transport | 통신 노드간 연결을 제어/송수신   tcp, ump |
|L2 | internet | 통신 노드간 ip패킷을 전소하는 기능 및 라우팅  IP, ICMP|
|L1 | network interface | ethernet, token ring|

### OSI 7layer - TCP/IP 4layer
 - 인터넷에서 찾아보니 아래와 같이 매핑하더라...[출처](http://codedragon.tistory.com/4215)

| OSI | TCP/IP |  
|---|---|---|
|l7 application| l4 application|
| l6 presentation|^| 
| l5 session |^|
| l4 transport | l3 tranport |
| l3 network | l2 internet |
| l2 datalinke | l1 network interface|
| l1 pysical |^|

