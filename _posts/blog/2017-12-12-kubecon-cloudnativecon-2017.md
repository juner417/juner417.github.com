---
layout: post
title: "Kubecon + CloudNative con 2017 review"
date: 2017-12-12 22:00:00
categories: blog
author: juner
tags: [kubernetes]
#image:
#  feature: so-simple-sample-image-7.jpg
#  credit: WeGraphics
#  #creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
share: true
comments: true
---

# kubecon + cloudnative con 2017
 - 내가 생각하는 Keyword

## container 더이상 docker가 답은 아니다. 
---
docker의 containerd, coreos의 rkt (CNCF donation)   
kubernetes의 [cri-o](https://github.com/kubernetes-incubator/cri-o) ibm의 [kata-container](https://katacontainers.io/)등 다양한 container runtime을 개발하고 있다.   
docker의 독주를 막기 위해 다양한 종류의 container환경이 추가로 생길 가능성이 높다. 



## service mesh의 개념의 등장 및 이것을 위한 opensource 들의 등장
---
> service mesh - A service mesh is a dedicated infrastructure layer for making service-to-service communication safe, fast, and reliable.      

service mesh란 service-to-service(inter microservice)의 통신을 안전하고, 빠르고, 신뢰할수 있게 만드는 전용 인프라 스트럭쳐 레이어이다.   
service mesh는 cloud native하게 개발된(microservice) 서비스들의 복잡한 토폴로지상을 통해 들어오는 요청의 안정적인 전달을 담당하고 트레이싱한다.  

왜 이것이 화두가 되는가?

cloud native model은 하나의 application이 수백개의 서비스로 구성된다(microservice), 그리고 각 서비스는 수천개의 인스턴스(pod or vm)들을 가지고있다.   
그리고 이런 인스턴스는 끊임없이 변화하는 상태(consistantly-changing state)를 가지고 있다. 그것은 k8s같은 것으로 동적으로 스케줄링되므로.  

이런 환경의 서비스 통신은 믿을 수 없을 정도로 복잡 할뿐만 아니라 런타임 동작에서 보편적이고 기본적인 부분이다.
이 service mesh layer를 관리하는 것은 엔드 투 엔드 성능과 신뢰성을 보장하는 데있어 중요하기 때문에 화두가 되고 있다.   

envoy, linkerd가 cloudnative foundation에 donation되었고, istio와 buoyant 회사가 conduit라는 service mesh application을 개발하고 있다.    

service mesh
 - envoy
 - linkerd (conduit...)

service observation
 - opentracing
 - jaeger  

## statefull app을 위한 cluster 
---
stateless application은 이미 많은 부분이 container와 kubernetes를 통해 배포 및 서비스 되고 있다.   
하지만 statefull한 application도 container화 되고 kubernetes를 통해 배포를 원하는 요구사항이 있었고, 이런 케이스의 발표들이 많았다.   
기본적인 구조는 하나의 클러스터에 storage layer와 application layer를 담당하는 노드로 구성하고, storage layer노드에 gluster, ceph등을 배포한 뒤, 
kubernetes storageclass를 이용하여 pv를 관리하고 application layer노드에는 db들의 프로세스를 pod으로 올린뒤 연결하여 사용하는 구조를 사용하고 있었다. 

이런 구조는 우리에게 적용 가능할듯하다, 간단히 나온 아이디어는 cluster를 크게 statefull layer, stateless layer, storage layer, gpu layer cluster군으로 나누어 구축한 뒤   
이 cluster를 federation으로 묶어서 관리하면, 다양한 수요를 수용할수 있는 큰 cluster service를 제공할 수 있을거라 생각한다.(cluster busting과 별개로...)

[You have stateful apps](https://kccncna17.sched.com/event/eb705c6c6186f308e9ce15c627853136)  
[Evolving and supporting stateful](http://sched.co/CU6n) - 이건 못봄...  
[Don't hassle me, I'm stateful](http://sched.co/CU7l) - 이것도 못봄...


## kubernetes의 BF인 CI/CD
---
kubernetes에 서비스를 배포하기 위해서 다양한 ci/cd가 사용되고 있다. 
그리고 이것은 container cluster 생태계에서는 필수요소이다.
jenkins - spinnaker - artifactory의 조합으로 많이 사용되고
특히 spinnaker는 contents delevery의 모든 파이프 라인을 하나의 workload로 구현가능하게 하여 많은 개발자들이 관심을 가진 듯 했다. 
그리고 엔지니어가 서비스를 매니징을 할수 있도록 각 서비스 workload를 모니터링을 제공한다. 
나중엔 ci까지 흡수할 예정이라는 포부까지 밝혔다. 

kubernetes에 서비스를 배포 트렌드를 보면, 
binary build -> containerizing(image) -> push image -> service define(service manifest templating) -> test deploy(canary test) -> prod deploy로 구성되어 있다. 
이런 프로세스는 기존의 프로세스와 다르고, 다양한 플랫폼(docker, k8s등)의 인터페이스를 사용하야 하고, 배포시 필요한 config를 별도로 관리해야 하기 때문에 빌드/배포의 자동화가 필수적이다. 
그래서 CI/CD 시스템을 kubernetes와 연결하여 kubernetes에 실행되는 서비스들의 모든 배포 프로세스를 통합 및 자동화하는 구조이다.  

만약 위에서 말한 federation cluster를 구축한다면, 이 cluster로 들어오는 CI/CD pipeline을 별도로 구성해야 하는지는 고민해봐야 할 문제 일 것이다. 

[Continuos intergration at scale on kubernetes](https://kccncna17.sched.com/event/8ccac3b6c94940936b22531bee36334b)  
[Microservice, Service mesh and CI/CD pipeline](https://kccncna17.sched.com/event/8194aa2d21cd034dd769e2280c0def26)


## application manifest
---
kubernetes에 배포를 하기 위한 application manifest(application을 k8s에 배포하기 위한 정의서)를 어떻게 저장(helm repo)하고, CD에 연동할 것인가 라는 주제의 세션을 종종 볼수 있었다. 
대부분 helm을 사용하여 배포하였지만, git과 config repo(application config)를 별도로 구성하여 사용하는 경우도 있었다. 
kubernetes를 사용하여 배포하기 위해 필수로 필요한 프로세스 이므로, 다들 어떻게 template를 만들고 자동으로 generation 하고, 각 cluster별로 다른 config를 저장하여 관리할지가 중요한 포인트였다. 

그리고 application이 아닌 인프라와 그 config들도 모두 manifest형태로 만들어 git으로 관리하라는 세션이 인상적이었다.
우리의 경우 kubespray(caas)로 배포할때만 infra manifest를 generation하고 그 내용을 서버에만 관리하는데, 이 부분을 수정하여 별도로 저장 관리할지, 아니면 manifest와 config를 함께 저장하여, 
필요할 경우 자동으로 generation하게 할지는 고민해 볼만한 이슈라 생각한다. 

[One chart to rule them all](https://kccncna17.sched.com/event/0d0f6ef64271553b59f9789cbbf9e1e9)  
[The architecture of a multi-cloud environment with kubernetes](https://kccncna17.sched.com/event/e341137b2dd8dcf8ceeba48c27c8cbfb)
[Zero configuration](https://kccncna17.sched.com/event/8a8c70ef42efca1ce00f6821d934ee74)


## Serverless, Machine Lerning, Data analysis
---
kubernetes에서 실행되는 서비스들 중 많이 R&D스럽게 연구되고 있는 분야로 보이는 것은 이 세가지였다.  
실 사용 예시보다는 플랫폼 회사들이 앞다투어 개발하고 있다는 것을 발표하였다.(돈이 될 것 같나?) 아쉽게도 시간이 없어서 발표는 챙겨보지 못했으나, 쇼케이스 부스에서 많은 설명을 들을수 있었는데 영어라 까먹었다...

serverless
 - fission : platform9
 - serverless computing : ibm
 - kubeless : bitnami
 - OpenFaaS : ...?

machine learning
 - alibaba cloud
 - azure 

data platform  
 - data dog
 - treasure data

[All tou need to know to build your gpu machine learning cloud](http://sched.co/CU61)   
[Democratizing machine learning on kubernetes](http://sched.co/CU6U) - 이거 못봄  
[Modern big data pipeline over kubernetes](http://sched.co/CU7d) - 이거 못봄


