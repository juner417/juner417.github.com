---
layout: post
title: "ResouceQuota per PriorityClass"
date: 2019-03-30 19:00:00
categories: blog
author: juner
tags: [kubernetes]
comments: true
---
# Resource Quota test

## Prerequisite
* kube-apiserver ```--enable-admission-plugins=ResourceQuota```이 지정되어야 한다.
* [ResourceQuota에 priority class 적용](https://juner417.github.io/blog/resourcequota-per-priority/)
  * test-quota1 namespace에 prirityClass가 다른(guaranteed, bustable) 2개의 quota가 있다

## Test

### Case1: Quota 사용시 resource limit 까지 Pod이 문제 없이 생성 되는가?
#### scale out on limits
```
## deployment 확인
watch -n 2 "kubectl describe quota -n test-quota1"

$ kc get deploy,rs -n test-quota1
NAME                                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx-deployment-on-bustable     10/10   10           10          4m8s
deployment.extensions/nginx-deployment-on-guaranteed   5/5     5            5           4m12s

NAME                                                              DESIRED   CURRENT   READY   AGE
replicaset.extensions/nginx-deployment-on-bustable-5cc9d55957     10        10        10      4m9s
replicaset.extensions/nginx-deployment-on-guaranteed-85f6d7d97f   5         5         5       4m13s

## scale out
$ kc scale --replicas=10 deployment.extensions/nginx-deployment-on-bustable -n test-quota1
deployment.extensions/nginx-deployment-on-bustable scaled
$ kc scale --replicas=5 deployment.extensions/nginx-deployment-on-guaranteed -n test-quota1
deployment.extensions/nginx-deployment-on-guaranteed scaled

## quota describe
Name:       pods-bustable
Namespace:  test-quota1
Resource    Used  Hard
--------    ----  ----
cpu         10    10
memory      10Gi  10Gi
pods        10    20


Name:       pods-guaranteed
Namespace:  test-quota1
Resource    Used  Hard
--------    ----  ----
cpu         5     5
memory      5Gi   5Gi
pods        5     10
## -> limit 까지 만드는것 확인, pod 개수는 넘지 못함, 이유는 cpu/memory가 limit를 만족했기 때문에
```

#### 결과
* 잘됨
* 여긴 없지만 pod 개수만 늘리고 cpu/mem을 줄여서 pod 개수 limit를 먼저 만나도 더이상 생성 안됨.

### Case2: Quota 사용시 resource limit 넘어서는 Pod replica scale out이 들어올 경우
#### scale out over limits
```
## scale out
$ kc scale --replicas=20 deployment.extensions/nginx-deployment-on-bustable -n test-quota1
deployment.extensions/nginx-deployment-on-bustable scaled

$ kc scale --replicas=10 deployment.extensions/nginx-deployment-on-guaranteed -n test-quota1
deployment.extensions/nginx-deployment-on-guaranteed scaled

## 확인
Name:       pods-bustable
Namespace:  test-quota1
Resource    Used  Hard
--------    ----  ----
cpu         10    10
memory      10Gi  10Gi
pods        10    20


Name:       pods-guaranteed
Namespace:  test-quota1
Resource    Used  Hard
--------    ----  ----
cpu         5     5
memory      5Gi   5Gi
pods        5     10
NAME                                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx-deployment-on-bustable     10/20   10           10          9m55s
deployment.extensions/nginx-deployment-on-guaranteed   5/10    5            5           9m59s

NAME                                                              DESIRED   CURRENT   READY   AGE
replicaset.extensions/nginx-deployment-on-bustable-5cc9d55957     20        10        10      9m55s
replicaset.extensions/nginx-deployment-on-guaranteed-85f6d7d97f   10        5         5       9m59s
```

> quota 이상은 안늘어남...
> 그런데 kubectl 로 증가시 에러가 나지 않음
> deployment의 .spec은 바뀜

* event, deployment .status.reason FailedCreate에 exceeded quota 메세지 남음
```
## describe deployment
$ kc get deployment.extensions/nginx-deployment-on-bustable -o yaml -n test-quota1
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2019-03-29T09:14:05Z"
  generation: 3
  labels:
    app: nginx2
  name: nginx-deployment-on-bustable
  namespace: test-quota1
  resourceVersion: "23042336"
  selfLink: /apis/extensions/v1beta1/namespaces/test-quota1/deployments/nginx-deployment-on-bustable
  uid: 007b8f3e-5203-11e9-906b-fa165cc9a582
spec:
  progressDeadlineSeconds: 600
  replicas: 20
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx2
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx2
    spec:
      containers:
      - image: nginx:1.7.9
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources:
          requests:
            cpu: "1"
            memory: 1Gi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      priorityClassName: bustable
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 10
  conditions:
  - lastTransitionTime: "2019-03-29T09:14:05Z"
    lastUpdateTime: "2019-03-29T09:14:07Z"
    message: ReplicaSet "nginx-deployment-on-bustable-5cc9d55957" has successfully
      progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  - lastTransitionTime: "2019-03-29T09:21:41Z"
    lastUpdateTime: "2019-03-29T09:21:41Z"
    message: Deployment does not have minimum availability.
    reason: MinimumReplicasUnavailable
    status: "False"
    type: Available
  - lastTransitionTime: "2019-03-29T09:21:41Z"
    lastUpdateTime: "2019-03-29T09:21:41Z"
    message: 'pods "nginx-deployment-on-bustable-5cc9d55957-hvr9q" is forbidden: exceeded
      quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi,
      limited: cpu=10,memory=10Gi'
    reason: FailedCreate
    status: "True"
    type: ReplicaFailure
  observedGeneration: 3
  readyReplicas: 10
  replicas: 10
  unavailableReplicas: 10
  updatedReplicas: 10

$ kc get deployment.extensions/nginx-deployment-on-guaranteed -o yaml -n test-quota1
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2019-03-29T09:14:01Z"
  generation: 3
  labels:
    app: nginx
  name: nginx-deployment-on-guaranteed
  namespace: test-quota1
  resourceVersion: "23042385"
  selfLink: /apis/extensions/v1beta1/namespaces/test-quota1/deployments/nginx-deployment-on-guaranteed
  uid: fdd1da07-5202-11e9-906b-fa165cc9a582
spec:
  progressDeadlineSeconds: 600
  replicas: 10
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.7.9
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources:
          limits:
            cpu: "1"
            memory: 1Gi
          requests:
            cpu: "1"
            memory: 1Gi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      priorityClassName: guaranteed
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 5
  conditions:
  - lastTransitionTime: "2019-03-29T09:14:01Z"
    lastUpdateTime: "2019-03-29T09:14:03Z"
    message: ReplicaSet "nginx-deployment-on-guaranteed-85f6d7d97f" has successfully
      progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  - lastTransitionTime: "2019-03-29T09:21:48Z"
    lastUpdateTime: "2019-03-29T09:21:48Z"
    message: Deployment does not have minimum availability.
    reason: MinimumReplicasUnavailable
    status: "False"
    type: Available
  - lastTransitionTime: "2019-03-29T09:21:48Z"
    lastUpdateTime: "2019-03-29T09:21:48Z"
    message: 'pods "nginx-deployment-on-guaranteed-85f6d7d97f-d7clh" is forbidden:
      exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi,
      limited: cpu=5,memory=5Gi'
    reason: FailedCreate
    status: "True"
    type: ReplicaFailure
  observedGeneration: 3
  readyReplicas: 5
  replicas: 5
  unavailableReplicas: 5
  updatedReplicas: 5
```

* event log
```
108s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-wr6ch" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
108s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-cvgqc" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
108s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-dbsfg" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
108s        Normal    ScalingReplicaSet   Deployment   Scaled up replica set nginx-deployment-on-bustable-5cc9d55957 to 20
108s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-hvr9q" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
107s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-47tdz" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
107s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-nd7hb" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
107s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-95s6h" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
107s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-pc5qm" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
106s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-mlblq" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
26s         Warning   FailedCreate        ReplicaSet   (combined from similar events): Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-gffw7" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
101s        Normal    ScalingReplicaSet   Deployment   Scaled up replica set nginx-deployment-on-guaranteed-85f6d7d97f to 10
101s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-hk5fx" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
101s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-5f8mh" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
101s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-khl79" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
101s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-vdq7x" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
101s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-f6gln" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
101s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-8g9hb" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
101s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-d7clh" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
100s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-zks4j" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
100s        Warning   FailedCreate        ReplicaSet   Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-94x4x" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
19s         Warning   FailedCreate        ReplicaSet   (combined from similar events): Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-bmjbs" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
```

* kube-controller-manager log
```
kube-controller-manager- kube-controller-manager E0329 09:22:22.925523       1 replica_set.go:450] Sync "test-quota1/nginx-deployment-on-bustable-5cc9d55957" failed with pods "nginx-deployment-on-bustable-5cc9d55957-2fps4" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
kube-controller-manager- kube-controller-manager I0329 09:22:22.925563       1 event.go:221] Event(v1.ObjectReference{Kind:"ReplicaSet", Namespace:"test-quota1", Name:"nginx-deployment-on-bustable-5cc9d55957", UID:"007c166b-5203-11e9-906b-fa165cc9a582", APIVersion:"apps/v1", ResourceVersion:"23042334", FieldPath:""}): type: 'Warning' reason: 'FailedCreate' Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-2fps4" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
kube-controller-manager- kube-controller-manager E0329 09:22:29.374835       1 replica_set.go:450] Sync "test-quota1/nginx-deployment-on-guaranteed-85f6d7d97f" failed with pods "nginx-deployment-on-guaranteed-85f6d7d97f-tx9f6" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
kube-controller-manager- kube-controller-manager I0329 09:22:29.375054       1 event.go:221] Event(v1.ObjectReference{Kind:"ReplicaSet", Namespace:"test-quota1", Name:"nginx-deployment-on-guaranteed-85f6d7d97f", UID:"fdd38c92-5202-11e9-906b-fa165cc9a582", APIVersion:"apps/v1", ResourceVersion:"23042380", FieldPath:""}): type: 'Warning' reason: 'FailedCreate' Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-tx9f6" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
kube-controller-manager- kube-controller-manager E0329 09:22:47.357220       1 memcache.go:135] couldn't get resource list for custom.metrics.k8s.io/v1beta1: <nil>
kube-controller-manager- kube-controller-manager I0329 09:23:03.894156       1 event.go:221] Event(v1.ObjectReference{Kind:"ReplicaSet", Namespace:"test-quota1", Name:"nginx-deployment-on-bustable-5cc9d55957", UID:"007c166b-5203-11e9-906b-fa165cc9a582", APIVersion:"apps/v1", ResourceVersion:"23042334", FieldPath:""}): type: 'Warning' reason: 'FailedCreate' Error creating: pods "nginx-deployment-on-bustable-5cc9d55957-gffw7" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
kube-controller-manager- kube-controller-manager E0329 09:23:03.894177       1 replica_set.go:450] Sync "test-quota1/nginx-deployment-on-bustable-5cc9d55957" failed with pods "nginx-deployment-on-bustable-5cc9d55957-gffw7" is forbidden: exceeded quota: pods-bustable, requested: cpu=1,memory=1Gi, used: cpu=10,memory=10Gi, limited: cpu=10,memory=10Gi
kube-controller-manager- kube-controller-manager E0329 09:23:10.337165       1 replica_set.go:450] Sync "test-quota1/nginx-deployment-on-guaranteed-85f6d7d97f" failed with pods "nginx-deployment-on-guaranteed-85f6d7d97f-bmjbs" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
kube-controller-manager- kube-controller-manager I0329 09:23:10.337594       1 event.go:221] Event(v1.ObjectReference{Kind:"ReplicaSet", Namespace:"test-quota1", Name:"nginx-deployment-on-guaranteed-85f6d7d97f", UID:"fdd38c92-5202-11e9-906b-fa165cc9a582", APIVersion:"apps/v1", ResourceVersion:"23042380", FieldPath:""}): type: 'Warning' reason: 'FailedCreate' Error creating: pods "nginx-deployment-on-guaranteed-85f6d7d97f-bmjbs" is forbidden: exceeded quota: pods-guaranteed, requested: cpu=1,memory=1Gi, used: cpu=5,memory=5Gi, limited: cpu=5,memory=5Gi
```

#### 결과
* limit를 넘어선 kubectl, k8s api 요청 모두 적용은 된다(deployment .spec.replicas)
* 하지만 kube-controller-manager에서 exceeded quota가 나고 event발생
* deployment .status.conditions에 failedcreate 값에 event 메세지가 들어감
* 위 이벤트를 가지고 client쪽에 이벤트를 보낼 방법 필요(현재 가장 좋은건 validating admission webhook)


### Case3: limit에 걸려 더이상 생성이 안될때 quota 수정시 동작확인
#### edit or patch quota
```
## edit quota
$ kc edit quota/pods-bustable -n test-quota1
$ kc edit quota/pods-guaranteed -n test-quota1


## 일반 quota와 다르게 priorityClass를 지정한 Quota는 quota limit을 변경해도 알아서 늘어나지 않는다.
$ kc scale --replicas=10 deployment.extensions/nginx-deployment-on-guaranteed -n test-quota1
deployment.extensions/nginx-deployment-on-guaranteed scaled

$ kc get ev -n test-quota1

$ kc scale --replicas=9 deployment.extensions/nginx-deployment-on-guaranteed -n test-quota1
deployment.extensions/nginx-deployment-on-guaranteed scaled
#-> 이때 부터 늘어남
$ kc scale --replicas=10 deployment.extensions/nginx-deployment-on-guaranteed -n test-quota1
deployment.extensions/nginx-deployment-on-guaranteed scaled

$ kc scale --replicas=20 deployment.extensions/nginx-deployment-on-bustable -n test-quota1
deployment.extensions/nginx-deployment-on-bustable scaled
# -> 변경 안됨

$ kc scale --replicas=19 deployment.extensions/nginx-deployment-on-bustable -n test-quota1
deployment.extensions/nginx-deployment-on-bustable scaled
#-> 이때부터 늘어남
$ kc scale --replicas=20 deployment.extensions/nginx-deployment-on-bustable -n test-quota1
deployment.extensions/nginx-deployment-on-bustable scaled

## describe quota
Name:       pods-bustable
Namespace:  test-quota1
Resource    Used  Hard
--------    ----  ----
cpu         20    20
memory      20Gi  20Gi
pods        20    20


Name:       pods-guaranteed
Namespace:  test-quota1
Resource    Used  Hard
--------    ----  ----
cpu         10    10
memory      10Gi  10Gi
pods        10    10
NAME                                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx-deployment-on-bustable     20/20   20           20          43m
deployment.extensions/nginx-deployment-on-guaranteed   10/10   10           10          43m

NAME                                                              DESIRED   CURRENT   READY   AGE
replicaset.extensions/nginx-deployment-on-bustable-5cc9d55957     20        20        20      43m
replicaset.extensions/nginx-deployment-on-guaranteed-85f6d7d97f   10        10        10      43m

NAME                                                  READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-on-bustable-5cc9d55957-4gn8x     1/1     Running   0          65s
pod/nginx-deployment-on-bustable-5cc9d55957-5fcz9     1/1     Running   0          41m
pod/nginx-deployment-on-bustable-5cc9d55957-6rc7d     1/1     Running   0          74s
pod/nginx-deployment-on-bustable-5cc9d55957-76vwx     1/1     Running   0          41m
pod/nginx-deployment-on-bustable-5cc9d55957-7rlrh     1/1     Running   0          41m
pod/nginx-deployment-on-bustable-5cc9d55957-894q8     1/1     Running   0          74s
pod/nginx-deployment-on-bustable-5cc9d55957-982mq     1/1     Running   0          74s
pod/nginx-deployment-on-bustable-5cc9d55957-9txcw     1/1     Running   0          41m
pod/nginx-deployment-on-bustable-5cc9d55957-9wgml     1/1     Running   0          43m
pod/nginx-deployment-on-bustable-5cc9d55957-b2m4q     1/1     Running   0          41m
pod/nginx-deployment-on-bustable-5cc9d55957-bd2tc     1/1     Running   0          41m
pod/nginx-deployment-on-bustable-5cc9d55957-fkxjh     1/1     Running   0          74s
pod/nginx-deployment-on-bustable-5cc9d55957-jwgnm     1/1     Running   0          43m
pod/nginx-deployment-on-bustable-5cc9d55957-k8sh2     1/1     Running   0          74s
pod/nginx-deployment-on-bustable-5cc9d55957-lhzbh     1/1     Running   0          74s
pod/nginx-deployment-on-bustable-5cc9d55957-mqtj4     1/1     Running   0          74s
pod/nginx-deployment-on-bustable-5cc9d55957-qpsfx     1/1     Running   0          43m
pod/nginx-deployment-on-bustable-5cc9d55957-s6bnv     1/1     Running   0          74s
pod/nginx-deployment-on-bustable-5cc9d55957-shvhc     1/1     Running   0          41m
pod/nginx-deployment-on-bustable-5cc9d55957-xgcq7     1/1     Running   0          74s
pod/nginx-deployment-on-guaranteed-85f6d7d97f-42b7g   1/1     Running   0          43m
pod/nginx-deployment-on-guaranteed-85f6d7d97f-f5svr   1/1     Running   0          2m20s
pod/nginx-deployment-on-guaranteed-85f6d7d97f-gf7bf   1/1     Running   0          43m
pod/nginx-deployment-on-guaranteed-85f6d7d97f-k2xb9   1/1     Running   0          2m27s
pod/nginx-deployment-on-guaranteed-85f6d7d97f-mrhzd   1/1     Running   0          2m27s
pod/nginx-deployment-on-guaranteed-85f6d7d97f-n86lc   1/1     Running   0          43m
pod/nginx-deployment-on-guaranteed-85f6d7d97f-p9ft7   1/1     Running   0          40m
pod/nginx-deployment-on-guaranteed-85f6d7d97f-sfhh5   1/1     Running   0          40m
pod/nginx-deployment-on-guaranteed-85f6d7d97f-t987m   1/1     Running   0          2m27s
pod/nginx-deployment-on-guaranteed-85f6d7d97f-z4gcq   1/1     Running   0          2m27s
```
#### 결과
* quota를 증가 시켜도 안늘어남...
* quota에 scopeSelector를 이용해서 priorityClass를 지정한것과 아닌것이 다르게 동작한다.(이거 좀 버그 같다...)
* 버그 같은 이유는 priorityClass를 지정하지 않은 quota는 hard limit를 변경하면 늘어나기 때문이다.
* 추가로 동일한 개수로 scale을 할 경우에도 늘어나지 않는다.(kubectl, api 모두) 이부분도 의심

## 결론.
* namespace에 resource quota를 지정하면(with priorityclass) 넘어설수는 없다.
* 하지만 deployment,replicaset .spec(desired) 의 값은 바뀐다(.status는 quota를 넘어설수는 없다.)
* quota를 침범할 경우 event가 발생한다. 그리고 deployment status에도 이벤트가 있다. (exceeded quota)
* client쪽에 메세지를 주기위해서(exceeded quota) 별도의 validating admission webbook이 필요하다.

