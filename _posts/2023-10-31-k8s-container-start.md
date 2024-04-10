---
title: \[Kubernetes] 쿠버네티스로 컨테이너 시작하기
author: 
date: 2023-10-31 12:30:00 +0900
categories: [Kubernetes, 쿠버네티스 입문:90가지 예제로 배우는 컨테이너 관리 자동화 표준]
tags: [Kubernetes]
---

[쿠버네티스 입문:90가지 예제로 배우는 컨테이너 관리 자동화 표준] 책의 3장 쿠버네티스로 컨테이너 시작하기 단원을 공부하고 작성한 글입니다.

# 3.1 kubectl

쿠버네티스 클러스터를 kubectl로 관리하는 방법을 알아보자

## 3.1.2 기본 사용법

### kubectl의 기본 사용법

간단한 에코 서버를 동작시키는 kubectl 명령 예에서 kubectl의 기본 사용법을 알아보자

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/93213721-7ad8-4d06-9bc3-a2d377f4f7f6/Untitled.png)

- 일반적으로 **애플리케이션의 개별 기능은 각각 파드로 따로 구현하는 것**을 추천

- 그래야 그 기능에 버그가 생겨도, 다른 파드에 영향 없이 그 파드만 제거, 수정 가능

- 각 파드(기능) 간의 통신은 파드의 private IP 주소를 통해 이루어짐 (REST API 등)

- 외부 사용자가 파드에 접근하기 위해서, Service라는 쿠버네티스 리소스를 통해 접근해야 함

1. `echoserver`라는 이름의 파드 생성

```shell
$ kubectl run echoserver --generator=run-pod/v1 --image="k8s.gcr.io/echoserver:1.10" --port=8080
error: unknown flag: --generator
See 'kubectl run --help' for usage.

$ kubectl run echoserver --image=k8s.gcr.io/echoserver:1.10 --port=8080
pod/echoserver created
```

2. 파드에 접근할 때 필요한 서비스를 생성

```shell
$ kubectl expose po echoserver --type=NodePort
service/echoserver exposed
```

3. 파드가 정상적으로 생성되었는지 확인하기 위해 `kubectl get pods`

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/a0a4558d-82d6-4b7e-91e6-030b4c46d399/Untitled.png)

- NAME - 파드의 이름

- READY
    - 0/1 현재 파드가 생성되었으나 사용할 준비가 되지 않았음
    - 1/1 파드가 생성되었고 사용할 준비가 끝났음

- STATUS - 파드의 현재 상태
    - Running - 파드 실행됨
    - Terminating - (새로운 파드 생성 중) 컨테이너 생성 중
    - ContainerCreating - (새로운 파드 생성 중) 컨테이너 생성 중

- RESTARTS - 해당 파드가 몇 번 재시작했는지 표시

- AGE - 파드를 생성한 후 얼마나 시간이 지났는지 표시

4. 서비스가 정상적으로 생성되었는지 확인하기 위해 `kubectl get services`

```shell
$ kubectl expose po echoserver --type=NodePort
service/echoserver exposed

$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
echoserver   NodePort    10.103.207.36   <none>        8080:31205/TCP   3s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          26d
```

5. 서버에 접근할 수 있도록 로컬 컴퓨터로 port-forwarding

```powershell
$ kubectl port-forward svc/echoserver 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080
Handling connection for 8080
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/e2cd4fce-56d1-4066-9a81-b8180fc4ba51/Untitled.png)

### 서버를 실행한 후 테스트할 때

다른 shell을 하나 실행한 후 `curl http://localhost:8080` 

- Minikube 또는 Kubernetes 클러스터 외부에서 클러스터 내의 서비스에 엑세스 하기 위한 명령어

- 로컬 머신에서 “localhost” 주소와 8080 포트를 통해 클러스터 내의 서비스에 HTTP 요청을 보내는데, 이것은 클러스터 내의 서비스로 요청을 전달하고, 해당 서비스가 에코 서버 Pod에 연결되어 응답을 반환하는 방식

