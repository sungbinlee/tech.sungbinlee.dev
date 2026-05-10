---
title: "쿠버네티스 딥다이브"
categories:
  - DevOps
tags:
  - Kubernetes
  - Container
  - Orchestration
  - Docker
  - Network
  - Service
  - Ingress
  - CNI
  - CSI
toc: true
toc_sticky: true
---

쿠버네티스를 단순히 “컨테이너를 실행해주는 도구”로만 이해하면 전체 구조가 잘 잡히지 않는다.

쿠버네티스의 핵심은 컨테이너 실행 그 자체가 아니라, 클러스터 전체의 상태를 계속 관찰하고 사용자가 선언한 원하는 상태로 맞춰가는 것이다.

사용자는 Deployment, StatefulSet, Service 같은 API 객체를 통해 원하는 상태를 선언한다. 쿠버네티스는 현재 상태와 원하는 상태를 계속 비교하면서 차이를 줄인다.

즉 쿠버네티스는 컨테이너 실행기가 아니라, 선언된 상태를 유지하기 위해 여러 제어 루프가 동작하는 분산 제어 시스템이다.



## 1. Kubernetes의 핵심 관점

### 1.1 Kubernetes는 컨테이너 실행기가 아니다

Kubernetes는 컨테이너화된 애플리케이션을 배포하고, 스케일링하고, 운영하기 위한 오케스트레이션 시스템이다.

하지만 더 정확히 보면 Kubernetes는 컨테이너를 직접 실행하는 도구라기보다, 클러스터의 상태를 관리하는 시스템에 가깝다.

흐름은 다음과 같다.

```text
사용자 선언
  ↓
kube-apiserver
  ↓
etcd에 상태 저장
  ↓
Controller가 상태 비교
  ↓
Scheduler가 Node 결정
  ↓
kubelet이 실제 Pod 실행
````

사용자는 “어떤 이미지를 실행할지”, “Pod를 몇 개 유지할지”, “어떤 Service로 노출할지”를 선언한다.

이 선언은 kube-apiserver를 통해 etcd에 저장된다. 이후 여러 컨트롤러가 etcd에 저장된 원하는 상태와 실제 클러스터 상태를 비교한다.

예를 들어 replicas를 3으로 선언했는데 실제 Pod가 2개만 있다면, Kubernetes는 부족한 Pod 1개를 다시 생성한다.

이 구조에서 중요한 점은 Kubernetes가 명령을 한 번 실행하고 끝나는 시스템이 아니라는 것이다.

Pod는 죽을 수 있고, Node는 장애가 날 수 있으며, 네트워크는 끊길 수 있다. 그래서 Kubernetes는 현재 상태와 원하는 상태를 계속 비교하면서 클러스터를 복구한다.



### 1.2 Desired State와 Control Loop

Kubernetes의 핵심 개념은 desired state와 actual state다.

desired state는 사용자가 선언한 원하는 상태다.

actual state는 현재 클러스터의 실제 상태다.

```text
desired state = 사용자가 원하는 상태
actual state  = 현재 클러스터의 실제 상태
```

Kubernetes는 이 둘을 계속 비교한다.

```text
현재 상태 조회
  ↓
원하는 상태와 비교
  ↓
차이 계산
  ↓
조치 실행
  ↓
다시 반복
```

이 반복 구조를 control loop라고 한다.

쿠버네티스의 자기복구 능력은 대부분 이 제어 루프에서 나온다.



## 2. Control Plane

Control Plane은 Kubernetes 클러스터의 두뇌 역할을 한다.

클러스터의 상태를 저장하고, 의사결정을 내리고, Worker Node가 무엇을 해야 하는지 알려준다.

Control Plane의 주요 구성요소는 다음과 같다.

```text
kube-apiserver
etcd
kube-scheduler
kube-controller-manager
cloud-controller-manager
```



### 2.1 kube-apiserver

kube-apiserver는 Kubernetes의 모든 요청이 들어오는 진입점이다.

사용자가 `kubectl apply`를 실행하든, 컨트롤러가 리소스를 조회하든, kubelet이 상태를 보고하든 대부분의 요청은 kube-apiserver를 거친다.

kube-apiserver의 역할은 다음과 같다.

```text
인증(Authentication)
인가(Authorization)
Admission Control
API 객체 검증
기본값 설정
etcd 저장
watch API 제공
```

중요한 점은 Kubernetes에서 etcd에 직접 접근하는 컴포넌트는 일반적으로 kube-apiserver뿐이라는 것이다.

모든 컴포넌트가 etcd를 직접 수정하면 인증과 인가를 우회할 수 있고, 객체 검증이 누락될 수 있으며, 버전 충돌 처리도 어려워진다.

그래서 kube-apiserver는 상태 변경의 유일한 게이트웨이 역할을 한다.

```text
Component
  ↓
kube-apiserver
  ↓
etcd
```

이 구조 덕분에 Kubernetes는 일관된 API 계약을 유지할 수 있다.



### 2.2 etcd

etcd는 Kubernetes 클러스터의 상태 저장소다.

Deployment, Pod, Service, ConfigMap, Secret, Node 상태 등 Kubernetes의 핵심 상태는 etcd에 저장된다.

etcd는 단순한 데이터베이스라기보다 Kubernetes 입장에서는 source of truth에 가깝다.

컨트롤러들은 etcd에 저장된 상태를 직접 읽는 것이 아니라 kube-apiserver를 통해 리소스를 watch한다. 그리고 리소스 변경 이벤트를 기반으로 자신의 제어 루프를 실행한다.

```text
Desired State
  ↓
kube-apiserver
  ↓
etcd
  ↓
watch
  ↓
Controller
```



### 2.3 Controller와 Control Loop

Controller는 특정 리소스의 상태를 계속 감시하고, 원하는 상태에 맞게 실제 상태를 조정하는 컴포넌트다.

예를 들어 Deployment에서 replicas를 3으로 선언했는데 실제 Pod가 2개만 있다면, ReplicaSet Controller는 부족한 Pod 1개를 추가로 생성한다.

```text
desired replicas = 3
actual pods = 2
  ↓
create 1 pod
```

Controller의 핵심은 control loop다.

```text
현재 상태 조회
  ↓
원하는 상태와 비교
  ↓
차이 계산
  ↓
조치 실행
  ↓
