---
title: Kubernetes 문제 풀이
author: dnya0
date: 2026-05-30 17:30:00 +0900
categories: [Infra, Kubernetes]
tag: [Study, Kubernetes, K8S, KodeKloud]
math: true
mermaid: true
image:
  path: ../assets/img/posts-image/k8s.png
  alt: Kubernetes Image
---

## 1. Deploy Pods in Kubernetes Cluster

### Problem

> The Nautilus DevOps team is diving into Kubernetes for application management. One team member has a task to create a pod according to the details below:
>
> Create a pod named `pod-httpd` using the `httpd` image with the `latest` tag. Ensure to specify the tag as `httpd:latest`.
>
> Set the `app` label to `httpd_app`, and name the container as `httpd-container`.
>
> Note: The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

<br>

### Explanation

위 문제는 Kubernetes 클러스터에 특정 조건의 Pod 하나를 생성하라는 뜻이다.

- Pod 이름: `pod-httpd`
- 사용할 이미지: `httpd:latest`
- label: `app=httpd_app`
- container 이름: `httpd-container`

또한 위에 *kubectl utility on the jump-host has been configured*라고 적혀있다. 이 뜻은 jump-host에 이미 접속해 있고 `kubectl` 설정(kubeconfig)이 끝나 있으니 바로 `kubectl` 명령을 쓰면 된다는 뜻이다.

가장 쉬운 방법은 `imperative command + yaml` 수정이다.

<br>

### Answer

우선 yaml을 생성시켜준다:

```shell
$ kubectl run pod-httpd \
  --image=httpd:latest \
  --dry-run=client -o yaml > pod.yaml
```

그 다음 pom.yaml을 수정해준다.

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: httpd_app
  name: pod-httpd
spec:
  containers:
    - image: httpd:latest
      name: httpd-container
      resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
status: {}
```

이후 적용해주면 된다.

```shell
$ kubectl apply -f pod.yaml
```

<br>

### Additional Notes

이번 문제에서는 단순히 Pod를 생성하는 것뿐 아니라 Kubernetes의 기본 구성 요소를 이해하는 것이 중요했다.

특히 Pod는 Kubernetes에서 가장 작은 배포 단위이며, 하나 이상의 컨테이너를 포함할 수 있다. 이번 실습에서는 `httpd` 이미지를 사용하는 단일 컨테이너 Pod를 생성했다.

또한 `label`은 Kubernetes에서 리소스를 식별하고 그룹화하는 데 사용된다. 이후 Service나 Deployment 등을 구성할 때 특정 label을 기준으로 Pod를 선택하게 된다.

예를 들어:

```yaml
selector:
  app: httpd_app
```

와 같이 사용된다.

이번 실습에서는 imperative command를 통해 기본 YAML을 생성한 뒤 수정하는 방식을 사용했다. Kubernetes를 처음 학습할 때는 직접 YAML을 작성하기보다 기본 구조를 생성한 뒤 수정하는 방식이 훨씬 이해하기 쉬운 것 같다.