만약 동작하지 않는다면, `minikube`로 `echoserver`를 서비스한 후, `minikube` 내에서 접근

```shell
$ minikube ssh
docker@minikube:/$ curl http://192.168.49.2:31663
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/1528692e-4a73-4961-a7fd-c5e263ffa8e7/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/06e402b6-223b-4a3e-a119-342118ab6b44/Untitled.png)

### 서버의 실행 중 로그를 수집할 때

- 시간, HTTP 버전, 웹 브라우저와 운영체제 버전 확인 가능

```shell
$ kubectl logs -f echoserver
Generating self-signed cert
Generating a 2048 bit RSA private key
......+++
...........................................................................................................+++
writing new private key to '/certs/privateKey.key'
-----
Starting nginx
127.0.0.1 - - [30/Oct/2023:09:20:29 +0000] "GET / HTTP/1.1" 200 1072 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36 Edg/118.0.2088.76"
127.0.0.1 - - [30/Oct/2023:09:20:29 +0000] "GET /favicon.ico HTTP/1.1" 200 1010 "http://localhost:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36 Edg/118.0.2088.76"
10.244.0.1 - - [30/Oct/2023:09:21:04 +0000] "GET / HTTP/1.1" 200 1074 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36 Edg/118.0.2088.76"
10.244.0.1 - - [30/Oct/2023:09:21:04 +0000] "GET /favicon.ico HTTP/1.1" 200 1013 "http://127.0.0.1:59950/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36 Edg/118.0.2088.76"
10.244.0.1 - - [30/Oct/2023:09:40:37 +0000] "GET / HTTP/1.1" 200 424 "-" "curl/7.81.0"
```

### 실습한 파드와 서비스 삭제하기

1. shell에서 서버 로그 수집, 서버 실행을 중지
- 만약 앞서 `minikube service echoserver`를 통해 실행했다면, `ctrl + ^C`로 중지한 후 명령어를 통해 파드와 서비스를 삭제

```shell
$ kubectl delete pod echoserver
pod "echoserver" deleted
$ kubectl delete service echoserver
service "echoserver" deleted
```

2. 파드와 서비스 정보를 확인하는 명령으로 파드와 서비스가 정상적으로 삭제되었는지 확인

- Kubernetes 클러스터의 `default namespace`에서 리소스(예: Pod, 서비스 등)를 찾을 수 없다는 의미이므로, 정상적으로 삭제됨!

```shell
$ kubectl get pods
No resources found in default namespace.
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6h48m
```

## 3.1.3 POSIX/GNU 스타일의 명령 작성 규칙

### kubectl의 POSIX/GNU 명령 작성 규칙

```shell
$ kubectl -n default exec my-pod -c my-container -- ls /
```

[Argument Syntax (The GNU C Library)](https://www.gnu.org/software/libc/manual/html_node/Argument-Syntax.html)

## 3.1.4 플래그

- 모든 명령에서 사용할 수 있는 전역 플래그
- 개별 명령에서만 사용할 수 있는 개별 플래그

### 유용한 전역 플래그

```powershell
kubectl [command] --help // 개별 명령의 도움말 출력
kubectl [command] --v [log level] // 명령을 실행하는 과정의 로그 출력, 로그 레벨 설정, 디버깅에서 유용
```

## 3.1.5 kubeconfig 환경 변수

- `kubectl`은 기본적으로 `$HOME/.kube/config` 파일에서 클러스터, 인증, 컨텍스트 정보를 읽어 들임, 이러한 클러스터 구성 정보 ⇒ kubeconfig

- 클러스터에서 사용할 수 있는 자원들은 `kubectl api-resources` 명령으로 확인할 수 있음

```shell
$ kubectl api-resources
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
# 중략
csistoragecapacities                           storage.k8s.io/v1                      true         CSIStorageCapacity
storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment
```

- docker desktop으로 쿠버네티스를 사용한다면 자동으로 kubeconfig가 설정됨

- `—kubeconfig` 옵션으로 다른 설정 파일 지정 가능

```shell
$ kubectl config use-context docker-desktop
Switched to context "docker-desktop".
```

## 3.1.6 자동 완성

- `kubectl` bash, Z shell에서 자동 완성을 공식적으로 지원
- Z shell을 사용한다면 Oh My Zsh 사용 추천

```powershell
# bash
echo 'source <(kubectl completion bash)' >>~/.bashrc

