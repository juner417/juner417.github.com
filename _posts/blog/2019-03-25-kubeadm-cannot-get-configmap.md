---
layout: post
title: "kubeadm join issue - cannot get configmap in namespace"
date: 2019-03-25 19:00:00
categories: blog
author: juner
tags: [kubernetes, lessons-learned]
comments: true
---
# kubernetes lessons-learned - kubeadm join error
## system:bootstrap cannot get configmap

### Issue
* v1.5부터 업그레이드 해서 사용한 kubernetes cluster(kubeadm으로 설치)
* v1.11로 업그레이드 후 노드 추가
* kubeadm으로  node join시 아래 이슈 발생

```
$ kubeadm join --discovery-token-unsafe-skip-ca-verification --token ***************** --node-name kube-node013 myk8s.cluster.com:6443
...
    "I0129 14:05:18.090145 24010 kernel_validator.go:81] Validating kernel version",
    "I0129 14:05:18.090274 24010 kernel_validator.go:96] Validating kernel config",
    "\t[WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 17.12.1-ce. Max validated version: 17.03",
    "\t[WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'",
    "configmaps \"kubelet-config-1.11\" is forbidden: User \"system:bootstrap\" cannot get configmaps in the namespace \"kube-system\""
# --> kubelet-config-1.11(dynamic config를 위해 준비한 kubelet config)
```

### cause
* kubeadm이 1.11로 버전이 올라가면서 join시 bootstrap config를 configmap에 접근하여 가져오기 위해 createConfigMapRBACRules 함수가 추가됨[코드참고](https://github.com/kubernetes/kubernetes/blob/716b25396305b97034b019c13a937fcdfd364f9c/cmd/kubeadm/app/phases/kubelet/config.go#L88
)
* 우리는 1.11이전 버전의 위 함수가 없는 kubeadm으로 init해서 kube-system namespace에 kubelet-bootstrap role과 rolebinding이 없어서 권한 문제 발생함

### solution
* 아래의 내용으로 role, role-binding을 생성해 주니 문제 해결
```
# role, rolebinding만들어줌
cat <<EOF > kubelet-bootsrap-role.yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: kube-system
  name: kubelet-bootstrap
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["configmaps"]
  verbs: ["get", "watch", "list"]

cat <<EOF > kubelet-bootstrap-rolebinding.yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: kubeadm:kubelet-bootstrap
  namespace: kube-system
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:bootstrappers:kubeadm:default-node-token
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubelet-bootstrap


# role, rolebinding 생성
$ kubectl apply -f kubelet-bootstrap-role.yaml
$ kubectl apply -f kubelet-bootstrap-rolebinding.yaml

```

