---
title: "도커 스웜 모드로 클러스터 구축하기 (개념 + 실습)"
categories:
  - DevOps
tags:
  - Docker
  - Docker Swarm
  -
toc: true
toc_sticky: true
---

## 도커 스웜 모드 클러스터 구축하기

도커와 쿠버네티스를 책으로 학습하면서 내용을 정리하고 있다. 기존에는 도커를 컨테이너를 생성하고 배포하는 용도로만 사용했지만, 여러 서버를 묶어 확장성을 확보하는 구조를 이해하기 위해 스웜 모드를 먼저 구성해본다.

도커 스웜은 여러 서버를 하나의 자원 풀로 묶어 클러스터를 구성하고, 컨테이너를 분산 실행해 스케일 아웃을 지원하는 기능이다. 쿠버네티스를 사용하기 전에 기본 개념을 이해하기 위해 스웜 모드를 먼저 실습한다.

## 스웜 구조

스웜은 매니저 노드와 워커 노드로 구성된다. 워커 노드는 실제 컨테이너가 실행되는 서버이고, 매니저 노드는 클러스터 전체를 관리하는 역할을 한다.

매니저 노드는 클러스터 상태를 저장하고 스케줄링을 수행하는 핵심 노드다. 여러 개의 매니저를 둘 수 있으며, 그 중 하나가 Leader가 된다.

스웜은 Raft 합의 알고리즘을 사용해 매니저 간 상태를 동기화한다. Leader가 상태 변경을 제안하고 과반수 매니저가 동의하면 반영된다. 따라서 매니저 노드는 보통 홀수 개로 구성한다.

## 실습 환경
실습은 VM 3대로 구성, 진행하였다.
```shell
docker-practice 192.168.0.14
swarm-worker1 192.168.0.16
swarm-worker2 192.168.0.17
````

## 스웜 클러스터 생성

```shell
root@docker-practice:/home/sungbin# docker swarm init --advertise-addr 192.168.0.14
Swarm initialized: current node (ba6yp5fu0aicdjit3av6p2vby) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-3b4bm4u6vn6e5pubd6ovau7yb6bt9qs54mj4bdhjqpzdyyxu0v-801uwui2lj03qa8jf2d80f6vp 192.168.0.14:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

각 워커 노드에서 join 명령어를 실행해 클러스터에 참여시킨다.

```shell
root@swarm-worker1:/home/sblee# docker swarm join --token SWMTKN-1-3b4bm4u6vn6e5pubd6ovau7yb6bt9qs54mj4bdhjqpzdyyxu0v-801uwui2lj03qa8jf2d80f6vp 192.168.0.14:2377
This node joined a swarm as a worker.
```

클러스터 상태 확인

```shell
root@docker-practice:/home/sungbin# docker node ls
ID                            HOSTNAME          STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ba6yp5fu0aicdjit3av6p2vby *   docker-practice   Ready     Active         Leader           29.3.0
m8694b48dgqk2pzdvbptrsrae     swarm-worker1     Ready     Active                          29.3.0
nn12t9nwbjaa2g60tfiv7757z     swarm-worker2     Ready     Active                          29.3.0
root@docker-practice:/home/sungbin#
```

현재 `*` 표시가 붙은 노드가 Leader 매니저 노드다. 워커 노드들도 정상적으로 Ready 상태로 추가된 것을 확인할 수 있다.

## 스웜 서비스

스웜은 컨테이너가 아닌 서비스 단위로 관리한다.

서비스는 정의된 레플리카 수를 유지하며, 부족한 경우 자동으로 컨테이너를 생성한다. 특정 노드에 문제가 발생하면 다른 노드에 재배치된다.

또한 롤링 업데이트를 지원해 서비스 중단 없이 배포가 가능하다.

## 서비스 생성

```shell
root@docker-practice:/home/sungbin# docker service create \
> ubuntu:14.04 \
> /bin/sh -c "while true; do echo hello world; sleep 1; done"

zdj1m3f1y8juutg7maalunlmo
overall progress: 1 out of 1 tasks
1/1: running
verify: Service zdj1m3f1y8juutg7maalunlmo converged
```

서비스 확인

```shell
root@docker-practice:/home/sungbin# docker service ls
ID             NAME             MODE         REPLICAS   IMAGE          PORTS
zdj1m3f1y8ju   cool_aryabhata   replicated   1/1        ubuntu:14.04
root@docker-practice:/home/sungbin# docker service ps cool_aryabhata
ID             NAME               IMAGE          NODE              DESIRED STATE   CURRENT STATE           ERROR     PORTS
o0iickbfqf40   cool_aryabhata.1   ubuntu:14.04   docker-practice   Running         Running 3 minutes ago
```

