---
title: "kubectl apply가 \"failed to download openapi\"로 죽었다 — YAML이 아니라 클러스터가 꺼져 있었다"
description: "에러 메시지는 YAML 검증 실패를 가리키는데, 실제로는 API 서버에 TCP 연결 자체가 안 되고 있었다."
date: 2026-09-01 08:40:00 +0900
categories: [트러블슈팅, 쿠버네티스]
tags: [kubectl, minikube, kubernetes, 네트워크]
error_signatures:
  - "error validating \"pod.yaml\": error validating data: failed to download openapi"
  - "dial tcp 192.168.49.2:8443: connect: no route to host"
  - "turn validation off with --validate=false"
verified: 2026-09-01
---

- `kubectl apply -f pod.yaml`이 `error validating "pod.yaml"`로 죽었다. 메시지만 보면 **YAML 문법 문제**처럼 읽힌다.
- 실제 원인은 YAML이 아니라, **minikube 클러스터가 꺼져 있어서 API 서버에 TCP 연결이 안 된 것**이었다.
- 에러 메시지가 제안하는 `--validate=false`는 **덮는 것일 뿐 고치는 게 아니다.**

---

## 1. 문제 상황

실습 중 Pod 매니페스트를 적용하려는데 이렇게 죽었다.

```text
error: error validating "pod.yaml": error validating data: failed to download openapi: Get "https://192.168.49.2:8443/openapi/v2?timeout=32s": dial tcp 192.168.49.2:8443: connect: no route to host; if you choose to ignore these errors, turn validation off with --validate=false
```

## 2. 환경

- 로컬 쿠버네티스: **minikube** (Docker driver, API 서버 `192.168.49.2:8443`)
- 클라이언트: `kubectl`
- 확인 시점: 2026-09-01

## 3. 원인을 좁혀나간 과정

### 3-1. 오진: "pod.yaml에 오타가 있다"

메시지가 `error validating "pod.yaml"`로 시작하고 파일명을 콕 집어 말하니, 당연히 YAML을 들여다봤다. 인덴트와 `apiVersion`을 한참 봤다.

**왜 틀렸나.** 메시지를 **끝까지** 읽지 않았다. 뒤쪽에 진짜 원인이 다 적혀 있었다.

### 3-2. 메시지를 뒤에서부터 읽기

이 한 줄은 사실 **세 겹**으로 중첩된 메시지다. 뒤에서부터 읽어야 원인이 나온다.

```text
error validating "pod.yaml"                 ← ③ 결과: 검증을 못 했다
  failed to download openapi                ← ② 이유: 스키마를 못 받아왔다
    dial tcp 192.168.49.2:8443:
      connect: no route to host             ← ① 원인: TCP 연결이 안 됐다
```

`dial tcp ... no route to host`는 **OS 네트워크 계층의 에러**다. 인증 실패(401)나 권한 거부(403)도 아니고, TLS 에러도 아니다. **패킷이 목적지까지 갈 경로가 없다**는 뜻이다. 즉 쿠버네티스 이야기가 아니라, 그 IP에 아무도 없다는 이야기다.

여기까지 읽으면 "클러스터가 안 떠 있다 / 네트워크가 끊겼다"로 바로 좁혀진다. YAML은 **애초에 검사조차 되지 않았다.**

## 4. 왜 YAML 검증이 네트워크를 타는가

`kubectl apply`는 YAML을 로컬에서 문법만 보고 보내지 않는다. 전송 전에 **API 서버에서 OpenAPI 스키마를 받아와** 리소스의 필드 이름·타입이 실제 클러스터에서 유효한지 확인한다. 클러스터에 설치된 CRD나 API 버전은 클러스터마다 다르니, 로컬 지식만으로는 검증할 수 없기 때문이다.

```text
kubectl apply -f pod.yaml
      │
      ├─(1) GET /openapi/v2  ──→ API 서버   ← 여기서 실패하면
      │                                       YAML은 읽히지도 않는다
      ├─(2) 스키마로 pod.yaml 검증
      └─(3) POST /api/v1/.../pods
```

