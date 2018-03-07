---
layout: post
title: "This is how we deploy a kubernetes cluster"
date: 2018-02-25 19:00:00
categories: blog
author: juner
tags: [kubernetes, ansible]
comments: true
---

# This is how we deploy a kubernetes cluster

## Intro
### kubernetes 배포의 어려움
 - kubernetes 배포는 어렵다. 
 - 다양한 component가 존재하고, 각 component별 옵션도 너무 많다.  
 - master-node의 클러스터 구성이고, 요구하는 리소스가 클 경우 노드의 개수가 많이 필요하여 한번에 제어하기 어렵다. 
 - 물리적으로 나뉜 다양한 망에 배포가 필요하다(내부적인 이슈)
 - 배포 자동화 툴이 필요하다

### kubernetes 배포툴은 다양하다. 
- kubernetes 배포방법은 다양하다.  
| 배포툴 | 설명 |  
|------|------|  
|minikube|  |  
|bootkube|  |  
|kubeadmin|  |  
|kops|  |  
|kube-aws|  |  
|kubespray|  |  
|juju|  |  
|rancher|  |  

### 우리의 선택 및 과정
> __*kubespray*__  

#### 선택의 이유는?
 - 모든 구성요소를 containerizing 해서 설치하는 방식 (hyperkube 이용)으로 구성환경의 제약이 가장 적고,
 - 기본적으로 ansible playbook으로 작성되어 필요에 따른 커스터마이징이 유연하기 때문이었습니다.
 - public cloud 서비스 이용시 지원이 가능
 - baremetal, cloud(aws, openstack)등 다양한 환경에 kubernetes를 배포해야하기 때문에 ansible을 이용하는 방식이 좋음
 - 하지만 kubespray를 그대로 사용하기 보단 수정을 해서 더 빠르게 배포하게 만들어야 함
 - 언제나 동일하게 배포하기 위해서 config관리가 필요함(git을 이용하고 소스 구조를 정리)

#### 어떻게 커스터마이징하여 사용하는가?
 - kubespray는 opensource이고, github에서 언제든지 clone 받을수 있다. 
 - 우리가 배포해야 할 물리적으로 구분된 망들은 node의 구성정보도 다르고 서로 연결되지 않는다.(보안이슈) 
 - 내부에 모든 망에서 접근 가능한 binary repository가 있고, 개발이 가능한 망에 git repository가 있다.(이 git repo에서 build하여 repository에 올려주는 시스템이 존재)

#### kubespray customizing