```shell
docker service rm cool_aryabhata
```

## Nginx 서비스 생성

```shell
root@docker-practice:/home/sungbin# docker service create --name myweb \
> --replicas 2 \
> -p 80:80 \
> nginx
uixdikr7y2vy1s9hsoatzn73g
overall progress: 2 out of 2 tasks
1/2: running
2/2: running
verify: Service uixdikr7y2vy1s9hsoatzn73g converged
root@docker-practice:/home/sungbin# docker service ps myweb
ID             NAME      IMAGE          NODE              DESIRED STATE   CURRENT STATE            ERROR     PORTS
kktu6al2vrj1   myweb.1   nginx:latest   swarm-worker1     Running         Running 32 seconds ago
qa6k2uyo5raz   myweb.2   nginx:latest   docker-practice   Running         Running 44 seconds ago
```

컨테이너가 여러 노드에 분산 배치된 것을 확인할 수 있다.

`-p 80:80` 옵션으로 스웜 전체에 포트를 개방했기 때문에 클러스터 내 어느 노드로 접근해도 동일한 서비스에 접근할 수 있다.

![swarm-worker2](/assets/images/devops/swarm-worker2.png)

컨테이너가 없는 노드로 접근해도 서비스가 정상적으로 동작한다.

## 스케일 아웃

```shell
root@docker-practice:/home/sungbin# docker service scale myweb=4
myweb scaled to 4
overall progress: 4 out of 4 tasks
1/4: running
2/4: running
3/4: running
4/4: running
verify: Service myweb converged
root@docker-practice:/home/sungbin# docker service ps myweb
ID             NAME      IMAGE          NODE              DESIRED STATE   CURRENT STATE            ERROR     PORTS
kktu6al2vrj1   myweb.1   nginx:latest   swarm-worker1     Running         Running 16 minutes ago
qa6k2uyo5raz   myweb.2   nginx:latest   docker-practice   Running         Running 16 minutes ago
meuehgn3uuzy   myweb.3   nginx:latest   swarm-worker2     Running         Running 12 seconds ago
omxv6abj9ljw   myweb.4   nginx:latest   swarm-worker2     Running         Running 12 seconds ago
root@docker-practice:/home/sungbin#
```

요청은 라운드 로빈 방식으로 분산된다.

## 글로벌 서비스

```shell
root@docker-practice:/home/sungbin# docker service create --name global_web \
> --mode global \
> nginx
zyix7ik84y88k5ei4yus8lutf
overall progress: 3 out of 3 tasks
ba6yp5fu0aic: running
m8694b48dgqk: running
nn12t9nwbjaa: running
verify: Service zyix7ik84y88k5ei4yus8lutf converged
```

모든 노드에 하나씩 컨테이너가 생성된다.

## 장애 복구

```shell
root@docker-practice:/home/sungbin# docker rm -f myweb.2.qa6k2uyo5raz2vkss021qy45d
myweb.2.qa6k2uyo5raz2vkss021qy45d
```

컨테이너가 삭제되면 자동으로 새로운 컨테이너가 생성된다.

노드 장애 발생

```shell
root@swarm-worker1:/home/sblee# service docker stop
```

```shell
root@docker-practice:/home/sungbin# docker node ls
m8694b48dgqk2pzdvbptrsrae     swarm-worker1     Down
```

다른 노드에 컨테이너가 재배치된다.

## 롤링 업데이트

```shell
root@docker-practice:/home/sungbin# docker service create --name myweb2 \
> --replicas 3 \
> nginx:1.24
```

```shell
root@docker-practice:/home/sungbin# docker service update --image nginx:1.26 myweb2
```

기존 컨테이너를 순차적으로 종료하고 새 컨테이너로 교체한다.

## Secret & Config

```shell
echo 1q2w3e4r | docker secret create my_mysql_password
```

Secret은 민감한 데이터를 안전하게 관리할 때 사용한다.

```shell
docker config create registry-config config.yml
```

Config는 설정 파일을 관리할 때 사용한다.

## 스웜 네트워크

```shell
root@docker-practice:/home/sungbin# docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
969a540e7639   bridge            bridge    local
cad05fc76214   docker_gwbridge   bridge    local
wekq9q6yj3i6   ingress           overlay   swarm
```

ingress 네트워크는 외부 요청을 서비스로 라우팅하고 로드밸런싱을 수행한다.

overlay 네트워크는 여러 노드 간 컨테이너 통신을 가능하게 한다.

## 정리

도커 스웜은 여러 서버를 하나의 클러스터로 묶고
컨테이너를 분산 실행하며
장애 복구와 스케일링을 자동으로 처리한다.