다시 반복
```

주요 컨트롤러는 다음과 같다.

```text
Deployment Controller
ReplicaSet Controller
StatefulSet Controller
DaemonSet Controller
Job Controller
Node Controller
EndpointSlice Controller
```



#### 2.3.1 Deployment와 ReplicaSet

Deployment는 애플리케이션 배포를 관리하는 상위 리소스다.

ReplicaSet은 실제로 Pod 개수를 유지하는 역할을 한다.

Deployment는 ReplicaSet을 직접 관리하면서 롤링 업데이트와 롤백을 제공한다.

```text
Deployment
  ↓
ReplicaSet
  ↓
Pod
```

ReplicaSet은 “Pod 몇 개를 유지할 것인가”에 집중한다.

Deployment는 그 위에서 “어떤 버전의 ReplicaSet을 사용할 것인가”, “어떻게 점진적으로 교체할 것인가”, “문제가 생기면 어떻게 롤백할 것인가”를 담당한다.

```text
ReplicaSet = Pod 개수 유지
Deployment = ReplicaSet 관리 + 배포 전략 제공
```



### 2.4 Scheduler

kube-scheduler는 새로 생성된 Pod를 어느 Node에 배치할지 결정한다.

Pod가 생성되면 처음에는 실행될 Node가 정해져 있지 않다. Scheduler는 이 Pod의 요구사항과 Node 상태를 확인한 뒤 가장 적합한 Node를 선택한다.

판단 기준에는 다음 요소들이 포함된다.

```text
CPU / Memory 요청량
Node 상태
taint / toleration
nodeSelector
nodeAffinity
podAffinity
podAntiAffinity
volume 제약
리소스 여유
```

Scheduler는 컨테이너를 직접 실행하지 않는다.

Scheduler는 “이 Pod는 이 Node에서 실행하라”는 결정을 API Server에 기록한다. 실제 실행은 해당 Node의 kubelet이 담당한다.

```text
Pod 생성
  ↓
Scheduler가 Node 결정
  ↓
Pod spec에 nodeName 기록
  ↓
해당 Node의 kubelet이 실행
```



## 3. Worker Node와 Pod 실행 구조

Worker Node는 실제 애플리케이션 Pod가 실행되는 곳이다.

Control Plane이 의사결정을 담당한다면 Worker Node는 그 결정을 실제 프로세스로 실행한다.



### 3.1 kubelet

각 Node에는 kubelet이 실행된다.

kubelet은 Node의 실행 관리자다.

kubelet은 API Server와 통신하면서 자신에게 할당된 Pod를 감지하고, 컨테이너 런타임을 통해 실제 컨테이너를 실행한다.

kubelet의 역할은 다음과 같다.

```text
Pod 실행
API Server와 통신
CRI 호출
CNI 호출
CSI 호출
Probe 수행
상태 보고
```

Pod 실행 흐름은 대략 다음과 같다.

```text
1. API Server에서 Pod spec 수신
2. 필요한 Volume 준비
3. CNI를 통해 네트워크 설정
4. CRI를 통해 Pod sandbox 생성
5. pause container 실행
6. init container 실행
7. app container 실행
8. probe 수행
9. 상태를 API Server에 보고
```

Control Plane이 “무엇을 해야 하는지”를 결정한다면, kubelet은 그 결정을 실제 프로세스와 컨테이너로 바꾸는 역할을 한다.



### 3.2 Pod

Pod는 Kubernetes의 최소 배포 단위다.

하나 이상의 컨테이너를 포함할 수 있으며, 같은 Pod 안의 컨테이너들은 같은 실행 문맥을 공유한다.

Pod 안의 컨테이너들은 다음을 공유한다.

```text
network namespace
volume
필요한 경우 IPC namespace
필요한 경우 PID namespace
```

가장 중요한 특징은 같은 Pod 안의 컨테이너들이 같은 네트워크 네임스페이스를 공유한다는 점이다.

그래서 같은 Pod 안의 컨테이너들은 서로 `localhost`로 통신할 수 있다.

```text
App Container
  ↔ localhost
Sidecar Container
```

이 구조 때문에 sidecar 패턴이 가능하다.



### 3.3 pause container

Pod가 생성될 때 가장 먼저 만들어지는 컨테이너가 pause container다.

pause container는 실제 애플리케이션 로직을 수행하지 않는다. 하지만 Pod의 네트워크 네임스페이스를 만들고 유지하는 기준점 역할을 한다.

```text
pause container
  ├── app container
  ├── sidecar container
  └── init container
```

pause container가 중요한 이유는 Pod의 네트워크 정체성을 컨테이너 생명주기와 분리하기 위해서다.

애플리케이션 컨테이너가 재시작되어도 pause container가 유지되면 Pod의 network namespace와 Pod IP는 유지될 수 있다.

즉 pause container는 Pod sandbox의 기준 프로세스다.



### 3.4 CRI와 OCI

Kubernetes는 컨테이너를 직접 실행하지 않는다.

kubelet은 CRI를 통해 컨테이너 런타임과 통신한다.

CRI는 Container Runtime Interface의 약자다.

```text
kubelet
  ↓ CRI
containerd / CRI-O
```

containerd 같은 컨테이너 런타임은 다시 OCI 런타임을 통해 실제 컨테이너 프로세스를 실행한다.

OCI는 Open Container Initiative의 약자로, 컨테이너 이미지 포맷과 런타임 실행 방식을 정의한 표준이다.

```text
kubelet
  ↓ CRI
containerd
  ↓ OCI
runc
  ↓
container process
```

정리하면 다음과 같다.

```text
CRI = Kubernetes와 컨테이너 런타임 사이의 인터페이스
OCI = 컨테이너 이미지와 실행 방식에 대한 표준
```



## 4. Linux 기반 기술

Kubernetes는 Linux의 여러 기능 위에서 동작한다.

컨테이너 자체도 완전히 새로운 기술이 아니라 Linux 커널 기능을 조합해 만든 실행 격리 방식이다.

주요 기반 기술은 다음과 같다.

```text
namespace
cgroup
VFS
iptables
IPVS
veth
bridge
VXLAN
seccomp
capabilities
AppArmor
SELinux
```



### 4.1 namespace와 cgroup

namespace는 프로세스가 볼 수 있는 세계를 분리한다.

예를 들어 컨테이너 안에서는 자신만의 프로세스 목록, 네트워크 인터페이스, 마운트 지점을 보는 것처럼 느낀다.

cgroup은 자원 사용량을 제한한다.

CPU, Memory, IO, PID 수 등을 제한하거나 측정할 수 있다.

```text
namespace = 보이는 것 분리
cgroup = 자원 사용량 제한
```

컨테이너 격리는 보통 namespace와 cgroup의 조합으로 이루어진다.

```text
namespace로 실행 환경 격리
cgroup으로 자원 사용 제한
```



### 4.2 VFS와 파일시스템

VFS는 Virtual File System의 약자다.

Linux 커널에서 서로 다른 파일시스템을 공통 인터페이스로 다룰 수 있게 해주는 추상화 계층이다.

애플리케이션은 `open`, `read`, `write` 같은 시스템 콜을 사용한다.

VFS는 이 요청을 실제 파일시스템 구현체로 연결한다.

```text
Application
  ↓