# Z
echo 'source <(kubectl completion zsh)' >>~/.zshrc 
```

## 3.1.7 다양한 사용 예

`kubectl`을 단순히 명령 실행에 사용하는 것뿐 아니라, shell script의 일부분으로 사용하여 클러스터의 많은 동작을 자동화 가능


## 쿠버네티스에서의 JSONPath 사용

### JSONPath란? 

//링크 첨부

쿠버네티스에서는 클러스터 `config` 내용이나 `node`, `resource` 정보를 조회할 때 JSONPath 템플릿을 사용할 수 있음

장점

1) `kubectl get` 기본 명령으로는 접근할 수 없는 정보를 알아볼 수 있음

2) 필요에 따라 출력 결과물을 원하는 조건에 따라 정렬시킬 수 있음

⇒ 수십 개의 노드를 동시에 관리해야 하는 상용 환경에서도 꼭 필요한 정보만 걸러내어 확인할 수 있음

### 실습

JSONPath를 이용해서 원하는 정보를 확인하자

1. echoserver, [자기이름]server 2개의 파드를 만들자.
- 앞서 echoserver를 삭제했지만, 연습 삼아서 다시 만듦

```powershell
PS C:\WINDOWS\system32> kubectl run echoserver --image=k8s.gcr.io/echoserver:1.10 --port=8080
pod/echoserver created

PS C:\WINDOWS\system32> kubectl expose po echoserver --type=NodePort
service/echoserver exposed

PS C:\WINDOWS\system32> kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
echoserver   1/1     Running   0          16s

PS C:\WINDOWS\system32> kubectl run yunjinserver --image=k8s.gcr.io/echoserver:1.10 --port=8080
pod/yunjinserver created

PS C:\WINDOWS\system32> kubectl expose po yunjinserver --type=NodePort
service/yunjinserver exposed

// 잘 생성되었는지 pods, services 확인
PS C:\WINDOWS\system32> kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
echoserver     1/1     Running   0          63s
yunjinserver   1/1     Running   0          17s
...

PS C:\WINDOWS\system32> kubectl get services
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
echoserver     NodePort    10.106.213.33   <none>        8080:32760/TCP   76s
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP          26d
yunjinserver   NodePort    10.107.50.145   <none>        8080:32614/TCP   26s
...
```

# 3.2 디플로이먼트를 이용해 컨테이너 실행하기

쿠버네티스를 이용해서 컨테이너를 실행하는 방법

1. kubectl run 명령으로 직접 컨테이너를 실행
2. 어떻게 실행할 지 세부 내용을 담은 YAML 형식의 템플릿으로 컨테이너를 실행 → 버전 관리 시스템과 연동해서 자원 정의 변동 사항을 추적하기 쉬움

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/93213721-7ad8-4d06-9bc3-a2d377f4f7f6/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/afc5d231-ac75-4e4e-8580-c8794cc473ba/Untitled.png)

- Service란 pod의 논리적인 집합이며, 어떻게 접근할 지에 대한 정책을 내놓은 것
- 이 Service는 Label Selector를 통해 어떤 pod를 포함할 지 정의할 수 있음
- 각 pod의 외부 IP로는 외부에서 직접적인 접근이 불가능하지만, 서비스는 이를 가능하게 함

## 3.2.1 kubectl run으로 컨테이너 실행하기

쿠버네티스는 파드를 실행하는 여러 가지 컨트롤러를 제공함, kubectl run으로 파드를 실행시킬 때 기본 컨트롤러는 디플로이먼트

디플로이먼트를 이용해 nginx 컨테이너를 실행하자

1. 책에 나온 대로 [kubectl run 디플로이먼트이름 —image 컨테이너이미지이름 —port=80]
- deployment.apps가 나오지 않고 pod가 생성됨

```json
PS C:\WINDOWS\system32> kubectl run nginx-app --image nginx --port=80
pod/nginx-app created
```

1. 디플로이먼트를 이용해 nginx를 실행하고, 해당 디플로이먼트를 노출시키고, nginx를 실행하는 파드를 시작함

```powershell
PS C:\WINDOWS\system32> docker stop nginx-app
nginx-app

