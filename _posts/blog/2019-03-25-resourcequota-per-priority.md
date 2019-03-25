---
layout: post
title: "ResouceQuota per PriorityClass"
date: 2019-03-25 19:00:00
categories: blog
author: juner
tags: [kubernetes]
comments: true
---
# How to set PriorityClass on Resource Quota

## 미리 알고 있으면 좋은 것들...
* [priorityClass](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#priorityclass)
* [priorityClass example](https://console.bluemix.net/docs/containers/cs_pod_priority.html#default_priority_class)
* [priorityClass API](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/#priorityclass-v1beta1-scheduling-k8s-io)
* [priorityClass in resource quota](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/pod-priority-resourcequota.md)
* [ResourceQuota](https://kubernetes.io/docs/concepts/policy/resource-quotas/#compute-resource-quota)
* [ResoucesQuota per PriorityClass](https://kubernetes.io/docs/concepts/policy/resource-quotas/#resource-quota-per-priorityclass)
* [ResourceQuotaScopeSelectors feature gate version](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/)

## Summary - PriorityClass
* PriorityClass는 namespace에 제약 받지 않는 object
* priority class name(.metadata.name)으로 부터 우선순위 정수 값(.value)을 매핑하여 정의한다. 
* 높은 값을 가질수록 높은 우선순위가 할당된다.
* PriorityClass는 32bit(대략 20억)의 정수값을 가지는데 10억 보다 작거나 같은 값은 가진다.
* 20억~10억 사이의 값은 일반적으로 선점(preempttion)되거나 제거 되지 않아야 하는 중요한 시스템 pod을 위해 사용됨. 
* priorityclass는 globalDefault 필드를 갖는데, 이것은 priorityClassName이 지정안된 pod에 사용여부를 나타냄.(true인것은 시스템에 하나만 존재할 수 있음.)
* globalDefault가 설정된 PriorityClass가 없으면 priorityclassname이 지정되지 않은 pod은 priority가 0
* [상세내용](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/#priorityclass)  

```bash
$ cat priorityclass-for-gurantted.yml
---
apiVersion: v1
kind: List
items:
- apiVersion: scheduling.k8s.io/v1beta1
  kind: PriorityClass
  metadata:
    name: guaranteed
  value: 900000000
  globalDefault: false
  description: "priority for guaranteed pod"
- apiVersion: scheduling.k8s.io/v1beta1
  kind: PriorityClass
  metadata:
    name: bustable
  value: 1000
  globalDefault: false
  description: "priority for bustable pod"

# create
$ kc create -f priorityclass-for-quota.yaml
priorityclass.scheduling.k8s.io/guranteed created
priorityclass.scheduling.k8s.io/bustable created

$ kc get priorityclass
NAME                      VALUE        GLOBAL-DEFAULT   AGE
bustable                  1000         false            2s
guaranteed                900000000    false            2s
system-cluster-critical   2000000000   false            23d
system-node-critical      2000001000   false            23d
```

## Pod priority
* pod은 priorityclass를 이용하여 specific priority로 생성될수 있다.
* priority adminssion controller는 priorityClassName필드를 이용하고 priority의 정수 값을 이용한다. 
* 만약 지정한 priority class가 없으면 pod은 리젝된다. 

```bash
apiVersion: v1
kind: Pod
metadata:
  name: guaranteed-priority
  namespace: test-quota1
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10;done"]
    resources:
      requests:
        memory: "500Mi"
        cpu: "500m"
      limits:
        memory: "500Mi"
        cpu: "500m"
  priorityClassName: guaranteed
```

## ResouceQuota per PriorityClass
* quota spec안에 scopeSelect 필드를 이용하면, pod들의 리소스 사용량을 priority 기반으로 컨트롤 할수 있다.
* K8S 1.12부터 beta feature가 됨.(테스트 환경은 stg-prod-sg1)
* 이전 버전에서 alpha feature이므로 feturegate에서 true로 변경해줘야 함 [ResourceQuotaScopeSelectors feature gate version](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/)
* Quota spec에 scopeSelector 필드를 이용하여 우선순위에 따라 pod의 시스템 리소스 소비량을 제어 할수 있다.
* Quota spec에 scopeSelector를 지정할 경우, 지정한 quota와 일치되고 사용됩니다.
* Feature gate 에 ResourceQuotaScopeSelectors기능을 추가(1.12 이전 버전의 경우)해야 사용가능하다.
* pod에서는 priority class로 scopeSelector와 일치하는 것을 지정한다.
* pritority class 의 value는 32bit 정수값보다 작겨나(21억), 10억과 동일한 값을 가질수 있다.   

```bash
# namespace and quota per priorityclass
$ cat namespace-quota.yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: test-quota1
---
apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-guaranteed
    namespace: test-quota1
  spec:
    hard:
      cpu: "10"
      memory: 10Gi
      pods: "20"
    scopeSelector:
      matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["guaranteed"]
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-bustable
    namespace: test-quota1
  spec:
    hard:
      cpu: "10"
      memory: 10Gi
    scopeSelector:
      matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["bustable"]

# deployment for guaranteed
$ cat test-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment-on-guaranteed
  namespace: test-quota1
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "500m"
            memory: "500Mi"
          limits:
            cpu: "500m"
            memory: "500Mi"
      priorityClassName: guaranteed

# deployment for bustable
$ cat test-deployment-bustable.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment-on-bustable
  namespace: test-quota1
  labels:
    app: nginx2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx2
  template:
    metadata:
      labels:
        app: nginx2
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "1000m"
            memory: "1Gi"
      priorityClassName: bustable
```
   
```bash
$ kc create -f namespace-quota.yaml
$ kc create -f test-deployment-guaranteed.yaml
$ kc create -f test-deployment-bustable.yaml
$ kc describe quota -n test-quota1
Name:       pods-bustable
Namespace:  test-quota1
Resource    Used  Hard
--------    ----  ----
cpu         3     10
memory      3Gi   10Gi
#-> bustable pod 3개 잡힘

Name:       pods-guaranteed
Namespace:  test-quota1
Resource    Used    Hard
--------    ----    ----
cpu         1500m   10
memory      1500Mi  10Gi
pods        3       20
#-> guaranteed pod 3개 잡힘

$ kc get deployment,pod -n test-quota1
NAME                                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx-deployment-on-bustable     3/3     3            3           31s
deployment.extensions/nginx-deployment-on-guaranteed   3/3     3            3           10m

NAME                                                  READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-on-bustable-5cc9d55957-89jmt     1/1     Running   0          31s
pod/nginx-deployment-on-bustable-5cc9d55957-q7q6d     1/1     Running   0          31s
pod/nginx-deployment-on-bustable-5cc9d55957-tj42g     1/1     Running   0          31s
pod/nginx-deployment-on-guaranteed-768b9dbd59-hsgf2   1/1     Running   0          10m
pod/nginx-deployment-on-guaranteed-768b9dbd59-lj4hp   1/1     Running   0          10m
pod/nginx-deployment-on-guaranteed-768b9dbd59-ms2dp   1/1     Running   0          10m

```

  
```bash
# pod 상태 확인 
## guaranteed
$ kc get pod/nginx-deployment-on-guaranteed-768b9dbd59-hsgf2 -n test-quota1 -o yaml
apiVersion: v1
kind: Pod
metadata:
...
spec:
...
  priority: 900000000
  priorityClassName: guaranteed
...
status:
...
  qosClass: Guaranteed

## bustable
$ kc get pod/nginx-deployment-on-bustable-5cc9d55957-q7q6d -n test-quota1 -o yaml
apiVersion: v1
kind: Pod
metadata:
...
spec:
...
  priority: 1000
  priorityClassName: bustable
...
status:
...
  qosClass: Burstable
```


## Steps
* create priority class
* create namespace and quota with specifying scopeSelector
* create deployment and pod with priorityclassname

## 적용시 사전에 준비할 것들
* k8s 1.12 이상 올라가야 함(feature gate로 enable도 가능)
* pod의 limit/requests 기분 사용량 정의(flavor같은걸 사용해도 좋고, default limit를 지정할수도 있음)
* k8s cluster에 priorityClass생성(guaranteed, bustable)
* k8s user namespace에 quota 생성 및 scopeSelector로 priorityClass지정
