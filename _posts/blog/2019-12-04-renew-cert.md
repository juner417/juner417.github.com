---
layout: post
title: "Renewal k8s master component certificate"
date: 2019-12-04 01:00:00
categories: blog
author: juner
tags: [kubernetes]
comments: true
---

# kubernetes cluster master certification 교체
## 내가 가진 cluster는 왜 100년 cert가 아닌가?
* kubeadm을 이용한 ansible 배포로 kubeadm의 cert 갱신 주기를 따름
  * root ca의 경우 10년 master component들의 경우 1년짜리
  * kubeadm upgrade기능을 이용하면 6개월 남았을경우 교체하지만, 현재 그 기능을 이용하지 않음
* 위와 같은 이유로 아래의 방식으로 수동 교체 해줘야 함(자동화 가능 하지만 안정성과 공수 효율성 문제로 바로 대응할수 있는 수동 방식 채택)

## cert due date 확인
* [leoh0 check cert kubectl plugin](https://github.com/leoh0/kubectl-check-cert)
```bash
$ kc check-cert
 9 / 9 [============================================================] 100.00% 1s
+--------------------+---------------------+----------------------------+------+-------------------------------+--------------------------------------------------+---------+
|        TYPE        |              NODE   |            NAME            | DAYS |              DUE              |                       PATH                       | WARNING |
+--------------------+---------------------+----------------------------+------+-------------------------------+--------------------------------------------------+---------+
| apiserver          | kube-master004-prod | kubelet-client-certificate |   66 | 2020-01-10 05:18:38 +0000 UTC | /etc/kubernetes/pki/apiserver-kubelet-client.crt |         |
| apiserver          | kube-master004-prod | proxy-client-cert-file     |   66 | 2020-01-10 05:18:38 +0000 UTC | /etc/kubernetes/pki/front-proxy-client.crt       |         |
| apiserver          | kube-master004-prod | tls-cert-file              |   66 | 2020-01-10 05:18:37 +0000 UTC | /etc/kubernetes/pki/apiserver.crt                |         |
| apiserver          | kube-master005-prod | kubelet-client-certificate |   66 | 2020-01-10 05:20:51 +0000 UTC | /etc/kubernetes/pki/apiserver-kubelet-client.crt |         |
| apiserver          | kube-master005-prod | proxy-client-cert-file     |   66 | 2020-01-10 05:20:51 +0000 UTC | /etc/kubernetes/pki/front-proxy-client.crt       |         |
| apiserver          | kube-master005-prod | tls-cert-file              |   66 | 2020-01-10 05:20:50 +0000 UTC | /etc/kubernetes/pki/apiserver.crt                |         |
| apiserver          | kube-master006-prod | kubelet-client-certificate |   66 | 2020-01-10 05:23:21 +0000 UTC | /etc/kubernetes/pki/apiserver-kubelet-client.crt |         |
| apiserver          | kube-master006-prod | proxy-client-cert-file     |   66 | 2020-01-10 05:23:21 +0000 UTC | /etc/kubernetes/pki/front-proxy-client.crt       |         |
| apiserver          | kube-master006-prod | tls-cert-file              |   66 | 2020-01-10 05:23:21 +0000 UTC | /etc/kubernetes/pki/apiserver.crt                |         |
| controller-manager | kube-master004-prod | client-cert                |   17 | 2019-11-21 11:32:58 +0000 UTC | /etc/kubernetes/controller-manager.conf          |         |
| controller-manager | kube-master005-prod | client-cert                |   17 | 2019-11-21 11:20:22 +0000 UTC | /etc/kubernetes/controller-manager.conf          |         |
| controller-manager | kube-master006-prod | client-cert                |   17 | 2019-11-21 11:25:52 +0000 UTC | /etc/kubernetes/controller-manager.conf          |         |
| scheduler          | kube-master004-prod | client-cert                |   17 | 2019-11-21 11:32:59 +0000 UTC | /etc/kubernetes/scheduler.conf                   |         |
| scheduler          | kube-master005-prod | client-cert                |   17 | 2019-11-21 11:20:23 +0000 UTC | /etc/kubernetes/scheduler.conf                   |         |
| scheduler          | kube-master006-prod | client-cert                |   17 | 2019-11-21 11:25:52 +0000 UTC | /etc/kubernetes/scheduler.conf                   |         |
+--------------------+---------------------+----------------------------+------+-------------------------------+--------------------------------------------------+---------+
```

## master component(controller, scheduler)의 endpoint를 확인하여 leader 확인
```bash
$ kc get ep -n kube-system kube-scheduler -o yaml
apiVersion: v1
kind: Endpoints
metadata:
  annotations:
    control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"kube-master004-prod_d10e36d9-1499-11e9-bcc0-a0369ff18b98","leaseDurationSeconds":15,"acquireTime":"2019-01-10T05:44:31Z","renewTime":"2019-11-12T05:03:00Z","leaderTransitions":13}'
  creationTimestamp: "2017-11-21T05:32:04Z"
  name: kube-scheduler
  namespace: kube-system
  resourceVersion: "384506007"
  selfLink: /api/v1/namespaces/kube-system/endpoints/kube-scheduler
  uid: 4e6e1019-ce7d-11e7-991b-a0369ff18b98

$ kc get ep -n kube-system kube-controller-manager -o yaml
apiVersion: v1
kind: Endpoints
metadata:
  annotations:
    control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"kube-master004-prod_d1395959-1499-11e9-a277-a0369ff18b98","leaseDurationSeconds":15,"acquireTime":"2019-01-10T05:44:28Z","renewTime":"2019-11-12T05:01:26Z","leaderTransitions":12}'
  creationTimestamp: "2017-11-21T05:32:05Z"
  name: kube-controller-manager
  namespace: kube-system
  resourceVersion: "384505162"
  selfLink: /api/v1/namespaces/kube-system/endpoints/kube-controller-manager
  uid: 4f60e6ad-ce7d-11e7-991b-a0369ff18b98

## 6->5->4 순서로 진행
```

## 각 노드별 변경 프로세스
```
$ systemctl stop kubelet
$ cd /etc/kubernetes/
$ cp -r pki pki_20191112
$ rm -rf pki/apiserver.* pki/apiserver-* pki/front-proxy-client.*
$ mv pki pki_old
$ kubeadm reset
$ mv pki_old pki
$ kubeadm init --config /etc/kubernetes/kubeadmin.yml
$ chown -R root:root pki # 혹시나 파일의 오너가 root가 아닐경우.
```

## kubadm(>v1.15)으로 controller-manager, scheduler client certification update하기
* master cert를 cfssl, openssl을 이용하여 100년으로 만들어도 controller-manager, scheduler, etcd의 client cert가 100년짜리가 아니다
* 이 경우 위 방식으로 하기에는 문제가 있다. 그래서 1.15 이상 버전의 cert renew 기능을 이용하자(client가 자신의 local cert만 변경)

```
Dockerfile
FROM ubuntu:18.04

RUN apt-get update
RUN apt-get install -y curl

COPY kubeadm /usr/bin/ # 여기서 kubeadm은 >v1.15 이상으로 받아둔다.
```

### build

```
$ docker build -t registry/junho-son/kubeadm:v1.15.2 .
$ docker push registry/junho-son/kubeadm:v1.15.2
```

### mount and renew

```
$ docker run -V /etc/kubernetes:/etc/kubernetes registry/junho-son/kubeadm:v1.15.2 bash
$ kubeadm alpha certs renew controller-manager.conf
$ kubeadm alpha certs renew scheduler.conf
```

