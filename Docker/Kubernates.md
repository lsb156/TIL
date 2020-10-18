# Kubernetes

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

## Kubernetes Cluster
Kubernetes는 하나의 `Master`와 여러개의 `Node`들로 구성되어있다.
물리적인 `Master`와 `Node`들로 `Kubernetes Cluster`를 만들어 하나의 논리적인 집합을 만들어낸다
Master는 Kubernetes의 전반적인 기능을 컨트럴 하는 역할이고
Node들이 Kubernetes에 자원을 제공하는 역할로 자원을 늘리고 싶다면 Node들을 늘리면 된다.

### Namespace
Kubernetes Objecte들을 분리되게 만들어 프로젝트별 각각의 독립적인 공간을 제공한다.
#### Pod
하나의 배포 단위로써
Pod에는 Contatiner가 1개 이상이 존재한다.
필요한 Pod만 확장이 가능하다

#### Service
외부에서 접근이 가능하도록 파드들을 묶어서 관리해주는 역할
IP를 할당해주며 다른 Namespace에 있는 자원에 접근이 불가능하다.


#### Volume
Pod에 문제가 생겨서 다시 실행하거나 내려가게 된다면 내부에 있는 데이터들이 모두 사라지게 된다.
그래서 Volume을 별도로 지정하게 될 경우 그 Volume에 데이터들이 저장되어 재시작에서 데이터가 남아있게된다.

#### Resource Quota, LimitRange
Namespace에 지정되어 자원의 양을 한정할 수 있다.
Pod의 갯수, CPU, Memory 제한이 가능

#### ConfigMap, Secret
Pod 생성시 Container에 환경변수 값 설정 및 파일 마운팅이 가능

#### Controller
Pod들을 관리해주는 역할
##### Replication Controller
Pod가 죽으면 다시 살려줌
파드의 갯수를 Scale In/Out을 하여 갯수 조정이 가능하다.
##### ReplicaSet

##### Deployment
Pod들을 새 버전으로 업그레이드 하여준다.
업그레이드중 문제가 발생할 롤백을 쉽게 할 수 있도록 도와준다

##### DemonSet
한 노드의 파드가 하나씩만 유지되도록 도와줌

##### CronJob
Batch성으로 특정 작업만 하고 종료하는 역할을 진행



## Pod
Pod 내부의 Contatiner들이 여러개의 Port를 가질 수 있는데 하나의 Pod 내에 있는 Container에서 Port가 중복되어서는 안된다
같은 Pod 내부에서 localhost로 다른 Contatiner에 접근이 가능
Pod가 생성될때 고유의 IP가 생성이 되는데 Kubernetes Cluster에서만 접근이 가능하고 외부에서는 그 아이피로 접근이 불가
Pod에 문제가 생겨 재생성이 이루어질시 주어진 IP는 소멸되고 다른 IP가 생성됨
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  containers:
    - name: container-1
      image: imageName
      ports:
        - containerPort: 8080
    - name: container-2
      image: imageName
      ports:
        - containerPort: 8081
```

### Lebel
Pod에는 여러개의 Label을 설정할 수 있다.
설정파일에서 `spec.selector`에 `{key}: {value}` 형태로 설정이 가능
Lebel은 Pod를 필터링하여 그룹화 할때 사용된다.
``` yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    type: web
  ports:
    - port: 8080
```


### Node Schedule
- Pod를 실행시킬때는 Node위에 실행이 되는데 자동/수동으로 설정이 가능
- 설정파일에 `nodeSelector.hostname` 에 노드를 직접 설정하여 수동으로 지정이 가능
- 자동으로 설정될 경우 남은 Node의 메모리를 보고 판단
- 설정파일에서 Limit 설정이 가능
  - Memory : limit 초과시 Pod가 종료된다.
  - Cpu : limit가 넘어도 Pod가 종료되진 않지만 request 수치까지 내려간다.
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  nodeSelector:
    hostname: node1
  containers:
    - name: container 
      image: imageName
      resources:
        requests:
          memory: 2Gi
        limits:
          memory: 3Gi
```


```
gcloud init
gcloud components update
gcloud components install kubectl

kubectl get nodes
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```