system call
  ↓
VFS
  ↓
ext4 / xfs / nfs / tmpfs
```

Kubernetes에서 사용되는 파일시스템 예시는 다음과 같다.

```text
Node local disk: ext4, xfs
emptyDir: Node local filesystem 위 임시 디렉토리
hostPath: Node 디렉토리 직접 사용
NFS: 네트워크 파일시스템
Block storage: attach 후 ext4/xfs로 format & mount
```



## 5. Kubernetes 네트워크

Kubernetes 네트워크는 Pod가 동적으로 생성되고 삭제되는 환경에서도 안정적으로 통신할 수 있도록 설계되어 있다.

핵심 구성요소는 다음과 같다.

```text
CNI
Pod Network
Service
EndpointSlice
kube-proxy
CoreDNS
Ingress
```



### 5.1 CNI

CNI는 Container Network Interface의 약자다.

Kubernetes에서 CNI는 Pod를 클러스터 네트워크에 연결하는 역할을 한다.

Pod가 생성되면 kubelet은 CNI 플러그인을 호출해 Pod의 네트워크를 구성한다.

CNI는 보통 다음 작업을 수행한다.

```text
Pod IP 할당
veth pair 생성
Pod network namespace에 eth0 생성
host network namespace와 연결
routing rule 설정
bridge 또는 tunnel 구성
NetworkPolicy 구현
```

일반적인 흐름은 다음과 같다.

```text
Pod network namespace
  eth0
   |
veth pair
   |
host network namespace
   |
bridge / route / overlay
```

즉 CNI는 단순히 IP를 하나 붙이는 도구가 아니라, Pod를 클러스터 네트워크에 편입시키는 메커니즘이다.



### 5.2 Kubernetes 네트워크 모델

Kubernetes 네트워크 모델의 핵심은 모든 Pod가 고유한 IP를 가지고, Pod 간 통신이 NAT 없이 가능해야 한다는 것이다.

기본 가정은 다음과 같다.

```text
모든 Pod는 고유한 IP를 가진다.
Pod 간 직접 통신이 가능해야 한다.
같은 Node든 다른 Node든 통신 방식이 같아야 한다.
같은 Pod 안의 컨테이너들은 localhost로 통신할 수 있다.
```

이 모델 덕분에 애플리케이션은 복잡한 네트워크 변환을 신경 쓰지 않고 다른 Pod IP로 통신할 수 있다.

다만 실무에서는 Pod IP를 직접 사용하는 방식은 권장되지 않는다. Pod는 재생성될 수 있고, 이 과정에서 IP가 바뀔 수 있기 때문이다.

그래서 일반적으로 Service를 통해 통신한다.



### 5.3 Overlay와 Underlay

Pod 네트워크를 구성하는 방식은 크게 Overlay와 Underlay로 나눌 수 있다.

Underlay는 물리 네트워크가 Pod 대역을 직접 라우팅하는 방식이다.

Overlay는 물리 네트워크 위에 논리 네트워크를 하나 더 얹는 방식이다.

Overlay에서는 Pod 패킷이 노드 간 이동할 때 한 번 감싸진다. 대표적으로 VXLAN이 사용될 수 있다.

```text
Pod A packet
  ↓
encapsulation
  ↓
Node A → Node B
  ↓
decapsulation
  ↓
Pod B
```

Overlay의 장점은 물리 네트워크와 독립적으로 Pod 네트워크를 구성하기 쉽다는 점이다.

반면 Underlay는 성능과 네트워크 통합 측면에서 유리할 수 있다.

```text
Overlay = 물리 네트워크 위에 논리 네트워크 구성
Underlay = 물리 네트워크가 Pod 대역을 직접 라우팅
```



## 6. Service 네트워크

Pod는 계속 생성되고 삭제된다.

Deployment 롤링 업데이트, 장애 복구, 스케일 아웃, 스케일 인 과정에서 Pod IP는 바뀔 수 있다.

클라이언트가 Pod IP를 직접 호출하면 안정적인 통신이 어렵다.

이 문제를 해결하기 위해 Kubernetes는 Service를 제공한다.



### 6.1 Service

Service는 Pod에 접근하기 위한 고정된 네트워크 진입점이다.

```text
Client
  ↓
Service
  ↓
Pod
```

Service는 고정된 ClusterIP와 DNS 이름을 제공한다. 클라이언트는 Pod IP를 직접 알 필요 없이 Service 이름으로 접근하면 된다.

Service는 Pod를 이름이나 IP로 직접 찾지 않는다. Label Selector를 기준으로 대상 Pod를 선택한다.

예를 들어 Pod에 다음 Label이 있다고 하자.

```yaml
labels:
  app: backend
```

Service는 다음 selector로 이 Pod들을 선택할 수 있다.

```yaml
selector:
  app: backend
```

이 방식 덕분에 Pod가 새로 생성되어도 같은 Label만 가지고 있으면 자동으로 Service 대상이 된다.



### 6.2 EndpointSlice

Service는 안정적인 진입점이지만, 실제 요청을 처리하는 것은 Service 뒤의 Pod다.

Service가 어떤 Pod로 트래픽을 보낼 수 있는지는 EndpointSlice에 저장된다.

EndpointSlice는 Service 뒤에 있는 실제 Pod IP와 Port 정보를 관리하는 리소스다.

```text
backend-service
  ├── 10.244.1.10:8080
  ├── 10.244.2.15:8080
  └── 10.244.3.20:8080
```

Pod가 새로 생성되면 EndpointSlice에 추가된다.

Pod가 삭제되면 EndpointSlice에서도 제거된다.

즉 EndpointSlice는 Service의 실제 backend 목록이다.



### 6.3 kube-proxy

kube-proxy는 각 Node에서 실행되는 네트워크 컴포넌트다.

kube-proxy는 Service와 EndpointSlice를 감시하고, Service IP로 들어온 요청이 실제 Pod로 전달되도록 네트워크 규칙을 설정한다.

보통 이 규칙은 Linux 커널의 iptables 또는 IPVS를 통해 구성된다.

```text
Client
  ↓
