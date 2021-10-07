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

어플리케이션이 시작되어 요청을 처리할 수 있는 상태이지만, 대용량 데이터이나 구성파일을 로드하는 것을 기다리고 싶을 때 사용됩니다. 이 시기에는 파드를 재시작하고 싶지는 않고 **파드로 들어오는 요청을 받지 않고 싶습니다**.

`readiness probe`에 실패한 파드는 요청을 받지 않습니다.

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



readiness probe의 동작을 정확하게 이해하기 위해서는 서비스, 엔드포인트, 파드 리소스의 네트워크 연결이 어떻게 이뤄지는 지 이해해야 합니다.

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



## 결론

`liveness probe`와 `startup probe`는 파드의 self-healing을 위해서 반드시 사용되어야한다고 생각하며, `readiness probe`는 서비스와 영향을 주는 제 3의 서비스의 상태를 확인하기 위해서 사용될 수 있을 것 같습니다.

> 예를 들어, 만약 **"주문 서비스"를 파드로 실행하고 있을 때** "제품 서비스"에서 에러가 발생한다면 주문 기능을 정상적으로 수행할 수 없을 것 입니다. 하지만 주문 서비스에는 아무런 문제가 없으므로 파드를 재시작할 필요가 없습니다. 이럴 경우 readiness probe를 통해  주문 서비스는 실행되고 있는 상태로 요청만 받지 않도록 구성할 수 있습니다.