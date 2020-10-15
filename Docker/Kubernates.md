# Kubernates

도커의 여러 컨테이너들을 오케스트레이션 해주는 프레임워크
적은 자원을 효율적으로 사용할 수 있게 해줌
Auto-Scaling, Auto-Healing, Deployment


## Container
VM은 각각의 OS를 띄워서 관리하고
Container는 하나의 OS를 여러곳에서 공유하면서 사용하는 원리

### 단점
Linux에서 Windows용 Container 사용이 불가능
하나의 Contatiner가 뚫리면 HostOS로 접근이 가능하여 다른 Container까지 피해가 갈 수 있음
VM은 그와 반대로 GuestOS와 HostOS 끼리 격리되어있어 하나의 VM이 뚫리더라도 HostOS에는 피해가 없다.

### 컨테이너간 격리
#### namespace
mnt, pid, net, ipc, uts, user
커널에 관련된 영역을 분리

#### cgroups
memory, CPU, I/O, Network
자원에 관련된 영역을 분리

## Pod
하나의 배포 단위로써
Contatiner가 1개 이상이 존재한다.
필요한 Pod만 확장이 가능하다