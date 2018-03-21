---
layout: post
title: "LAN,WAN,VLAN,TCP/IP 개념"
date: 2018-03-21 20:00:00
categories: blog
author: juner
tags: [network, cs]
comments: true
---

# Network 기초 - LAN/WAN
## 참고자료
- [그림으로 배우는 네트워크 구조](http://www.yes24.com/24/Goods/36552194?Acode=101)
- [후니의 쉽게 쓴 시스코 네트워킹](http://www.yes24.com/24/Goods/4747319?Acode=101)
- [TCP/IP란](https://www.joinc.co.kr/w/Site/Network_Programing/Documents/IntroTCPIP)
- [VLAN이란](http://onecellboy.tistory.com/278)


## LAN(Local Area Network)
 - 사무실이나 가정 등 가까운 지역을 하나로 묶는(거점) 네트워크를 말한다.
 - L2 switch를 이용한 네트워크 구성이 대표적임
 - 하나의 lan은 동일한 네트워크 주소(IP 주소 중 네트워크 주소 부분)를 갖는다.  

## WAN(Wide Area Network)
 - 여러개의 LAN이 모여 거점과 거점을 연결한 네트워크를 가르킨다. 
 - 서로 다른 네트워크를 연결하기 위해 L3 switch(router)를 이용한 구성이 대표적임

## LAN/WAN 구성
![lan-wan]({{ site.url }}/assets/post_images/2018-03-21-network-101-lan-wan/lan-wan.png)     
> L2 switch로 연결된 구간이 LAN  
> L3 router로 연결된 구간이 WAN

## TCP/IP
- TCP/IP란 프로토콜이다. [OSI 7 Layer 참조]({{ site.url }}/blog/network-101-osi-7-layer/)
- TCP(Transmission Cotrole Protocol)는 데이터의 확인이나 재전송등을 수행하며 신뢰도가 높은 연결형 통신을 구현하는 프로토콜로 L4 transport 레이어의 대표적인 프로토콜이다.  
- IP(Internet Protocol)는 인터넷상에 있는 개별 네트워크끼리 패킷을 라우팅하는 프로토콜이다. 이 프로토콜을 이용하여 다른 네트워크에 있는 컴퓨터와 통신할 수 있다. L3 network 레이어의 대표 프로토콜이다. 

## 왜 TCP/IP가 인터넷을 지칭하는 대명사가 되었을까?
- 현재 인터넷에서 가장 많이 사용되는 서비스(www, email, ftp, telnet)가 TCP/IP를 기반으로 만들어졌다.
- 하드웨어, 운영체제, 접속매체에 상관없이 동작할수 있다는 개방성 때문에 인터넷 통신의 인기 프로토콜이 될수 있었다. 

## VLAN이란
virtual local area network의 약자로 물리적 배치와 상관없이 논리적으로 lan을 구성할 수 있는 기술
하나의 스위치에 여러개의 포트를 그룹으로 묶어 vlan을 구성 할수 있다.  
### 장점
 - 네트워크 리소스 보안을 높힌다.  - 실제적인 네트워크 그룹의 이동이 없어도 되어 보안상의 문제를 줄인다. 
 - 비용을 절감할 수 있다.  - 서로 차단된 환경을 구성하기 위해 스위치 장비를 구매할 필요가 없다. 
 - 관리자의 네트워크 설정 작업에 용이하다. - 특정장비의 네트워크 설정을 옮길떄 스위치 설정만으로 변경 가능
 - 불필요한 트래픽을 줄인다. - vlan은 서로다른 네트워크 그룹이기 때문에 브로드케스트 패킷이 다른 vlan으로 전송되지 않는다.  