Service IP
  ↓
iptables / IPVS rule
  ↓
EndpointSlice에 등록된 Pod
  ↓
Backend Pod
```

중요한 점은 kube-proxy가 항상 모든 트래픽을 직접 중계하는 애플리케이션 프록시가 아니라는 것이다.

kube-proxy는 Service와 EndpointSlice 정보를 기반으로 커널 네트워크 규칙을 설정하는 역할에 가깝다.



### 6.4 CNI와 kube-proxy의 차이

CNI와 kube-proxy는 모두 Kubernetes 네트워크와 관련이 있지만 역할이 다르다.

CNI는 Pod 네트워크 자체를 구성한다.

kube-proxy는 Service 가상 IP를 실제 Pod IP로 연결한다.

```text
CNI = Pod가 통신할 수 있는 길을 만든다
kube-proxy = Service 주소를 실제 Pod로 번역한다
```

조금 더 구체적으로 보면 다음과 같다.

```text
CNI
- Pod IP 할당
- veth 생성
- route 설정
- overlay/underlay 구성
- NetworkPolicy 구현

kube-proxy
- Service 감시
- EndpointSlice 감시
- iptables/IPVS 규칙 설정
- Service IP를 backend Pod로 전달
```



### 6.5 CoreDNS

Kubernetes에서는 Service와 Pod가 동적으로 생성되고 삭제된다.

IP를 직접 사용하는 방식은 안정적이지 않기 때문에 Kubernetes는 DNS 기반 서비스 디스커버리를 제공한다.

CoreDNS는 Kubernetes 클러스터 내부 DNS 서버 역할을 한다.

Service가 생성되면 DNS 이름이 자동으로 만들어진다.

같은 Namespace 안에서는 다음처럼 접근할 수 있다.

```text
backend-service
```

다른 Namespace에서는 전체 DNS 이름을 사용할 수 있다.

```text
backend-service.default.svc.cluster.local
```

Pod 안의 `/etc/resolv.conf`는 클러스터 DNS를 바라보도록 설정된다.

애플리케이션이 Service 이름을 조회하면 CoreDNS가 Service의 ClusterIP를 응답한다.

```text
Pod
  ↓ DNS query
CoreDNS
  ↓
Service ClusterIP
```



### 6.6 내부 Service 통신 흐름

Pod가 Service 이름으로 다른 애플리케이션을 호출하는 흐름은 다음과 같다.

```text
Client Pod
  ↓
Service DNS 이름 조회
  ↓
CoreDNS가 ClusterIP 반환
  ↓
Client Pod가 ClusterIP로 요청
  ↓
kube-proxy가 설정한 iptables/IPVS 규칙 적용
  ↓
EndpointSlice에 등록된 Pod 중 하나 선택
  ↓
Backend Pod로 전달
```

이 구조 덕분에 클라이언트는 실제 Pod IP를 몰라도 안정적으로 backend를 호출할 수 있다.



## 7. HTTP, TCP/IP, Ingress

Kubernetes 네트워크를 제대로 이해하려면 Service뿐 아니라 HTTP, HTTPS, TCP, UDP, IP, L4, L7 개념도 함께 이해해야 한다.

Service와 Ingress의 차이도 결국 L4와 L7 차이에서 나온다.



### 7.1 IP

IP는 Internet Protocol의 약자다.

IP의 역할은 패킷을 목적지 주소까지 전달하는 것이다.

네트워크에서 각 장비나 Pod는 IP 주소를 가진다.

```text
192.168.0.10
10.244.1.15
```

Kubernetes에서도 Pod마다 고유한 IP가 부여된다.

다만 IP는 목적지까지 패킷을 보내는 역할을 할 뿐, 데이터가 반드시 도착했는지 보장하지 않는다.

전달 보장, 순서 보장, 재전송은 TCP 같은 전송 계층 프로토콜이 담당한다.



### 7.2 TCP

TCP는 Transmission Control Protocol의 약자다.

TCP는 신뢰성 있는 통신을 제공하는 전송 계층 프로토콜이다.

TCP는 데이터를 안정적으로 전달하기 위해 다음 기능을 제공한다.

```text
연결 기반 통신
데이터 전달 보장
데이터 순서 보장
손실 데이터 재전송
흐름 제어
혼잡 제어
```

HTTP, HTTPS, SSH, MySQL, PostgreSQL 같은 통신은 대부분 TCP 위에서 동작한다.

```text
HTTP
  ↓
TCP
  ↓
IP
```

Kubernetes Service에서도 TCP 트래픽을 처리할 수 있다.

```yaml
ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
```

TCP는 신뢰성이 중요한 통신에 적합하다.



### 7.3 UDP

UDP는 User Datagram Protocol의 약자다.

UDP는 TCP와 달리 연결을 맺지 않고 데이터를 빠르게 보내는 전송 계층 프로토콜이다.

UDP의 특징은 다음과 같다.

```text
비연결형 통신
데이터 전달 보장 없음
데이터 순서 보장 없음
재전송 없음
상대적으로 낮은 오버헤드
```

UDP는 신뢰성보다 속도나 실시간성이 중요한 곳에서 사용된다.

예시는 다음과 같다.

```text
DNS
실시간 스트리밍
온라인 게임
VoIP
일부 로그 수집 시스템
```

Kubernetes Service에서도 UDP 트래픽을 처리할 수 있다.

```yaml
ports:
  - port: 53
    targetPort: 53
    protocol: UDP
