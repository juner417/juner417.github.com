---
layout: post
title: "ETCD backup and restore"
date: 2018-03-06 18:00:00
categories: blog
author: juner
tags: [kubernetes, etcd]
comments: true
---
# ETCD backup and restore

## Summary
### 목적
 - etcd v2, v3의 backup, restore의 차이가 있음, 우리는 두 버전 모두 사용하고 있어서 모두 복구를 해야함. 테스트 필요  

### 테스트 방식
 - kubernetes cluster 생성 후 etcd data backup(v2, v3)
 - etcd v2, v3의 backup data 를 가지고 restore를 했을 때, 두 버전의 저장된 key store가 모두 복구 가능한지 확인
 - restore etcd를 가지고 kubernetes cluster가 정상 동작 하는지 확인

### 결과정리   
* v2, v3 backup 데이터를 가지고 restore 가능
* v3의 경우, db 파일의 위치를 변경해 주면 모두 복구 가능
* restore된 etcd를 가지고 kubernetes cluster 정상 동작 가능
* 하지만 component들의 재시작이 필요  

## 테스트 환경
- aws ec2 node3개
- 초반 테스트 데이터는 k8s 클러스터 데이터(kubespray default 배포 v2.2.0 기준)
- 데이터는   
  - v2 : flannel  
  - v3 : k8s
- 마지막 복구 테스트는 live cluster 데이터 기준

