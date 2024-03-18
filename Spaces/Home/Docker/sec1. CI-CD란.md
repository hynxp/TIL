# docker run - 컨테이너 생성 명령어
### 1.  기본 명령어
`# docker run -p 8080:8080 -p 50000:50000 --restart=on-failure jenkins/jenkins:lts-jdk17-v`  
### 2. -v
도커를 껐을 때 데이터가 날라가지 않게 도커 바깥 저장소와 도커를 마운트(연결)하는 커맨드
`# docker run -p 8080:8080 -p 50000:50000 --restart=on-failure -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk17-d` 
### 3. -d
detach모드
현재 우리가 실행하고 있는 콘솔, 터미널과 분리한 상태에서 백그라운드(데몬)형태로 실행하겠다
`# docker run -d -v jenkins_home:/var/jenkins_home -p 8080:8080 -p 50000:50000 --restart=on-failure jenkins/jenkins:lts-jdk17--name`
### 4. --name
이름을 설정하는 것  
`# docker run -d -v jenkins_home:/var/jenkins_home -p 8080:8080 -p 50000:50000 --restart=on-failure --name jenkins-server jenkins/jenkins:lts-jdk17` 

# docker ps - 프로세스 정상 작동 확인 명령어
`# docker ps`
![[sec1_1.png]]

> [!NOTE] 8080:8080과 50000:50000이 무슨 뜻인가?
0.0.0.0:8080->8080/tcp  
0.0.0.0:50000->50000/tcp  
**8080번과 50000포트로 들어오는 요청이 컨테이너 8080과 50000이 처리해 주겠다는 뜻이다**
# docker exec - 도커 컨테이너 내부 접속 명령어
`# docker exec -it jenkins-server(컨테이너명) bash`  
![[sec1_2.png]]