PS C:\WINDOWS\system32> docker rm nginx-app
nginx-app

PS C:\WINDOWS\system32> docker run -d --restart=always -e DOMAIN=cluster --name nginx-app -p 80:80 nginx
e070188344c507f20942944ab1178ac60fabae6c559ba26bf1e25a47e04d5987
```

```json
PS C:\WINDOWS\system32> docker ps
CONTAINER ID   IMAGE      COMMAND                   CREATED          STATUS          PORTS                  NAMES
677b69a6ec05   nginx     "/docker-entrypoint.…"   4 seconds ago    Up 4 seconds    0.0.0.0:80->80/tcp     nginx-app
```

```json
PS C:\WINDOWS\system32> kubectl create deployment --image=nginx nginx-app
deployment.apps/nginx-app created
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/6b0916d9-04c3-47db-a635-4e8efdc707ec/Untitled.png)

- 사용자가 쿠버네티스 클러스터에 컨테이너를 실행하라고 명령하면, 지정된 컨테이너 이미지를 가져와 쿠버네티스 클러스터 안에서 실행함

1. nginx 컨테이너가 제대로 실행됐는지 확인하자

```json
PS C:\WINDOWS\system32> kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
echoserver                   1/1     Running   0          38m
nginx-app-5c64488cdf-lhtg7   1/1     Running   0          68s <<
yunjinserver                 1/1     Running   0          37m

PS C:\WINDOWS\system32> kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app   1/1     1            1           82s <<
```

1. 디플로이먼트의 상태를 확인하는 항목 각각의 의미
- NAME - 클러스터에 배포한 디플로이먼트 이름
- READY - 사용자가 최종 배포한 파드 개수와 디플로이먼트를 이용해 현재 클러스터에 실제로 동작시킨 파드 개수를 X/X 형태로 표시
- UP-TO-DATE - 디플로이먼트 설정에 정의한 대로 동작 중인 신규 파드 개수
- AVAILABLE - 서비스 가능한 파드 개수
- AGE - 디플로이먼트를 생성한 후 지난 시간

1. 디플로이먼트를 이용해서 실행 중인 파드 개수를 늘려보자

```json
PS C:\WINDOWS\system32> kubectl scale deploy nginx-app --replicas=2
deployment.apps/nginx-app scaled

PS C:\WINDOWS\system32> kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
echoserver                   1/1     Running   0          51m
nginx-app                    1/1     Running   0          31m
nginx-app-5c64488cdf-lh67r   1/1     Running   0          26s <<
nginx-app-5c64488cdf-lhtg7   1/1     Running   0          14m <<
yunjinserver                 1/1     Running   0          50m

PS C:\WINDOWS\system32> kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app   2/2     2            2           14m 
```

- READY의 출력 결과가 2/2 → 사용자가 최종 배포한 파드 개수와 실제로 동작하는 파드 개수가 각각 2개라는 뜻

1. 원활한 실습을 위해 파드와 디플로이먼트를 삭제하자

```json
PS C:\WINDOWS\system32> kubectl delete deployment nginx-app
deployment.apps "nginx-app" deleted

PS C:\WINDOWS\system32> kubectl get deployments
No resources found in default namespace.

PS C:\WINDOWS\system32> kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
echoserver     1/1     Running   0          56m
yunjinserver   1/1     Running   0          55m
```

