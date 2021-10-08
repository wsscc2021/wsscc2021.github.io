---
title: Kubernetes - Healthcheck (liveness-probe, startup-probe, readiness-probe)
author: D.G.Lee
date: 2021-10-07 20:30:00 +0900
categories: [DevOps]
tags: [DevOps,Container,Kubernetes]
math: true
mermaid: true
---

## Overview

Kubernetes에서는 pod 의 상태를 확인하기 위해서 `liveness-probe`, `startup-probe`, `readiness-probe`가 사용됩니다. 각각을 살펴보면서 어떻게 활용할지에 대한 고민을 해봅니다. (HTTP, TCP, Exec 3가지 타입의 probe가 존재하지만, 가장 많이 사용되는 HTTP probe 타입으로 진행합니다.)



## liveness probe

liveness probe는 에러가 발생하여 요청을 처리할 수 없는 상태의 **pod를 재시작**하는 것에 사용됩니다. (Self-Healing)

liveness probe는 healthcheck에 실패한 pod를 kill하고, 다시 시작합니다.

```
livenessProbe:
  httpGet:
    path: /healthcheck/liveness
    port: 8080
  initialDelaySeconds: 0 # default 0, 어플리케이션이 시작될때까지 기다리는 시간입니다.
  periodSeconds: 10 # default 10, Healthcheck 요청을 보내는 주기(Interval)입니다.
  timeoutSeconds: 5 # default 1, Healthcheck 요청의 타임아웃 값입니다.
  failureThreshold: 3 # default 3, 3번 이상 실패하면 파드를 재시작합니다.
  successThreshold: 1 # default 1, 1번 이상 성공하면 Running 상태로 넘어갑니다.
```



## startup probe

파드는 어플리케이션이 시작될 때 까지 요청을 처리하지 못할 수 있습니다. 이 시기에 **`liveness probe`로 파드가 재시작되는 것을 방지**하기 위해서 사용됩니다. 이는 `initialDelaySeconds` 옵션과 흡사하지만, 동적인 것과 정적인 것의 차이를 줄 수 있습니다.

> `initialDelaySeconds`는 정적으로 시간을 지정하여 어플리케이션 시작을 기다리지만, `startup probe`는 동적인 시간 대역을 가져갈 수 있습니다.

어플리케이션이 시작되는 데 걸리는 시간이 항상 일정할 수도 있지만, 구성 파일을 로드할 때 발생하는 지연시간, 서버의 리소스 사용률이 높을 경우 발생하는 지연시간 등의 여러 상황들로 인해서 시작되는 시간이 변동될 수 있습니다. 그리고 이러한 경우 `startup probe`가 유용하게 활용됩니다.

`startup probe` 는 healthcheck에 성공할 때까지 `liveness probe`, `readiness probe`를 시작하지 않습니다.

```
livenessProbe:
  httpGet:
    path: /healthcheck/liveness
    port: 8080
  initialDelaySeconds: 0 # default 0, 어플리케이션이 시작될때까지 기다리는 시간입니다.
  periodSeconds: 10 # default 10, Healthcheck 요청을 보내는 주기(Interval)입니다.
  timeoutSeconds: 5 # default 1, Healthcheck 요청의 타임아웃 값입니다.
  failureThreshold: 3 # default 3, 3번 이상 실패하면 파드를 재시작합니다.
  successThreshold: 1 # default 1, 1번 이상 성공하면 Running 상태로 넘어갑니다.

startupProbe:
  httpGet:
    path: /healthcheck/liveness
    port: 8080
  initialDelaySeconds: 0 # default 0, 어플리케이션이 시작될때까지 기다리는 시간입니다.
  periodSeconds: 10 # default 10, Healthcheck 요청을 보내는 주기(Interval)입니다.
  timeoutSeconds: 5 # default 1, Healthcheck 요청의 타임아웃 값입니다.
  failureThreshold: 6 # default 3, 3번 이상 실패하면 readiness probe, liveness probe를 시작하지 않습니다.
  successThreshold: 1 # default 1, 1번 이상 성공하면 readiness probe와 liveness probe를 시작합니다.
```



## readiness probe

파드에 트래픽이 몰려 cpu/memory 사용률이 일정 수준 이상 높아지면 요청을 처리하는 것에 지연이 생기거나 500에러가 발생하여 Healthcheck에 실패할 수 있습니다. 이는 어플리케이션의 에러가 아니며 cpu/memory 사용률을 낮춰주면 자연스럽게 해결되는 문제입니다. 이럴 경우 readiness probe을 활용하여 트래픽을 받지 않음으로써 cpu/memory 사용률을 낮춰주는 역할을 수행할 수 있습니다.

`readiness probe`에 실패한 파드는 **요청을 받지 않습니다.**

```
readinessProbe:
  httpGet:
    path: /healthcheck/readiness
    port: 8080
  initialDelaySeconds: 0 # default 0, 어플리케이션이 시작될때까지 기다리는 시간입니다.
  periodSeconds: 10 # default 10, Healthcheck 요청을 보내는 주기(Interval)입니다.
  timeoutSeconds: 5 # default 1, Healthcheck 요청의 타임아웃 값입니다.
  failureThreshold: 3 # default 3, 3번 이상 실패하면 파드에 요청을 받지 않습니다.
  successThreshold: 1 # default 1, 1번 이상 성공하면 파드에 요청을 받습니다.
```



`readiness probe`의 동작을 정확하게 이해하기 위해서는 서비스, 엔드포인트, 파드 리소스의 네트워크 연결이 어떻게 이뤄지는 지 이해해야 합니다.

![readinessprobe.png](/assets/img/posts/2021-10-07-Kubernetes - Healthcheck (liveness-probe, startup-probe, readiness-probe)/readinessprobe.png)

위 그림에서 볼 수 있듯이, 파드는 서비스 리소스에 연결되기 전에 endpoints라는 리소스에 연결되며 endpoints 리소스 내부적으로는 `Addresses` 에 등록된 IP만 서비스 리소스에 연결됩니다. 즉, `NotReadyAddresses`로 등록된 IP는 서비스 리소스에 연결되지 않으며 요청을 받을 수 없는 상태가 됩니다.

그리고 `readiness probe `에 실패한 파드는 endpoints 리소스의 `NotReadyAddresses`로 등록됩니다.

```bash
kubectl describe pod <pod-name>

# Name:         kubia-readiness-pod
# ...
# IP:           10.0.45.55
```

```bash
kubectl describe endpoints <service-name>

# Name:         kubia-readiness-clusterip
# ...
# Subsets:
#  Addresses:          <none>
#  NotReadyAddresses:  10.0.45.55
```


만약 liveness probe로 파드가 재시작되기 전에 트래픽을 완화하기 위한 용도로 readiness probe를 사용한다면 아래를 주의해야 하여 설정하는 것이 좋습니다.

- readiness probe는 liveness probe와 다른 엔드포인트로 healthcheck를 진행합니다.
- readiness probe는 liveness probe보다 먼저 동작해야합니다.  


## 결론

`liveness probe`와 `startup probe`는 파드의 self-healing을 위해서 반드시 사용되어야 하지만 정확하게 이해하지 못한다면 pod가 무분별하게 재시작되어 시스템에 악영향을 줄 수 있기 때문에 조심해야 합니다.
`readiness probe`는 어플리케이션 시작 후 liveness probe의 무분별한 파드 재시작을 억제하기 위해서 사용될 수 있을 것입니다. 이를 위해서 liveness probe와 다른 엔드포인트로 healthcheck를 진행하며 liveness probe보다 먼저 동작하도록 설정해야 합니다.