```



### 7.4 TCP와 UDP의 차이

TCP와 UDP의 가장 큰 차이는 신뢰성과 속도다.

| 구분    | TCP                  | UDP                 |
| ----- | -------------------- | ------------------- |
| 연결 방식 | 연결 기반                | 비연결형                |
| 전달 보장 | 보장함                  | 보장하지 않음             |
| 순서 보장 | 보장함                  | 보장하지 않음             |
| 재전송   | 있음                   | 없음                  |
| 속도    | 상대적으로 느림             | 상대적으로 빠름            |
| 사용 예시 | HTTP, HTTPS, SSH, DB | DNS, 스트리밍, 게임, VoIP |

TCP는 데이터가 정확히 도착해야 하는 통신에 적합하다.

UDP는 일부 손실이 있더라도 빠른 전달이 중요한 통신에 적합하다.



### 7.5 HTTP

HTTP는 HyperText Transfer Protocol의 약자다.

웹에서 클라이언트와 서버가 요청과 응답을 주고받기 위한 애플리케이션 계층 프로토콜이다.

예를 들어 브라우저나 프론트엔드 애플리케이션이 서버 API를 호출할 때 HTTP를 사용한다.

```http
GET /users HTTP/1.1
Host: api.example.com
```

서버는 요청을 처리한 뒤 HTML, JSON, 이미지 같은 데이터를 응답한다.

```json
{
  "id": 1,
  "name": "kim"
}
```

HTTP는 L7, 즉 애플리케이션 계층 프로토콜이다.

HTTP에서는 단순히 IP와 Port만 보는 것이 아니라 다음 정보를 다룬다.

```text
Method
Path
Header
Host
Body
Cookie
```

그래서 HTTP 기반 라우팅에서는 `/users`, `/orders`, `/login` 같은 경로에 따라 서로 다른 서비스로 요청을 보낼 수 있다.



### 7.6 TLS와 HTTPS

TLS는 Transport Layer Security의 약자다.

TLS는 네트워크 통신에서 데이터를 암호화하고, 서버가 신뢰할 수 있는 대상인지 확인하며, 데이터가 중간에서 변조되지 않았는지 검증하는 보안 프로토콜이다.

HTTP에 TLS가 적용된 형태가 HTTPS다.

```text
HTTP
  ↓
TLS
  ↓
TCP
  ↓
IP
```

TLS의 역할은 크게 세 가지다.

```text
암호화
인증
무결성 검증
```

암호화는 중간에서 패킷을 가로채더라도 실제 내용을 읽기 어렵게 만든다.

인증은 클라이언트가 접속한 서버가 진짜 신뢰할 수 있는 서버인지 확인하는 과정이다. 이때 서버는 TLS 인증서를 제공하고, 클라이언트는 인증서가 신뢰 가능한 CA에서 발급되었는지 확인한다.

무결성 검증은 데이터가 전송 중간에 바뀌지 않았는지 확인하는 것이다.

HTTPS는 HTTP에 TLS를 적용한 보안 프로토콜이다.

HTTP는 데이터를 평문으로 주고받기 때문에 중간에서 데이터가 노출될 수 있다.

HTTPS는 HTTP 메시지를 TLS로 암호화해서 전송한다.

```text
Client
  ↓
HTTP Request
  ↓
TLS 암호화
  ↓
TCP 전송
  ↓
Server
```

HTTPS는 다음을 제공한다.

```text
데이터 암호화
서버 인증
데이터 무결성 검증
```

운영 환경에서 로그인, 결제, 개인정보, API Token 같은 민감한 정보를 다룰 때 HTTPS는 필수에 가깝다.



#### 7.6.1 TLS Handshake

TLS Handshake는 클라이언트와 서버가 안전하게 통신하기 전에 암호화 방식과 세션 키를 협상하고, 인증서를 검증하는 과정이다.

간단한 흐름은 다음과 같다.

```text
1. ClientHello
   - 클라이언트가 지원하는 TLS 버전, 암호화 방식, 랜덤 값을 보낸다.

2. ServerHello
   - 서버가 사용할 TLS 버전과 암호화 방식을 선택한다.

3. Certificate
   - 서버가 자신의 TLS 인증서를 보낸다.

4. Certificate 검증
   - 클라이언트는 인증서가 신뢰 가능한 CA에서 발급되었는지 확인한다.

5. Key Exchange
   - 클라이언트와 서버가 세션 키 생성을 위한 정보를 교환한다.

6. Secure Communication
   - 이후 HTTP 요청과 응답은 세션 키로 암호화되어 전송된다.
```

핵심은 TLS Handshake에서 안전한 통신에 필요한 세션 키를 만들고, 이후 HTTP 데이터는 그 세션 키로 암호화되어 전송된다는 점이다.



### 7.7 L4와 L7

Service와 Ingress를 이해하려면 L4와 L7의 차이를 알아야 한다.

L4는 전송 계층이다.

대표적인 프로토콜은 TCP와 UDP다.

L4에서는 주로 다음 정보를 기준으로 트래픽을 처리한다.

```text
IP
Port
Protocol
```

예를 들면 다음과 같다.

```text
10.0.0.10:80 TCP
10.0.0.20:53 UDP
```

L7은 애플리케이션 계층이다.

대표적인 프로토콜은 HTTP와 HTTPS다.

L7에서는 HTTP 요청의 내용을 보고 판단할 수 있다.

```text
Host
Path
Header
Method
Cookie
```

그래서 L7에서는 다음과 같은 라우팅이 가능하다.

```text
api.example.com/users  → user-service
api.example.com/orders → order-service
```

정리하면 다음과 같다.

```text
L4 = IP, Port, Protocol 기준
L7 = Host, Path, Header, Method 같은 애플리케이션 정보 기준
```



### 7.8 Service와 Ingress

Service는 주로 L4 계층에서 Pod에 접근하기 위한 고정 진입점을 제공한다.

Ingress는 HTTP/HTTPS 기반의 L7 라우팅을 담당한다.

Service는 TCP/UDP 트래픽을 Pod로 전달한다.

Ingress는 Host나 Path를 기준으로 요청을 적절한 Service에 전달한다.

| 구분      | Service            | Ingress            |
| ------- | ------------------ | ------------------ |
| 계층      | L4                 | L7                 |
| 주요 프로토콜 | TCP, UDP           | HTTP, HTTPS        |
| 라우팅 기준  | IP, Port, Protocol | Host, Path, Header |
| 역할      | Pod 접근을 위한 고정 진입점  | 외부 HTTP/HTTPS 라우팅  |
| 대상      | Pod                | Service            |

Service는 다음처럼 동작한다.

```text
backend-service:8080 → backend Pod들
mysql-service:3306 → mysql Pod들
redis-service:6379 → redis Pod들
```

Ingress는 다음처럼 동작한다.

```text
api.example.com/users  → user-service
api.example.com/orders → order-service
admin.example.com      → admin-service
```

Ingress 자체는 규칙을 정의하는 Kubernetes 리소스다.

실제로 트래픽을 처리하려면 NGINX Ingress Controller, Traefik, HAProxy, AWS Load Balancer Controller 같은 Ingress Controller가 필요하다.



### 7.9 외부 요청 흐름

외부 사용자가 Ingress를 통해 애플리케이션에 접근하는 흐름은 다음과 같다.

```text
External Client
  ↓
Ingress Controller
  ↓
Ingress Rule
  ↓
Service
  ↓
EndpointSlice
  ↓
