# 배포 방식 비교 - 기본 vs VM vs Container
![[sec4_1.png]]
# 기본 명령어
### 모든 리소스 조회
`kubectl get all`
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

`kubectl apply -f sample1.yml`을 실행하면 replicas가 2개인 nginx-deployment가 생성된다.
![[sec4_2.png]]
# 예제2 - Ansible로 kubenetes script 실행하기

1. **도커 이미지를 사용해 deployment생성하는 yml파일 작성(Host PC)**

**cicd-devops-deployment.yml**
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cicd-deployment
spec:
  selector:
    matchLabels:
      app: cicd-devops-project
  replicas: 2

  template:
    metadata:
      labels:
        app: cicd-devops-project
    spec:
      containers:
      - name: cicd-devops-project
        image: hyuxp/cicd-project-ansible
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        
```

2. **service 생성하는 yml파일 작성(Host PC)**

**cicd-devops-service.yml**
```yml
apiVersion: v1
kind: Service
metadata:
  name: cicd-service
  labels:
    app: cicd-devops-project
spec:
  selector:
    app: cicd-devops-project
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32000
      
```

3. **위 yml파일을 실행하는 playbook yml 스크립트 작성(ansible-server)**

**k8s-cicd-deployment-playbook.yml**
기존에 cicd-deployment를 삭제하고 2번의 yml을 실행하라는 스크립트
```yml
- name: Create pods using deployment
  hosts: kubernetes
  # become: true
  # user: ubuntu

  tasks:
  - name: delete the previous deployment
    win_command: kubectl delete deployment.apps/cicd-deployment

  - name: create a deployment
    win_command: kubectl apply -f C:\Users/박경현_인터넷망\Downloads\cicd-devops-deployment.yml
```

**k8s-cicd-service-playbook.yml**
2번의 service생성하는 yml을 실행하라는 스크립트
```
- name: create service for deployment
  hosts: kubernetes
  # become: true
  # user: ubuntu

  tasks:
  - name: create a service
    win_command: kubectl apply -f C:\Users/박경현_인터넷망\Downloads\cicd-devops-service.yml
```

4. **ansible-playbook으로 스크립트 실행**
`# ansible-playbook -i ./k8s/hosts k8s-cicd-deployment-playbook.yml`
`# ansible-playbook -i ./k8s/hosts k8s-cicd-service-playbook.yml`

# 예시3 - Jenkins + Ansible + Kubernetes 연동
예시2번 과정에서 ansible-playbook 명령어를 직접 치지 않고 jenkins를 사용해 실행해보자

1. **Jenkins 관리 - SSH Server 등록**
k8s가 설치되어있는 서버를 새로 등록한다.
- username, password는 윈도우 계정 입력

![[sec4_3.png]]

2. **Item 생성**
빌드 후 커맨드에 ansible-playbook 명령어를 등록한다.
![[sec4_4.png]]

3. **배포 확인**
빌드가 완료되면 pods, deployments, service가 생성된 것을 확인할 수 있다.
```bash
hyun@DESKTOP-1S2IB77 C:\Users\박경현_인터넷망>kubectl get all
NAME                                   READY   STATUS    RESTARTS   AGE
pod/cicd-deployment-5b5795dcf7-jvf6v   1/1     Running   0          29s
pod/cicd-deployment-5b5795dcf7-rcmjt   1/1     Running   0          29s

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/cicd-service   NodePort    10.108.44.15   <none>        8080:32000/TCP   21s
service/kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP          2d1h

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cicd-deployment   2/2     2            2           29s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/cicd-deployment-5b5795dcf7   2         2         2       29s
```

service 포트인 localhost:32000에 접속하면 정상적으로 배포된 것을 확인할 수 있다.
![[sec4_5.png]]
