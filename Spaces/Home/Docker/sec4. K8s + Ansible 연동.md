# 배포 방식 비교 - 기본 vs VM vs Container
![[Pasted image 20240308154830.png]]
# 기본 명령어
### 노드 확인  
`kubectl get nodes`
이 명령은 Kubernetes 클러스터 내의 모든 노드들을 나열하고 각 노드의 상태와 정보를 확인한다.
### pods 확인
`kubectl get pods`
현재 클러스터에서 실행 중인 모든 Pod들의 목록을 표시한다.
### deployments 확인
`kubectl get deployments`
현재 클러스터에서 실행 중인 모든 Deployments의 목록을 표시한다.
### 서비스 확인 
`kubectl get services`
현재 클러스터에서 정의된 모든 서비스의 목록을 표시한다.
### Nginx 서버 실행 
`kubectl run sample-nginx --image=nginx --port=80`
Nginx Docker 이미지를 사용하여 `sample-nginx`라는 이름의 Pod를 생성하고 80번 포트로 노출시킨다.
### 컨테이너 정보 확인 
`kubectl describe pod/sample-nginx`
### pods 삭제
`kubectl delete pod/sample-nginx-XXXXX-XXXXX`
### Scale 변경 (2개로 변경)
`kubectl scale deployment sample-nginx --replicas=2`
Deployment의 replica 수를 변경하여 scale을 조절한다. 위 예제에서는 2개의 replica로 변경하고 있다.
### Script 실행
`kubectl apply -f sample1.yml`
YAML 파일에 정의된 Kubernetes 리소스를 클러스터에 적용한다.
### 파드 확인
`kubectl get pos -o wide`
Pods의 목록을 표시하면서 각 Pod의 상세한 정보와 IP 주소 등을 함께 표시한다.
### 파드에 터널링으로 접속
`kubectl exec -it nginx-deployment-XXXX-XXXX -- /bin/bash`
특정 Pod에 접속하여 컨테이너 내부로 들어갈 수 있다.
### 파드 노출(공개)
`kubectl expose deployment nginx-deployment --port=80 --type=NodePort`
Deployment를 서비스로 노출시키고, 외부에서 해당 서비스에 접근할 수 있도록 한다.
NodePort 타입의 서비스로 80번 포트를 노출한다는 뜻이다.

```ad-note
title: 만든 deployments를 삭제할 때
`kubectl create deployment sample-nginx --image=nginx`
deployment를 만들면 sample-nginx의 pod가 1개 생성된다.
`kubectl delete pod/sample-nginx-56dfc8544d-5ffsj`
만들어진 pod를 삭제해도 자동으로 replica의 수만큼 다른 이름의 pod가 생성돼 1개를 유지한다.

**deployment 자체를 삭제해야 pods도 한꺼번에 삭제된다.**
```

# 예제1 - yml파일로 deployments, pod 생성

**sample1.yml**
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        
```

`kubectl apply -f sample1.yml`