- 디플로이먼트를 삭제하면, 파드도 함께 삭제함, 다음과 같은 구조로 인해, 디플로이먼트를 삭제하면 파드도 함께 삭제됨
- 디플로이먼트는 파드를 업데이트 하기 위한 선언적 명세

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/cd1e050d-4a2e-4d0c-8966-a11cb4b84d80/Untitled.png)

## 3.2.2 템플릿으로 컨테이너 실행하기

디플로이먼트 설정이 담긴 템플릿(yaml 파일)로 컨테이너를 실행해보자

템플릿을 먼저 설명

### 실습

1. 바탕 화면에 deployment 폴더를 만들고 nginx-app.yaml 파일을 만든다.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/101c8601-7aad-484e-9765-42a43d0f5cd6/Untitled.png)
    

- 교재에 나온 코드와 확인 실습 (열면 안돼!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!)
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-app
      labels:
        app: nginx-app
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx-app
      template:
        metadata:
          labels:
            app: nginx-app
        spec:
          containers:
          - name: nginx-app
            image: nginx
            ports:
            - containerPort: 80
    ```
    
    ```yaml
    PS C:\WINDOWS\system32> cd C:\Users\82107\Desktop\deployment
    
    PS C:\Users\82107\Desktop\deployment> kubectl apply -f nginx-app.yaml
    deployment.apps/nginx-app created
    
    PS C:\Users\82107\Desktop\deployment> kubectl get pods
    NAME                         READY   STATUS    RESTARTS   AGE
    echoserver                   1/1     Running   0          68m
    nginx-app                    1/1     Running   0          49m
    nginx-app-5f8b897944-cjtw8   1/1     Running   0          15s <<
    yunjinserver                 1/1     Running   0          68m
    
    PS C:\Users\82107\Desktop\deployment> kubectl get deployments
    NAME        READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-app   1/1     1            1           24s
    ```
    

### 결과 화면

```powershell
PS C:\Users\82107\Desktop\deployment> kubectl apply -f nginx-app.yaml
deployment.apps/nginx-app-deployment-perfect created

PS C:\Users\82107\Desktop\deployment> kubectl get pods
NAME                                            READY   STATUS    RESTARTS   AGE
echoserver                                      1/1     Running   0          4h29m
nginx-app-deployment-perfect-5bbc997b77-blshn   1/1     Running   0          12s
nginx-app-deployment-perfect-5bbc997b77-xwmwj   1/1     Running   0          12s
yunjinserver                                    1/1     Running   0          4h28m

PS C:\Users\82107\Desktop\deployment> kubectl get deployments
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app-deployment-perfect   2/2     2            2           24s

PS C:\Users\82107\Desktop\deployment> kubectl expose deployment nginx-app-deployment-perfect --type=NodePort             
service/nginx-app-deployment-perfect exposed

PS C:\Users\82107\Desktop\deployment> kubectl get service
NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
echoserver                     NodePort    10.106.213.33   <none>        8080:32760/TCP   4h43m
kubernetes                     ClusterIP   10.96.0.1       <none>        443/TCP          26d
nginx-app-deployment-perfect   NodePort    10.97.250.254   <none>        80:31628/TCP     15s
yunjinserver                   NodePort    10.107.50.145   <none>        8080:32614/TCP   4h43m

PS C:\Users\82107\Desktop\deployment> kubectl get pods -o jsonpath='{.items[*].spec.containers[*].name}'
echoserver 
nginx-app-good 
nginx-app-good 
yunjinserver
```

[쿠버네티스에서 JSON 데이터 처리를 위한 JSONPath 사용법 (seongjin.me)](https://seongjin.me/how-to-use-jsonpath-in-kubernetes/)

### 원하는 결과를 위해, 직접 YAML을 작성하자!

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/cd1e050d-4a2e-4d0c-8966-a11cb4b84d80/Untitled.png)

1. 기본 template

```yaml
apiVersion: apps/v1
kind: ???
metadata:
  name: ???
  labels:
    app: ???
