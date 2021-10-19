---
title: Kubernetes - Pod 불균형 없애고 가용성 높이기
author: D.G.Lee
date: 2021-10-19 16:00:00 +0900
categories: [DevOps]
tags: [DevOps, Kubernetes]
math: true
mermaid: true
---

## Pod 불균형

파드를 생성할 때에 스케줄링에 아무런 제약을 주지 않으면 리소스 공간이 있는 노드에 무분별하게 스케줄링된다. 이러한 스케줄링 방식은 시스템 가용성의 허점을 줄 수 있다. 아래를 살펴보자.

서비스가 현재 트래픽을 견디기 위해서 4.0 CPU Core를 필요로 하고, 이를 위해서 2.0 CPU Core 를 가지는 파드를 4대 운영하고 있다고 가정해보자. 그리고, 파드를 생성할 때 스케줄링에 별다른 제약이 없다면, node-A의 리소스 공간이 충분하여 node-A에 3대의 파드가 스케줄링되고, node-B에 1대의 파드가 스케줄링된다. 물론 아무런 장애가 발생하지 않는다면, 이러한 시스템도 정상적으로 운영될 것이다.

![No-PodTopologySpreadContraints.png](/assets/img/posts/2021-10-19-Kubernetes - Pod 불균형 없애고 가용성 높이기/No-PodTopologySpreadContraints.png)

하지만 node-A에 장애가 생기게 된다면 node-B에는 1대의 파드만 존재하므로 일시적으로 기존 트래픽을 견딜 수 없는 상태가 된다. 이는 가용성이 충분히 확보된 시스템이라고 보기는 어렵다.

![No-PodTopologySpreadContraints-2.png](/assets/img/posts/2021-10-19-Kubernetes - Pod 불균형 없애고 가용성 높이기/No-PodTopologySpreadContraints-2.png)


## Pod Topology Spread Constraints

위와 같은 문제를 해결하기 위해서는 노드에 고르게 퍼져서 스케줄링되도록 해야한다. 만약 클라우드를 사용하고 있다면 가용영역 및 리전을 고려하여 스케줄링되도록 하여 가용성을 한층 더 높일 수 있을 것이다. 이러한 요구를 만족하기 위해서 파드에 `Pod Topology Spread Constraints` 을 정의할 수 있다. `Pod Topology Spread Constraints`을 간략하게 살펴보면, 파드를 스케줄링할 때 각 노드에 `labelSelector`에 매치되는 파드가 얼마나 할당되어 있는 지를 확인하여 파드 수의 차이(maxskew)가 벌어지지 않도록 적절한 노드를 선택하여 스케줄링한다. 결과적으로 파드가 각 노드에 고르게 퍼져서 스케줄링될 수 있다.

```yaml
topologySpreadConstraints:
- maxSkew: 1
  topologyKey: "kubernetes.io/hostname"
  whenUnsatisfiable: "ScheduleAnyway"
  labelSelector:
    matchLabels:
      app.kubernetes.io/name: account-app
```

![PodTopologySpreadContraints.png](/assets/img/posts/2021-10-19-Kubernetes - Pod 불균형 없애고 가용성 높이기/PodTopologySpreadContraints.png)


해당 글은 간략히 필요한 이유와 전체적인 흐름을 이해하는 정도로만 읽으면 좋을 것 같아서, 설정에 대한 내용은 정리하지 않겠습니다. 자세한 설정은 공식 홈페이지를 참조하여 정확하게 구현하는 것이 좋을 것입니다.

- https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/


## descheduler

`Pod Toplogy Spread Constraints` 는 기본적으로 스케줄링에 제약을 주는 기능입니다. 즉, 이미 스케줄링된 기존의 파드에게는 적용되지 않습니다. 이는 파드가 scale-in 되면서 파드 불균형이 발생할 수 있음을 의미합니다. scale-in 되는 상황에서도 파드 불균형이 발생하지 않도록 descheduler를 사용할 수 있습니다.

- https://github.com/kubernetes-sigs/descheduler