그래서 **클러스터가 죽어 있으면 "YAML 검증 실패"라는 모양으로 에러가 나온다.** 메시지의 바깥쪽 표현(③)이 가장 눈에 띄는데 가장 쓸모가 없고, 가장 안쪽(①)이 진짜 원인이다.

`--validate=false`가 작동하는 이유도 같다. (1)단계를 건너뛰니 에러가 사라진다. 하지만 (3)단계에서 결국 같은 주소로 연결해야 하므로 **똑같이 실패한다.** 에러 메시지만 바뀐다.

> `--validate=false`는 "네트워크는 멀쩡한데 스키마에 없는 필드를 의도적으로 보내야 하는" 상황을 위한 옵션이다. 연결이 안 될 때 쓰는 우회로가 아니다.
{: .prompt-warning }

## 5. 직접 확인하는 법

클러스터가 살아 있는지부터 본다.

```bash
minikube status
```

`host`/`kubelet`/`apiserver`가 모두 `Running`이 아니면 그게 원인이다.

```bash
kubectl cluster-info
```

kubeconfig가 가리키는 주소와 minikube의 실제 IP가 **일치하는지** 비교한다. 재시작 과정에서 IP가 바뀌면 여기서 어긋난다.

```bash
minikube ip
kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}'; echo
```

주소까지 맞다면 그 포트에 실제로 닿는지 본다.

```bash
curl -k https://192.168.49.2:8443/version
```

## 6. 해결

클러스터를 다시 띄우면 된다.

```bash
minikube status
minikube start
minikube ip
kubectl cluster-info
```

`minikube start`가 IP를 바꾸면서 kubeconfig도 같이 갱신한다. `cluster-info`가 정상 출력되면 원래 명령을 다시 실행한다.

## 7. 재발 방지

- 노트북을 재부팅하면 minikube는 자동으로 올라오지 않는다. **실습 시작 전 `minikube status`를 먼저 치는 습관**을 들이면 이 에러를 아예 안 만난다.
- `kubectl` 명령이 실패했을 때 **메시지 맨 뒤를 먼저 읽는다.** 쿠버네티스 에러는 래핑이 깊어서, 가장 바깥 문장이 가장 오해를 유발한다.
- 연결 실패에 `--validate=false`로 대응하지 않는다.

## 8. 정리

| | |
| --- | --- |
| 증상 | `error validating "pod.yaml"` — YAML 문법 오류처럼 보임 |
| 메커니즘 | `apply`는 전송 전에 API 서버에서 OpenAPI 스키마를 받아온다. 그 요청이 실패하면 "검증 실패"로 보고된다 |
| 근본 원인 | minikube 클러스터가 꺼져 있어 `192.168.49.2:8443`에 TCP 연결 불가 (`no route to host`) |
| 해결 | `minikube start` 후 `kubectl cluster-info`로 확인 |
| 재발 방지 | 실습 전 `minikube status` / 메시지는 뒤에서부터 읽기 |

**설명** — YAML을 한 글자도 안 고쳤는데 해결된 이유는, YAML이 **읽히지도 않았기** 때문이다. 실패는 파일을 열기 전 단계에서 일어났다.

**예측** — 같은 상태에서 `kubectl get pods`를 치면? 검증 단계가 없으니 `error validating`은 안 나오지만, **같은 `dial tcp ... no route to host`로 실패한다.** 이게 "YAML 문제가 아니다"의 1초짜리 증명이다.

**파괴** — 클러스터를 띄운 뒤 `minikube stop`을 하고 같은 `apply`를 치면 이 에러가 그대로 재현된다. 반대로 `kubeconfig`의 `server` 주소만 엉뚱한 IP로 바꿔도 같은 메시지가 나온다 — **원인이 둘인데 메시지가 하나**라는 걸 보여준다.

교훈은 하나다. **에러 메시지의 첫 문장은 "무엇이 실패했는지"를 말하고, 마지막 문장은 "왜 실패했는지"를 말한다.** 고칠 수 있는 건 마지막 문장 쪽이다.