spec:
  replicas: ???
  selector:
    matchLabels:
      app: nginx-app-??
  template:
    metadata:
      labels:
        app: nginx-app-??
    spec:
      containers:
      - name: nginx-app-??
        image: nginx
        ports:
        - containerPort: 80
```

- 정답 (열면 안돼!!!!!!!!!!!!!!!!!!!!!!!)
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-app-deployment-perfect
      labels:
        app: nginx-app-deployment-perfect
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nginx-app-hello
      template:
        metadata:
          labels:
            app: nginx-app-hello
        spec:
          containers:
          - name: nginx-app-good
            image: nginx
            ports:
            - containerPort: 80
    ```
    

### 권장 방법?

쿠버네티스 자원들은 관련 설정을 정의한 템플릿(manifest)과 kubectl apply 명령을 이용해 선언적 형태로 관리할 것을 권장

자원을 생성할 때 사용한 템플릿 파일들은 앱 소스 코드와 함께 버전 관리 시스템으로 이력과 변경 사항을 추적하는 것이 좋음

## 3.3 클러스터 외부에서 클러스터 안 앱에 접근하기

실행 중인 nginx 컨테이너에 접속하는 방법을 살펴보자

쿠버네티스 내부에서 사용하는 네트워크는, 외부와 격리되었음! 쿠버네티스 내부에서 실행한 컨테이너를 외부에서 접근하려면, 쿠버네티스 서비스를 사용해야 함

서비스 타입 - ClusterIP, NodePort, LoadBalancer, ExternalName

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/30bd8554-f034-4567-8c82-d51af975062f/Untitled.png)

1. 서비스 하나에 모든 노드의 지정된 포트를 할당하는 NodePort를 설정하자
- 아까 .yaml 파일을 저장한 경로에서 하자
- nginx-app-deployment-perfect라는 서비스가 생성되었음
- 쿠버네티스 내부의 80번 포트가 31756라는 외부 포트와 연결되었음
- Endpoint를 보면 서비스에 컨테이너 2개가 연결되어 있음

```powershell
PS C:\Users\82107\Desktop\deployment> kubectl expose deployment nginx-app-deployment-perfect  --type=NodePort
service/nginx-app-deployment-perfect exposed

PS C:\Users\82107\Desktop\deployment> kubectl get service
NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
echoserver                     NodePort    10.109.240.33    <none>        8080:31163/TCP   56m
kubernetes                     ClusterIP   10.96.0.1        <none>        443/TCP          26d
nginx-app-deployment-perfect   NodePort    10.96.155.121    <none>        **80:31756/TCP**     17s
yunjinserver                   NodePort    10.102.144.249   <none>        8080:32174/TCP   55m

PS C:\Users\82107\Desktop\deployment> kubectl describe service nginx-app-deployment-perfect
Name:                     nginx-app-deployment-perfect
Namespace:                default
Labels:                   app=nginx-app-deployment-perfect
Annotations:              <none>
Selector:                 app=nginx-app-hello
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.155.121
IPs:                      10.96.155.121
LoadBalancer Ingress:     localhost
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31756/TCP
**Endpoints:                10.1.0.59:80,10.1.0.60:80**
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

1. localhost:31756로 접속하면 nginx의 기본 페이지를 확인할 수 있음 → 클러스터 안의 앱에 접근함!

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7ee22bbf-f04e-4fa4-9f6a-3c3277e80c15/e4c8e493-446d-4d5e-b3a7-17a39981634b/Untitled.png)

1. 원활한 실습을 위해 실행 중인 컨테이너를 삭제함

```powershell
PS C:\Users\82107\Desktop\deployment> kubectl delete deployment nginx-app
deployment.apps "nginx-app" deleted

PS C:\Users\82107\Desktop\deployment> kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
echoserver     1/1     Running   0          81m
nginx-app      1/1     Running   0          62m
yunjinserver   1/1     Running   0          81m

PS C:\Users\82107\Desktop\deployment> kubectl get deployments
No resources found in default namespace.
```