## 출처
[etcd v2 backup and restore](https://github.com/coreos/etcd/blob/master/Documentation/v2/admin_guide.md#disaster-recovery)  
[etcd v3 backup and restore](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/recovery.md)  
[etcd tip](https://www.mirantis.com/blog/everything-you-ever-wanted-to-know-about-using-etcd-with-kubernetes-v1-6-but-were-afraid-to-ask/)  
[etcd admin guide](https://coreos.com/etcd/docs/latest/v2/admin_guide.html)  

## 테스트 준비
---  
### step1. vm 확인, etcd 배포
#### 1. ansible inventory 확인
```bash  
$ cat sites/test_etcd/inventory/inventory.cfg
[all]
node1 ansible_ssh_host=172.31.23.192  ip=172.31.23.192
node2 ansible_ssh_host=172.31.17.103  ip=172.31.17.103
node3 ansible_ssh_host=172.31.23.132  ip=172.31.23.132

[kube-master]
node1
node2

[etcd]
node1
node2
node3

[kube-node]
node3
```

#### 2. 배포후 확인   
```bash
root@node1:/etc/kubernetes# kubectl get nodes
NAME      STATUS                     AGE       VERSION
node1     Ready,SchedulingDisabled   14m       v1.7.4
node2     Ready,SchedulingDisabled   14m       v1.7.4
node3     Ready                      14m       v1.7.4

root@node1:/etc/kubernetes# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-2               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true”}
```   


#### 3. etcd 확인 v2, v3 모두 있음
```bash
root@node1:/etc/kubernetes# ETCDCTL_API=2 etcdctl --endpoints https://172.31.23.192:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE ls /
/cluster.local

root@node1:/etc/kubernetes# ETCDCTL_API=3 etcdctl --endpoints https://172.31.23.192:2379 --cacert $ETCD_TRUSTED_CA_FILE --cert $ETCD_CERT_FILE --key $ETCD_KEY_FILE get / --prefix --keys-only
/registry/apiregistration.k8s.io/apiservices/v1.
/registry/apiregistration.k8s.io/apiservices/v1.authentication.k8s.io
…
```


#### 4. etcd data dir 확인   
```bash
root@node1:~# ls -l $ETCD_DATA_DIR/member/
total 8
drwx------ 2 root root 4096 Mar  5 06:55 snap
drwx------ 2 root root 4096 Mar  5 06:55 wal

root@node2:~# ls -l $ETCD_DATA_DIR/member/
total 8
drwx------ 2 root root 4096 Mar  5 06:55 snap
drwx------ 2 root root 4096 Mar  5 06:55 wal

root@node3:~# ls -l $ETCD_DATA_DIR/member/
total 8
drwx------ 2 root root 4096 Mar  5 06:55 snap
drwx------ 2 root root 4096 Mar  5 06:55 wal
```  


#### 5. etcd key 확인 v2, v3
```bash
root@node1:~# ETCDCTL_API=2 etcdctl --endpoints https://172.31.23.192:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE ls -r /
/cluster.local
/cluster.local/network
/cluster.local/network/config
/cluster.local/network/subnets
/cluster.local/network/subnets/10.233.68.0-24
/cluster.local/network/subnets/10.233.88.0-24
/cluster.local/network/subnets/10.233.90.0-24

root@node1:~#  ls -r / | wc -l
7

root@node1:~# ETCDCTL_API=3 etcdctl --endpoints https://172.31.23.192:2379 --cacert $ETCD_TRUSTED_CA_FILE --cert $ETCD_CERT_FILE --key $ETCD_KEY_FILE get / --prefix --keys-only | wc -l
898
```

#### 6. etcd  member list 확인  
```bash
root@node1:~# ETCDCTL_API=2 etcdctl --endpoints https://172.31.23.192:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE cluster-health
member 1a7bb8b2e700533 is healthy: got healthy result from https://172.31.17.103:2379
member 5ceab3c92ca18c9c is healthy: got healthy result from https://172.31.23.132:2379
member 97c46e859e451551 is healthy: got healthy result from https://172.31.23.192:2379
cluster is healthy

root@node1:~# ETCDCTL_API=2 etcdctl --endpoints https://172.31.23.192:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE member list
1a7bb8b2e700533: name=etcd2 peerURLs=https://172.31.17.103:2380 clientURLs=https://172.31.17.103:2379 isLeader=true
5ceab3c92ca18c9c: name=etcd3 peerURLs=https://172.31.23.132:2380 clientURLs=https://172.31.23.132:2379 isLeader=false
97c46e859e451551: name=etcd1 peerURLs=https://172.31.23.192:2380 clientURLs=https://172.31.23.192:2379 isLeader=false

ETCDCTL_API=3 etcdctl --endpoints https://172.31.23.192:2379 --cacert $ETCD_TRUSTED_CA_FILE --cert $ETCD_CERT_FILE --key $ETCD_KEY_FILE member list
1a7bb8b2e700533, started, etcd2, https://172.31.17.103:2380, https://172.31.17.103:2379
5ceab3c92ca18c9c, started, etcd3, https://172.31.23.132:2380, https://172.31.23.132:2379
97c46e859e451551, started, etcd1, https://172.31.23.192:2380, https://172.31.23.192:2379
```  


## 테스트
---  
### step2. etcd backup
#### 1. data backup v2, v3
```bash
source /etc/etcd.env
ETCDCTL_API=2 etcdctl --endpoints $ETCD_LISTEN_CLIENT_URLS --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE backup --data-dir $ETCD_DATA_DIR --backup-dir /data/etcd.backup
ETCDCTL_API=3 etcdctl --endpoints $ETCD_LISTEN_CLIENT_URLS --cacert $ETCD_TRUSTED_CA_FILE --cert $ETCD_CERT_FILE --key $ETCD_KEY_FILE snapshot save /data/etcd.backup/snapshot.db

v2 wc -l : 7
v3 wc -l : 270
```  


#### 2. check the data snapshot
```bash
root@node1:~# ls -lR /data/etcd.backup/
/data/etcd.backup/:
total 4452
drwx------ 4 root root    4096 Mar  5 08:07 member
-rw-r--r-- 1 root root 4550688 Mar  5 08:07 snapshot.db

/data/etcd.backup/member:
total 8
drwx------ 2 root root 4096 Mar  5 08:07 snap
drwx------ 2 root root 4096 Mar  5 08:07 wal

/data/etcd.backup/member/snap:
total 0

/data/etcd.backup/member/wal:
total 62500
-rw------- 1 root root 64000000 Mar  5 08:07 0000000000000000-0000000000000000.wal
```  

#### 3. etcd down & move current data dir
```bash
systemctl stop etcd
mv /data/var/lib/etcd /data/var/lib/etcd.old
mv /data/etct.backup/ /data/var/lib/etcd

root@node1:~# mv /data/var/lib/etcd /data/var/lib/etcd.old
root@node1:~# ls -al /data/var/lib/
total 16
drwx--x--x  4 root root 4096 Mar  5 08:11 .
drwx--x--x  4 root root 4096 Mar  5 04:21 ..
drwx--x--x 11 root root 4096 Mar  5 04:21 docker
drwxr-xr-x  3 etcd etcd 4096 Mar  5 06:55 etcd.old
```  



### step3. etcd restore v3 snapshot.db를 이용하여 restore

#### 1. v3 snapshot db restore
```bash
ETCDCTL_API=3 etcdctl snapshot restore /data/etcd.backup/snapshot.db \
  --name etcd1 \
  --data-dir=“$ETCD_DATA_DIR" \
  --initial-cluster="etcd1=https://172.31.23.192:2380,etcd2=https://172.31.17.103:2380,etcd3=https://172.31.23.132:2380" \
  --initial-cluster-token=k8s_etcd \
  --initial-advertise-peer-urls="https://172.31.23.192:2380” \

root@node1:~# ETCDCTL_API=3 etcdctl snapshot restore /data/etcd.backup/snapshot.db   --name etcd1   --data-dir /data/var/lib/etcd \
  --initial-cluster etcd1=https://172.31.23.192:2380,etcd2=https://172.31.17.103:2380,etcd3=https://172.31.23.132:2380 \
  --initial-cluster-token k8s_etcd \
  --initial-advertise-peer-urls https://172.31.23.192:2380
2018-03-05 08:23:05.551770 I | mvcc: restore compact to 6766
2018-03-05 08:23:05.557023 I | etcdserver/membership: added member 1a7bb8b2e700533 [https://172.31.17.103:2380] to cluster 4a6a738e5680df92
2018-03-05 08:23:05.557051 I | etcdserver/membership: added member 5ceab3c92ca18c9c [https://172.31.23.132:2380] to cluster 4a6a738e5680df92
2018-03-05 08:23:05.557070 I | etcdserver/membership: added member 97c46e859e451551 [https://172.31.23.192:2380] to cluster 4a6a738e5680df92

root@node1:~# ls -al /data/var/lib/etcd
total 12
drwx------ 4 root root 4096 Mar  5 08:23 member

root@node1:~# ls -al /data/var/lib/etcd/member/
total 16
drwx------ 2 root root 4096 Mar  5 08:23 snap
drwx------ 2 root root 4096 Mar  5 08:23 wal

root@node1:~# ls -al /data/var/lib/etcd/member/wal/
total 62508
-rw------- 1 root root 64000000 Mar  5 08:23 0000000000000000-0000000000000000.wal
```


#### 2. etcd start
```bash
vi /etc/etcd.env
# add below
ETCD_FORCE_NEW_CLUSTER=true
systemctl start etcd
```

#### 3. check the process  
```bash  
systemctl status etc

root@node1:~# systemctl status etcd
● etcd.service - etcd docker wrapper
   Loaded: loaded (/etc/systemd/system/etcd.service; enabled; vendor p
   Active: active (running) since Mon 2018-03-05 08:25:58 UTC; 1min 3s
  Process: 4467 ExecStop=/usr/bin/docker stop etcd1 (code=exited, stat
  Process: 5851 ExecStartPre=/usr/bin/docker rm -f etcd1 (code=exited,
 Main PID: 5858 (etcd)
    Tasks: 9
   Memory: 6.4M
      CPU: 27ms
   CGroup: /system.slice/etcd.service
           ├─5858 /bin/bash /usr/local/bin/etcd
           └─5860 /usr/bin/docker run --restart=on-failure:5 --env-fil
...

root@node1:~# ETCDCTL_API=3 etcdctl --endpoints https://172.31.23.192:2379 --cacert $ETCD_TRUSTED_CA_FILE --cert $ETCD_CERT_FILE --key $ETCD_KEY_FILE member list
97c46e859e451551, started, etcd1, https://172.31.23.192:2380, https://172.31.23.192:2379
—> id가 안바뀜...
```  

#### 4. etcd key 확인   
```bash  
watch "ETCDCTL_API=2 etcdctl --endpoints https://172.31.23.192:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE ls -r / | wc -l; ETCDCTL_API=3 etcdctl --endpoints https://172.31.23.192:2379 --cacert $ETCD_TRUSTED_CA_FILE --cert $ETCD_CERT_FILE --key $ETCD_KEY_FILE get / --prefix --keys-only | wc -l"
Every 2.0s: ETCDCTL_API=2 etcdctl --endpo...  Mon Mar  5 08:29:32 2018
v2 wc -l : 0  <— v2는 없음, v3의 백업을 가지고 하면 v2의 key는 살아나지 않는다.
v3 wc -l : 270
```  



### step4. etcd restore v2 snapshot 파일을 이용하여)

#### 1. v2 snapshot restore  
```bash
root@node1:~# mv /data/var/lib/etcd /data/var/lib/etcd.v3

root@node1:~# ls -l /data/var/lib/
total 12
drwx--x--x 11 root root 4096 Mar  5 04:21 docker
drwxr-xr-x  3 etcd etcd 4096 Mar  5 06:55 etcd.old
drwx------  3 root root 4096 Mar  5 08:25 etcd.v3

root@node1:~# cp -r /data/etcd.backup/ /data/var/lib/etcd

root@node1:~# ls -al /data/var/lib/etcd
total 4460
drwx------ 3 root root    4096 Mar  5 08:36 .
drwx--x--x 6 root root    4096 Mar  5 08:36 ..
drwx------ 4 root root    4096 Mar  5 08:36 member
-rw-r--r-- 1 root root 4550688 Mar  5 08:36 snapshot.db


root@node1:~# systemctl start etcd
root@node1:~# systemctl status etcd
● etcd.service - etcd docker wrapper
   Loaded: loaded (/etc/systemd/system/etcd.service; enabled; vendor p
   Active: active (running) since Mon 2018-03-05 08:37:37 UTC; 3s ago
  Process: 7593 ExecStop=/usr/bin/docker stop etcd1 (code=exited, stat
  Process: 8245 ExecStartPre=/usr/bin/docker rm -f etcd1 (code=exited,
 Main PID: 8252 (etcd)
    Tasks: 9
   Memory: 6.4M
      CPU: 27ms
   CGroup: /system.slice/etcd.service
           ├─8252 /bin/bash /usr/local/bin/etcd
           └─8255 /usr/bin/docker run --restart=on-failure:5 --env-fil

root@node1:~# ETCDCTL_API=3 etcdctl --endpoints https://172.31.23.192:2379 --cacert $ETCD_TRUSTED_CA_FILE --cert $ETCD_CERT_FILE --key $ETCD_KEY_FILE member list
61f533950a01, started, etcd1,http://localhost:2380, https://172.31.23.192:2379
—> member id가 바뀜, 그리고 env에 advertise url을 IP로 줬는데 왜 localhost로 된건지 확인필요
```  

#### 2. peer url update  
```bash  
ETCDCTL_API=2 etcdctl --endpoints https://172.31.23.192:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE member update 61f533950a01 https://172.31.23.192:2380

root@node1:~# ls -al /data/var/lib/etcd
total 4460
drwx------ 4 root root    4096 Mar  5 08:37 member
-rw-r--r-- 1 root root 4550688 Mar  5 08:36 snapshot.db
root@node1:~# ls -al /data/var/lib/etcd/member/
total 16
drwx------ 2 root root 4096 Mar  5 08:37 snap
drwx------ 2 root root 4096 Mar  5 08:37 wal
root@node1:~# ls -al /data/var/lib/etcd/member/snap/
total 5232
-rw------- 1 root root 16805888 Mar  5 08:42 db
root@node1:~# ls -al /data/var/lib/etcd/member/wal/
total 125008
-rw------- 1 root root 64000000 Mar  5 08:45 0000000000000000-0000000000000000.wal
-rw------- 1 root root 64000000 Mar  5 08:37 0.tmp

root@node1:~# watch "ETCDCTL_API=2 etcdctl --endpoints https://172.31.23.192:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE ls -r / | wc -l; ETCDCTL_API=3 etcdctl --endpoints https://172.31.23.192:2379 --cacert $ETCD_TRUSTED_CA_FILE --cert $ETCD_CERT_FILE --key $ETCD_KEY_FILE get / --prefix --keys-only | wc -l”

Every 2.0s: ETCDCTL_API=2 etcdctl --endpo...  Mon Mar  5 08:39:28 2018
v2 wc -l : 7
v3 wc -l : 270
```   
 > 둘다 살아남…   
 하지만 아래의 github 이슈를 확인해 보니 정확히 backup data 를 가지고 한것이 아니라는 코멘트가 있음 추가 테스트 필요 (wal파일에 저장된 v3 데이터가 복구된것이라고 이야기 함, 이것은 backup 데이터가 아니다.)  
 [github issue link](https://github.com/coreos/etcd/issues/7002)    


#### 3. 나머지 노드 등록하기
```bash  
ETCDCTL_API=2 etcdctl --endpoints https://172.31.23.192:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE member add etcd2 https://172.31.17.103:2380
ETCDCTL_API=2 etcdctl --endpoints https://172.31.23.192:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE member add etcd3 https://172.31.23.132:2380

root@node1:~# ETCDCTL_API=2 etcdctl --endpoints https://172.31.23.192:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE member list
61f533950a01: name=etcd1 peerURLs=https://172.31.23.192:2380 clientURLs=https://172.31.23.192:2379 isLeader=true
29a5417fde632c9f: name=etcd2 peerURLs=https://172.31.17.103:2380 clientURLs=https://172.31.17.103:2379 isLeader=false
d3fdb7c22e878801: name=etcd3 peerURLs=https://172.31.23.132:2380 clientURLs=https://172.31.23.132:2379 isLeader=false

root@node1:~# ETCDCTL_API=2 etcdctl --endpoints https://172.31.23.192:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE cluster-health
member 61f533950a01 is healthy: got healthy result from https://172.31.23.192:2379
member 29a5417fde632c9f is healthy: got healthy result from https://172.31.17.103:2379
member d3fdb7c22e878801 is healthy: got healthy result from https://172.31.23.132:2379
cluster is healthy
```



### step5. etcd restore snapshot.db 제외하고...

#### 1. 다시 복원을 위해 삭제   
```bash  
root@node2:~# cp -r /data/etcd.backup/ /data/var/lib/etcd

root@node2:~# ls -al /data/var/lib/etcd/
total 4460
drwx------ 3 root root    4096 Mar  5 12:13 .
drwx--x--x 5 root root    4096 Mar  5 12:13 ..
drwx------ 4 root root    4096 Mar  5 12:13 member
-rw-r--r-- 1 root root 4550688 Mar  5 12:13 snapshot.db

root@node2:~# rm /data/var/lib/etcd/snapshot.db

root@node2:~# ls -al /data/var/lib/etcd/
total 12
drwx------ 3 root root 4096 Mar  5 12:14 .
drwx--x--x 5 root root 4096 Mar  5 12:13 ..
drwx------ 4 root root 4096 Mar  5 12:13 member
```  

#### 2. start etcd  
```bash  
root@node2:~# systemctl restart etcd
root@node2:~# systemctl status etcd
● etcd.service - etcd docker wrapper
   Loaded: loaded (/etc/systemd/system/etcd.service; enabled; vendor
   Active: active (running) since Mon 2018-03-05 12:15:04 UTC; 6s ago
  Process: 8958 ExecStop=/usr/bin/docker stop etcd2 (code=exited, sta
  Process: 9219 ExecStartPre=/usr/bin/docker rm -f etcd2 (code=exited
 Main PID: 9226 (etcd)
    Tasks: 8
   Memory: 4.4M
      CPU: 29ms
   CGroup: /system.slice/etcd.service
           ├─9226 /bin/bash /usr/local/bin/etcd
           └─9229 /usr/bin/docker run --restart=on-failure:5 --env-fi
```  

#### 3. 확인   
```bash  
root@node2:~# ETCDCTL_API=2 etcdctl --endpoints https://172.31.17.103:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE member list
61f533950a01: name=etcd2 peerURLs=https://172.31.17.103:2380 clientURLs=https://172.31.17.103:2379 isLeader=true

root@node2:~# ETCDCTL_API=2 etcdctl --endpoints https://172.31.17.103:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE ls -r / | wc -l; ETCDCTL_API=3 etcdctl --endpoints https://172.31.17.103:2379 --cacert $ETCD_TRUSTED_CA_FILE --cert $ETCD_CERT_FILE --key $ETCD_KEY_FILE get / --prefix --keys-only | wc -l
7
282
```  
 > snapshot.db가 없어도 v3 의 데이터가 있음, 위 github이슈가 맞는 이야기.  
 wal 파일에 있는 v3의 데이터가 있는 것으로 backup data는 아님  
 실제로 v3의 백업데이터에서 key store를 복구 하려면 snapshot.db를 $ETCD_DATA_DIR/member/snap/db 로 변경하고 실행해야함...


#### 4. 나머지 노드 추가   
```  
ETCDCTL_API=2 etcdctl --endpoints https://172.31.17.103:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE member add etcd1 https://172.31.23.192:2380

ETCDCTL_API=2 etcdctl --endpoints https://172.31.17.103:2379 --ca-file $ETCD_TRUSTED_CA_FILE --cert-file $ETCD_CERT_FILE --key-file $ETCD_KEY_FILE member add etcd3 https://172.31.23.132:2380
```  

#### 5. 노드 확인   
```  
root@node1:~# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health": "true"}
etcd-0               Healthy   {"health": "true"}
etcd-2               Healthy   {"health": "true”}

NAMESPACE     NAME                              READY     STATUS    RESTARTS   AGE
kube-system   flannel-node1                     1/1       Running   1          5h
kube-system   flannel-node2                     1/1       Running   1          5h
kube-system   flannel-node3                     1/1       Running   1          5h
kube-system   heapster-v1.4.0-587193324-6k6x4   2/2       Running   0          5h
kube-system   kube-apiserver-node1              1/1       Running   0          5h
kube-system   kube-apiserver-node2              1/1       Running   0          5h
kube-system   kube-controller-manager-node1     1/1       Running   0          5h
kube-system   kube-controller-manager-node2     1/1       Running   0          5h
kube-system   kube-proxy-node1                  1/1       Running   1          5h
kube-system   kube-proxy-node2                  1/1       Running   1          5h
kube-system   kube-proxy-node3                  1/1       Running   1          5h
kube-system   kube-scheduler-node1              1/1       Running   0          5h
kube-system   kube-scheduler-node2              1/1       Running   0          5h
kube-system   nginx-proxy-node3                 1/1       Running   1          5h

member 61f533950a01 is healthy: got healthy result from https://172.31.17.103:2379
member 33ed48cf8292ea7f is healthy: got healthy result from https://172.31.23.132:2379
member 87c04d4f9d7d8fc1 is healthy: got healthy result from https://172.31.23.192:2379
cluster is healthy
```  

---
### 테스트시 확인된 이슈들
---  
#### issue1)  
etcd 가 장애가 난 상황에서 restore를 하면 master kubernetes component들과 노드의 kubelet을 재시작 해줘야 함, 안하면 controller, scheduler, kubelet이 정상적으로 동작하지 않아서 pod을 생성하지 못한다.(정확히 말하면 master election의 lock파일을 제어하지 못해서 scheduling이 되지 않음)

#### issue2)  
백업한 데이터로 restore 하려면 최초 한대에서 ETCD_FORCE_NEW_CLUSTER=true 옵션을 /etc/etcd.env에 추가하고, etcd initial cluster를 한개로 한뒤 실행하여, 데이터를 확인한다.
그리고 한대 씩 차례대로 member add를 하고 나오는 결과를 /etc/etcd.env에 추가한뒤 etcd를 실행하면 된다.

#### issue3)  
restore할 때 최초 노드로 클러스터를 새롭게 생성하여 member list를 확인하면 peerurl이 localhost로 되어있다. 이것을 member update 로 ip로 수정해야 한다.(이유는 잘 모르겠다... 좀더 찾와봐야 함)

#### issue4)  
v3, v2 모두 백업을 하지만, v3로 백업한 snapshot.db를 이용하여 restore하면 v2의 데이터는 복구가 안된다. v2백업데이터로 복구하면 v3의 데이터가 살아 있으나, 기존에 v2의 백업된 wal파일에 있는 데이터가 복구된 것으로 진정한 백업 데이터는 아니다. [이슈](https://github.com/coreos/etcd/issues/7002)를 뒤져봐도 개발은 되고 있으나 기능이 추가되진 않았다 (--with-v3라는 옵션이 추가될 것이다)  

#### issue5)
v3 backup 데이터인 snapshot.db(내가 지정한 파일명)을 v2 backup 데이터 경로($ETCD_DATA_DIR/member/snap/)에 db라는 파일명으로 옮겨 주면 v3, v2 모두 안전하게 복구가 가능하다. 이 단계를 안할경우, [snapshot file doesn't exist etcd critical error](https://github.com/giantswarm/etcd-backup/issues/4) 발생

