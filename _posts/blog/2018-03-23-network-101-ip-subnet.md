---
layout: post
title: "IP,Netmask,Subnet,CIDR"
date: 2018-03-23 14:00:00
categories: blog
author: juner
tags: [network, cs]
comments: true
---

# Network 기초 - IP/Netmask/Subnet/CIDR
## 참고자료

- [그림으로 배우는 네트워크 구조](http://www.yes24.com/24/Goods/36552194?Acode=101)
- [후니의 쉽게 쓴 시스코 네트워킹](http://www.yes24.com/24/Goods/4747319?Acode=101)
- [IP 란](https://namu.wiki/w/IP)
- [네트워크 클래스](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC_%ED%81%B4%EB%9E%98%EC%8A%A4)
- [subnet](https://ko.wikipedia.org/wiki/%EB%B6%80%EB%B6%84%EB%A7%9D)
- [subnet mask](https://namu.wiki/w/%EC%84%9C%EB%B8%8C%EB%84%B7%20%EB%A7%88%EC%8A%A4%ED%81%AC)
- [CIDR](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%EB%8D%94_(%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%82%B9))

## IP

* IP(Internet Protocol)는 인터넷상에 있는 개별 네트워크끼리 패킷을 라우팅하는 프로토콜이다. 이 프로토콜을 이용하여 다른 네트워크에 있는 컴퓨터와 통신할 수 있다. [L3 network 레이어]({{ site.url }}/blog/network-101-osi-7-layer/)의 대표 프로토콜이다.
* 호스트(컴퓨터)에서 호스트까지의 통신을 책임진다. Process data unit은 패킷(packet)이다. 

## IP 주소

* IP 통신에 필요한 고유 주소, IPv4, IPv6 두가지 체계가 있다. 
* IPv4는 32비트의 값을 가진다. 일반적으로 8비트씩 끊어 0~255의 10진수 숫자로 나타내며, 각 숫자는 점(.)으로 구분한다. 일반적인 IP 주소는 이 IPv4를 말한다. 
* IP 주소는 네트워크부와 호스트부로 나뉜다. 
* 네트워크 주소는 특정 네트워크의 주소이고 라우팅시 사용한다. 호스트 주소는 네트워크에 속한 호스트의 주소이다. 
* 여기서 하나의 네트워크란 하나의 브로드캐스트 영역이라고 생각하면 된다. 라우터를 거치지 않고도 통신이 가능한 영역.  
* 이 네트워크 주소 부분의 비트를 1로 치환한 것이 그 네트워크의 넷마스크이다.  

![ip-address]({{ site.url }}/assets/post_images/2018-03-23-network-101-ip-subnet/ip-address.png)

## IP 주소의 클래스

* 네트워크(lan/vlan)의 규모에 따라 결정된다. 
* 클래스의 차이는 A~C(D,E도 있음)사이에 몇개의 호스트를 넣을수 있느냐의 차이.
* 고정구간으로 낭비가 많아 CIDR 체계로 변경됨.

|클래스|첫 고정 비트|네트워크주소영역|호스트주소영역|
|---|---|---|---|
|A class|0|8bit(2^7개)|24bit(2^24-2개)|
|B class|10|16bit(2^14개)|16bit(2^16-2개)|
|C class|110|24bit(2^21개)|8bit(2^8-2개)|

## Netmask

* 네트워크 주소 부분의 비트를 1로 치환한 것이 넷마스크이다. 
* IP 주소와 넷마스크를 AND연산을 하면 네트워크 주소를 얻을수 있다. 

![netmask]({{ site.url }}/assets/post_images/2018-03-23-network-101-ip-subnet/netmask.png)

## Subnet/Subnetmask

* 한개의 네트워크를 여러개의 서브넷으로 분할 하는것
* 서브넷화 하는 경우에는 네트워크 주소 부분의 비트를 연장한다. 그리고 그 나머지 호스트 부분이 호스트 식별자가 된다. 

![subnet]({{ site.url }}/assets/post_images/2018-03-23-network-101-ip-subnet/subnet.png)

## CIDR

* Classless Inter-Domain Routing(CIDR, 사이더)로 클래스 없는 도메인 간 라우팅 기법으로 최신의 IP 주소 할당 방법이다. 
* 사이더는 기존의 IP 주소 할당 방식이었던 네트워크 클래스틑 대체하였다. 
* ip 주소 뒤에 / 로 구분하고, 서브넷마스크의 비트수를 적어 표시하는 것을 CIDR 표시법이라 부른다. 
* [CIDR 테이블 참조](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%EB%8D%94_(%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%82%B9))

```bash
192.168.1.0/26
```