Pod
```

Ingress Controller는 HTTP/HTTPS 요청의 Host와 Path를 보고 적절한 Service로 요청을 전달한다.

Service는 다시 EndpointSlice에 등록된 실제 Pod 중 하나로 트래픽을 보낸다.



### 7.10 Headless Service

Headless Service는 `clusterIP: None`으로 생성한 Service다.

일반 Service는 ClusterIP라는 가상 IP를 제공한다.

반면 Headless Service는 가상 IP를 만들지 않고, backend Pod의 실제 IP를 DNS 응답으로 반환한다.

```yaml
spec:
  clusterIP: None
```

Headless Service는 StatefulSet과 자주 함께 사용된다.

StatefulSet은 stable pod name과 stable network identity를 제공한다.

예를 들어 다음과 같은 DNS 이름이 가능하다.

```text
web-0.nginx.default.svc.cluster.local
web-1.nginx.default.svc.cluster.local
web-2.nginx.default.svc.cluster.local
```

이 구조는 데이터베이스, Kafka, ZooKeeper처럼 각 인스턴스의 정체성이 중요한 워크로드에 유용하다.



## 8. Storage

Kubernetes에서 스토리지는 Pod 생명주기와 분리해서 이해해야 한다.

Pod는 언제든지 삭제되고 다시 생성될 수 있다. 따라서 중요한 데이터는 Pod 내부 파일시스템에만 저장하면 안 된다.

Kubernetes는 PV, PVC, CSI 같은 추상화를 통해 스토리지를 관리한다.



### 8.1 CSI

CSI는 Container Storage Interface의 약자다.

Kubernetes가 외부 스토리지를 표준 방식으로 사용할 수 있게 해주는 인터페이스다.

CSI를 통해 Kubernetes는 다음 작업을 수행할 수 있다.

```text
Volume 생성
Volume attach
Volume mount
Volume resize
Volume detach
```

CSI 구조는 크게 Controller Plugin과 Node Plugin으로 나뉜다.

```text
CSI Controller Plugin
- Volume 생성
- Volume 삭제
- Attach/Detach
- Resize

CSI Node Plugin
- Node에서 Volume mount
- Pod에 bind mount
```

CSI 동작 흐름은 다음과 같다.

```text
1. PVC 생성
2. StorageClass 확인
3. CSI Controller가 실제 Volume 생성
4. PV와 PVC 바인딩
5. Pod가 특정 Node로 스케줄링
6. kubelet이 CSI Node Plugin 호출
7. Node에 Volume mount
8. Container에 bind mount
```

즉 CSI는 제어 plane에서 Volume을 만들고, Node에서 실제로 mount하는 구조로 나뉜다.



### 8.2 PV와 PVC

Kubernetes에서 스토리지는 PV와 PVC로 추상화된다.

PV는 PersistentVolume의 약자로 실제 스토리지 자원을 의미한다.

PVC는 PersistentVolumeClaim의 약자로 사용자의 스토리지 요청이다.

```text
PVC = 저장공간 요청
PV = 실제 준비된 저장공간
```

Pod는 보통 PV를 직접 사용하지 않고 PVC를 통해 스토리지를 요청한다.

```text
Pod
  ↓
PVC
  ↓
PV
  ↓
Storage
```

이 구조 덕분에 애플리케이션은 실제 스토리지가 NFS인지, 클라우드 블록 스토리지인지, 로컬 디스크인지 직접 알 필요가 줄어든다.



### 8.3 emptyDir

emptyDir은 Pod 생명주기 동안 유지되는 임시 볼륨이다.

Pod가 생성될 때 만들어지고, Pod가 삭제되면 함께 사라진다.

emptyDir은 다음 상황에서 유용하다.

```text
임시 파일 저장
컨테이너 간 파일 공유
init container와 main container 간 데이터 전달
캐시
중간 결과물 저장
scratch 공간
```

예를 들어 init container가 설정 파일을 생성하고, main container가 그 파일을 읽는 구조를 만들 수 있다.

하지만 emptyDir은 영속 저장소가 아니다.

Pod가 삭제되면 데이터도 사라진다.



### 8.4 hostPath

hostPath는 Node의 파일시스템 경로를 Pod 안에 직접 마운트하는 볼륨이다.

```text
Node /var/log
  ↓
Pod /host/var/log
```

hostPath는 일반 애플리케이션 데이터 저장용보다는 Node 자원 접근용에 가깝다.

사용 예시는 다음과 같다.

```text
로그 수집기
node exporter
CNI plugin
CSI node plugin
보안 에이전트
Node socket 접근
```

대표 경로는 다음과 같다.

```text
/var/log
/proc
/sys
/etc/cni/net.d
/var/lib/kubelet
```

hostPath는 강력하지만 위험하다.

Pod가 Node 파일시스템에 접근할 수 있기 때문에 잘못 사용하면 보안 문제가 생길 수 있다.



### 8.5 CNI, CSI, hostPath의 관계

CNI와 CSI 구현체는 보통 DaemonSet으로 실행된다.

이들은 일반 애플리케이션처럼 Pod 안에서 실행되지만, 실제 작업 대상은 Pod 내부가 아니라 Node다.

CNI는 Node의 네트워크 설정을 변경해야 한다.

CSI Node Plugin은 Node에 디스크를 마운트하고 kubelet 경로와 상호작용해야 한다.

그래서 CNI/CSI 구현체는 hostPath를 통해 Node의 파일시스템이나 소켓에 접근하는 경우가 많다.

```text
CNI / CSI Pod
  ↓ hostPath
