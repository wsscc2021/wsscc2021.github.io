---
title: Kubernetes - Cluster Autoscaler + OverProvisioning
author: D.G.Lee
date: 2021-10-27 15:00:00 +0900
categories: [DevOps]
tags: [DevOps, Kubernetes]
math: true
mermaid: true
---

## 컨테이너 생성 시간 vs VM 생성 시간

컨테이너는 VM보다 경량적인 isolation 기술로써, VM보다 이미지 크기가 작고 빠르게 프로비저닝되는 장점을 가지고 있습니다. 이는 분산 시스템 형태에서 수평적 확장을 수행하는 클라우드 환경에서 더욱 부각됩니다.

때문에 일반적으로 클라우드 환경에서는 클라우드 업체에서 제공해주는 VM 위에서 경량의 컨테이너를 오케스트레이션하는 방식으로 운영하게 되는 데, 대략적으로 아래와 같은 구성을 가집니다.

![ec2_container.png](/assets/img/posts/2021-10-27-Kubernetes - Cluster Autoscaler + OverProvisioning/ec2_container.png)

(확장 시 VM이 아닌 컨테이너만 생성하여 빠르게 확장합니다.)

![container_scaleout.png](/assets/img/posts/2021-10-27-Kubernetes - Cluster Autoscaler + OverProvisioning/container_scaleout.png)

## Cluster Autoscaler

위에서 살펴본 것처럼 노드(VM) 위에서 컨테이너를 실행함으로써 프로비저닝 시간을 단축하여 확장의 성능 측면에서 장점을 가질 수 있습니다.

하지만 VM의 모든 리소스를 사용하고 있는 상태에서 새로운 파드가 생성되면 스케줄링되지 못하고 pending 상태가 유지되는 현상이 발생합니다. 이러한 경우, 클라우드 업체에 맞는 ClusterAutoScaler를 사용하여 해결할 수 있습니다.

일반적으로 ClusterAutoScaler는 리소스가 부족하여 파드를 스케줄링하지 못할 때 노드(VM)을 추가하여 리소스를 확보하는 방식으로 동작합니다.

- AWS Cluster Auto Scaler: [https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/cluster-autoscaler.html](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/cluster-autoscaler.html)

![cluster_autoscaler1.png](/assets/img/posts/2021-10-27-Kubernetes - Cluster Autoscaler + OverProvisioning/cluster_autoscaler1.png)

![cluster_autoscaler2.png](/assets/img/posts/2021-10-27-Kubernetes - Cluster Autoscaler + OverProvisioning/cluster_autoscaler2.png)

## Over Provisioning

Cluster Auto Scaler는 파드가 생성되고 스케줄링할 때 리소스가 부족하다면 새로운 노드를 생성하는 방식으로 동작합니다.

![cluster_autoscaler.png](/assets/img/posts/2021-10-27-Kubernetes - Cluster Autoscaler + OverProvisioning/cluster_autoscaler.png)

즉, 리소스가 부족할 때 생성된 파드는 새로운 노드(VM)가 프로비저닝되는 시간 동안 스케줄링되지 못하고 Pending 상태로 남아있는 문제가 여전히 남아있습니다.

이는 아무런 의미가 없는 파드를 낮은 우선순위로 미리 생성함으로써 해결할 수 있습니다. (Over Provisioning)

![over_provisioning1.png](/assets/img/posts/2021-10-27-Kubernetes - Cluster Autoscaler + OverProvisioning/over_provisioning1.png)

새로운 파드가 생성되었을 때 우선순위가 낮은 빈 파드는 Eviction 되고, 그 자리에 새로운 파드가 스케줄 됩니다.

![over_provisioning2.png](/assets/img/posts/2021-10-27-Kubernetes - Cluster Autoscaler + OverProvisioning/over_provisioning2.png)

Eviction된 빈 파드를 위해서 Cluster Autoscaler가 동작하여 리소스를 확보합니다.

![over_provisioning3.png](/assets/img/posts/2021-10-27-Kubernetes - Cluster Autoscaler + OverProvisioning/over_provisioning3.png)

## 낮은 우선순위를 가지는 빈 파드 만들기

`PriorityClass`를 활용하여 우선순위가 낮은 빈 파드를 생성할 수 있습니다. 아래 예시를 참조합니다.

특히, resources 부분의 cpu/memmory 용량과 replica 수를 자신의 시스템에 맞게 적절한 값으로 정의해야 합니다.

```yaml
---
apiVersion: scheduling.k8s.io/v1beta1
kind: PriorityClass
metadata:
  name: overprovisioning
value: -1
globalDefault: false
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: overprovisioning
  namespace: kube-system
spec:
  replicas: 3
  selector:
    matchLabels:
      run: overprovisioning
  template:
    metadata:
      labels:
        run: overprovisioning
    spec:
      priorityClassName: overprovisioning
      containers:
      - name: reserve-resources
        image: k8s.gcr.io/pause
        resources:
          requests:
            cpu: 820m
            memory: 2000Mi
```

## 결론

over provisioning은 수평적 확장을 보다 빠르게 수행할 수 있다는 장점을 가지고 있지만, 리소스 공간을 낭비한다는 단점도 수반합니다. 때문에, 확장성에 큰 비중을 두지 않아도 되는 시스템에서는 over provisioning을 도입하지 않는 것이 더 효율적일 수 있습니다. 언제나 그렇듯 상황에 맞게 기술을 구현하여 사용할 필요가 있습니다.
