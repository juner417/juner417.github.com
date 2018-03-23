---
layout: post
title: "TCP,UDP"
date: 2018-03-23 14:00:00
categories: blog
author: juner
tags: [network, cs]
comments: true
---

# Network 기초 - TCP/UDP
## 참고자료
- [그림으로 배우는 네트워크 구조](http://www.yes24.com/24/Goods/36552194?Acode=101)
- [후니의 쉽게 쓴 시스코 네트워킹](http://www.yes24.com/24/Goods/4747319?Acode=101)

## TCP/UDP
### tcp

 - 연결형서비스를 지원하는 전송계층(L4) 프로토콜
 - 연결 후 통신, 1-1 통신방식
 - 높은 신뢰성 보장(seq num, ack num)
 - 연결(3way handsaking) 해제(4way handshaking) 로 연결관리
 - 데이터 흐름제어(수신자 버퍼 오버플로우 방지) 혼잡제어(네트워크내 패킷수가 과도하게 증가하는것 방지)

### UDP

 - 비연결형 서비스를 지원하는 전송계층(L4) 프로토콜
 - 연결없이 통신 1-1, 1-N, N:M 통신방식
 - 사용자 데이터그램형 프로토콜
 - 정보를 주고받을 때 보낸다는 신호나 받는다는 신호 절차를 거치지 않고, 보내는 쪽에서 일방적으로 데이터를 전달하는 프로토콜
 - 패킷 오버헤드가 적어 네트워크 부하감소
 - 비신뢰성
 - DNS, SNMP, NFS등에 사용