Node filesystem / socket
```

즉 CNI와 CSI가 hostPath에 의존한다기보다는, Node 수준 작업을 수행하기 위해 hostPath를 자주 사용한다고 보는 것이 정확하다.



## 9. 운영 패턴

Kubernetes를 운영할 때는 단순히 Pod를 실행하는 것뿐 아니라 생명주기, 종료 처리, 설정 변경, 모니터링, 워크로드 유형까지 함께 이해해야 한다.



### 9.1 Pod Lifecycle

Pod의 생명주기는 생성, 실행, 종료로 나눌 수 있다.

Pod 생성 흐름은 다음과 같다.

```text
1. 사용자가 Pod 또는 Deployment 생성
2. API Server가 요청 수신
3. etcd에 상태 저장
4. Scheduler가 Node 배치
5. kubelet이 Pod spec 감지
6. Volume 준비
7. CNI 네트워크 설정
8. pause container 실행
9. init container 실행
10. app container 실행
11. readiness probe 통과
12. Service endpoint로 등록
```

이 흐름에서 중요한 점은 Service backend 등록이 readiness와 연결된다는 것이다.

Pod가 Running 상태라고 해서 반드시 트래픽을 받을 준비가 된 것은 아니다.

readiness probe가 성공해야 Service endpoint로 등록되어 트래픽을 받을 수 있다.



### 9.2 Graceful Shutdown

Graceful shutdown은 애플리케이션이 갑자기 종료되지 않고 정상 종료 절차를 밟도록 하는 방식이다.

Kubernetes에서 Pod 삭제가 발생하면 다음 흐름이 진행된다.

```text
1. Pod 삭제 요청
2. deletionTimestamp 설정
3. kubelet이 종료 절차 시작
4. endpoint 제거 반영
5. preStop hook 실행
6. SIGTERM 전달
7. terminationGracePeriodSeconds 동안 대기
8. 종료되지 않으면 SIGKILL 전달
```

핵심은 바로 SIGKILL을 보내지 않는다는 점이다.

먼저 SIGTERM을 보내 애플리케이션이 정리 작업을 할 시간을 준다.

SIGTERM과 SIGKILL의 차이는 다음과 같다.

```text
SIGTERM = 정상 종료 요청
SIGKILL = 즉시 강제 종료
```

SIGTERM을 받은 애플리케이션은 다음 작업을 수행할 수 있다.

```text
처리 중 요청 마무리
DB connection close
queue ack flush
로그 flush
파일 write 마무리
트랜잭션 정리
```

반면 SIGKILL은 커널이 프로세스를 즉시 제거하기 때문에 애플리케이션이 정리 작업을 수행할 수 없다.

운영에서는 다음 사항이 중요하다.

```text
애플리케이션이 SIGTERM을 처리하는지 확인
terminationGracePeriodSeconds를 적절히 설정
preStop hook을 너무 길게 만들지 않기
readiness를 먼저 false로 만들어 신규 트래픽 차단
worker, HTTP server, stream consumer별 종료 전략 분리
```



### 9.3 StatefulSet

StatefulSet은 상태를 가진 워크로드를 위한 리소스다.

Deployment와 달리 StatefulSet은 각 Pod에 안정적인 이름과 네트워크 정체성, 스토리지를 제공한다.

StatefulSet의 특징은 다음과 같다.

```text
stable pod name
stable network identity
stable storage
순차적 생성과 종료
```

예를 들어 다음과 같은 Pod 이름이 유지된다.

```text
mysql-0
mysql-1
mysql-2
```

StatefulSet은 다음 워크로드에 적합하다.

```text
Database
Kafka
ZooKeeper
Redis Cluster
Elasticsearch
```



### 9.4 DaemonSet

DaemonSet은 모든 Node 또는 특정 조건을 만족하는 Node마다 Pod를 하나씩 실행하기 위한 리소스다.

DaemonSet은 Node 단위 에이전트에 적합하다.

예시는 다음과 같다.

```text
node-exporter
log collector
CNI plugin
CSI node plugin
security agent
monitoring agent
```

StatefulSet과 DaemonSet의 차이는 다음과 같다.

```text
StatefulSet = 상태 유지가 중요한 애플리케이션
DaemonSet = Node마다 실행되어야 하는 시스템 에이전트
```



### 9.5 Sidecar 패턴

Sidecar는 메인 애플리케이션을 보조하는 컨테이너를 같은 Pod 안에 함께 배치하는 패턴이다.

```text
Pod
  ├── app container
  └── sidecar container
```

Sidecar는 메인 앱과 같은 네트워크 네임스페이스와 볼륨을 공유한다.

그래서 다음이 가능하다.

```text
localhost 통신
같은 volume 접근
생명주기 결합
설정 파일 공유
로그 파일 공유
```

활용 예시는 다음과 같다.

```text
로그 수집기
프록시
config reload watcher
metrics exporter
secret agent
file sync
tracing agent
```

Sidecar는 메인 애플리케이션과 강하게 결합된 보조 기능을 분리할 때 유용하다.



#### 9.5.1 Sidecar Proxy

Sidecar Proxy는 네트워크 프록시 역할을 하는 Sidecar 컨테이너다.

대표적으로 Service Mesh 환경에서 Envoy Proxy가 사용된다.

구조는 다음과 같다.

```text
Inbound Traffic
  ↓
Sidecar Proxy
  ↓
App Container

App Container
  ↓
Sidecar Proxy
  ↓
Outbound Traffic
```

Sidecar Proxy를 사용하면 애플리케이션 코드를 수정하지 않고도 네트워크 기능을 추가할 수 있다.

예시는 다음과 같다.

```text
mTLS
인증/인가 위임
rate limiting
retry
timeout
circuit breaker
tracing header 삽입
metrics 수집
canary routing
A/B testing
```

장점은 공통 네트워크 기능을 애플리케이션 밖으로 분리할 수 있다는 점이다.

단점도 있다.

```text
컨테이너 수 증가
자원 사용량 증가
디버깅 복잡도 증가
시작/종료 순서 문제
프록시 장애가 앱 장애처럼 보일 수 있음
```



### 9.6 ConfigMap

ConfigMap은 설정 데이터를 Kubernetes 리소스로 분리하기 위한 객체다.

애플리케이션 설정을 이미지 안에 넣으면 설정 변경 때마다 이미지를 다시 빌드해야 한다.

ConfigMap을 사용하면 설정과 이미지를 분리할 수 있다.

ConfigMap을 사용하는 대표 방식은 다음과 같다.

```text
환경변수 주입
파일로 volume mount
command argument로 사용
```

주의할 점은 ConfigMap이 변경되었다고 애플리케이션 설정이 자동으로 반영되는 것은 아니라는 점이다.

환경변수로 주입한 ConfigMap은 일반적으로 Pod 재시작이 필요하다.

volume으로 mount한 ConfigMap은 kubelet이 파일 변경을 반영할 수 있지만, 애플리케이션이 그 파일을 다시 읽어야 실제 설정이 반영된다.

그래서 운영에서는 다음 방식이 사용된다.

```text
SIGHUP reload
sidecar watcher
reloader controller
rolling restart
```

즉 ConfigMap 갱신과 애플리케이션 설정 반영은 별개의 문제다.



### 9.7 Prometheus와 /proc

Linux의 `/proc`는 커널이 제공하는 가상 파일시스템이다.

프로세스, CPU, 메모리, 네트워크, 디스크 등의 상태 정보를 파일처럼 조회할 수 있다.

Prometheus 자체가 `/proc`를 직접 읽는 경우도 있지만, 일반적으로는 exporter가 `/proc`와 `/sys`를 읽어 메트릭으로 노출한다.

대표적으로 node-exporter가 있다.

```text
/proc, /sys
  ↓
node-exporter
  ↓ HTTP metrics
Prometheus
  ↓
time-series database
```

즉 `/proc`는 커널 상태의 원천 데이터이고, exporter는 이를 Prometheus가 읽을 수 있는 형식으로 변환하며, Prometheus는 주기적으로 scrape하여 저장한다.



### 9.8 kind

kind는 Kubernetes IN Docker의 약자다.

kind는 Docker 컨테이너를 Kubernetes Node처럼 사용해 로컬 Kubernetes 클러스터를 구성한다.

```text
Docker Container = Kubernetes Node
```

kind 클러스터의 Node 컨테이너 안에는 kubelet, containerd, kubeadm 기반 구성요소가 들어 있다.

kind의 장점은 다음과 같다.

```text
가볍다
빠르게 생성/삭제할 수 있다
로컬 개발에 좋다
CI 테스트에 적합하다
Kubernetes 기능 실험에 유용하다
```

실제 운영 클러스터와 완전히 같지는 않지만, Kubernetes 구조를 학습하거나 테스트 환경을 구성하기에는 매우 유용하다.



## 10. Kubernetes 보안

Kubernetes 보안은 컨테이너, Pod, Node, 클러스터 API 계층으로 나누어 봐야 한다.



### 10.1 컨테이너 보안

컨테이너는 VM이 아니다.

컨테이너는 Host Kernel을 공유한다.

따라서 컨테이너 보안의 핵심은 커널 공격면을 줄이고 권한을 최소화하는 것이다.

중요한 설정은 다음과 같다.

```text
non-root 실행
readOnlyRootFilesystem
capabilities drop
seccomp 적용
AppArmor / SELinux 적용
신뢰 가능한 이미지 사용
이미지 취약점 스캔
```



### 10.2 Pod 보안

Pod는 Kubernetes에서 실행 문맥을 구성하는 단위다.

잘못 설정된 Pod는 Node 전체로 이어지는 공격 경로가 될 수 있다.

주의해야 할 설정은 다음과 같다.

```text
privileged 사용 금지
hostNetwork 최소화
hostPID 최소화
hostIPC 최소화
hostPath 남용 금지
allowPrivilegeEscalation 금지
runAsNonRoot 적용
Pod Security Admission 적용
```

특히 privileged Pod와 hostPath 조합은 매우 강력하므로 신중하게 사용해야 한다.



### 10.3 Node 보안

Node는 실제 컨테이너가 실행되는 기반이다.

Node가 뚫리면 그 위에서 실행되는 Pod와 Secret, ServiceAccount Token도 위험해질 수 있다.

Node 보안에서 중요한 요소는 다음과 같다.

```text
kubelet 인증/인가 보호
runtime/OS 패치
SSH 접근 최소화
/var/lib/kubelet 보호
/etc/kubernetes 보호
audit logging
디스크 암호화
최소 OS 이미지 사용
```



## 11. 전체 정리

### 11.1 전체 구조 요약

Kubernetes는 여러 계층이 함께 동작하는 시스템이다.

큰 흐름은 다음과 같다.

```text
사용자는 API 객체로 원하는 상태를 선언한다.
kube-apiserver는 요청을 검증하고 etcd에 저장한다.
Controller는 원하는 상태와 현재 상태를 비교한다.
Scheduler는 Pod가 실행될 Node를 결정한다.
kubelet은 Node에서 실제 Pod를 실행한다.
CRI는 컨테이너 런타임과 연결한다.
CNI는 Pod 네트워크를 구성한다.
CSI는 스토리지를 연결한다.
Service는 Pod 앞의 안정적인 진입점을 제공한다.
EndpointSlice는 실제 Pod IP 목록을 관리한다.
kube-proxy는 Service 트래픽을 Pod로 전달하는 규칙을 설정한다.
CoreDNS는 Service 이름을 IP로 해석한다.
Ingress는 HTTP/HTTPS 외부 요청을 Service로 라우팅한다.
```



### 11.2 핵심 요약

Kubernetes는 Linux namespace, cgroup, 네트워크 스택, 파일시스템, 분산 저장소, 제어 루프를 결합한 분산 제어 시스템이다.

kube-apiserver는 모든 상태 변경의 관문이다.

etcd는 클러스터 상태의 source of truth다.

Controller는 desired state와 actual state를 계속 비교한다.

Scheduler는 Pod가 실행될 Node를 결정한다.

kubelet은 Node에서 실제 Pod를 실행한다.

Pod는 Kubernetes의 최소 배포 단위이며, 같은 Pod 안의 컨테이너들은 네트워크 네임스페이스와 볼륨을 공유한다.

pause container는 Pod sandbox와 network namespace의 기준점이다.

CNI는 Pod를 클러스터 네트워크에 연결한다.

CSI는 외부 스토리지를 Kubernetes에 연결한다.

Service는 변하는 Pod 앞에 안정적인 진입점을 제공한다.

EndpointSlice는 Service 뒤의 실제 Pod IP와 Port 목록을 관리한다.

kube-proxy는 Service 트래픽을 실제 Pod로 전달하기 위한 커널 네트워크 규칙을 설정한다.

CoreDNS는 Service 이름을 IP로 변환한다.

Ingress는 HTTP/HTTPS 기반 L7 라우팅을 담당한다.

Graceful shutdown은 요청 유실과 데이터 불일치를 줄이기 위한 정상 종료 절차다.

Sidecar는 메인 애플리케이션의 보조 기능을 같은 Pod 안에서 분리하는 패턴이다.

보안은 컨테이너, Pod, Node, API 계층을 나누어 최소 권한 원칙으로 접근해야 한다.



## 12. 최종 정리

Kubernetes를 제대로 이해하려면 단순히 Deployment나 Service 사용법만 외우는 것으로는 부족하다.

Kubernetes는 선언된 상태를 저장하고, 이를 계속 관찰하며, 실제 클러스터 상태를 원하는 상태로 맞추는 시스템이다.

그 과정에서 Control Plane은 의사결정을 담당하고, Worker Node는 실제 실행을 담당한다.

네트워크에서는 CNI, Service, EndpointSlice, kube-proxy, CoreDNS, Ingress가 함께 동작한다.

스토리지에서는 PV, PVC, CSI가 애플리케이션과 실제 저장소 사이를 추상화한다.

결국 Kubernetes는 컨테이너를 실행하는 도구가 아니라, Linux 기반 실행 환경과 분산 시스템의 제어 루프를 결합해 애플리케이션을 안정적으로 운영하기 위한 플랫폼이다.

