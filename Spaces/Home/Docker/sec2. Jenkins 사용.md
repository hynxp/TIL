# 빌드 후 조치 

**구조**
![[sec2_1.excalidraw]]

위 구조에서 Jenkins 빌드 후 조치 - Deploy war/ear to a container에서 war파일을 톰캣으로 배포하고 싶을 때 Tomcat URL을 localhost:8088 or http://127.0.0.1:8088/로 입력하면 안된다!

![[sec2_2.webp|200]]

```ad-question
title: 왜?
Jenkins에서는 이 경로 자체를 자신이 가지고 있는 경로로 접속을 시도한다.(루프백)
하지만 현재 Jenkins 컨테이너 내에 http://127.0.0.1:8088/는 없기 때문에 **HOST IP:8080**로 입력해야 한다.

만약 도커 내에 젠킨스를 설치한 게 아니라 윈도우 자체에 젠킨스를 별도로 설치한 경우에는 로컬호스트로 입력해도 된다.
```

# 빌드 유발
### Build periodically
코드에 변경사항이 없어도 일단 빌드를 다시 함  
### Poll SCM
커밋된 내용이 있어야지만 빌드를 다시 함

# 도커 내에 SSH 도커 컨테이너 구축

## 구조
![[sec2_3.webp]]
Password: **_P@ssw0rd_**

## SSH 접속 및 Jenkins와 SSH 서버 연동
### ssh 접속 명령어
`ssh root@localhost -p 10022` 

## Jenkins에 SSH 서버 등록
![[sec2_4.webp]]
### Hostname
젠킨스 내에 SSH 설정 입력할 때도 localhost를 입력하면 안 된다.  
localhost자체가 루프백(본인을 가리킴)하기 때문에 젠킨스 내에 10022번 포트는 없으므로  본인 IP를 입력해야 함!

### Remote Directory
처음에 ssh 서버에 접속하면 /root 폴더로 이동하는데 여기에다가 war파일을 복사하겠다는 의미다.


# Dockerfile
이 파일을 통해 도커 이미지를 생성한다.

```ad-note
title: Dockerfile 예시
~~~
FROM tomcat:9.0 
COPY ./hello-world.war /usr/local/tomcat/webapps
~~~
tomcat:9.0 이미지를 기반으로 하고,  
루트 디렉토리 밑에 hello-word.war 파일을 /usr/local/tomcat/webapps로 복사해라
```

# Docker 기본 명령어들
### 이미지 빌드 명령어(Container 내에서 테스트)
`# docker build --tag docker-server -f Dockerfile .`

### 컨테이너 실행 명령어 (Container 내에서 테스트)
`# docker run --privileged_ -p 8080:8080 --name mytomcat docker-server:latest`

```ad-question
title: 왜 8080:8080이냐
맨 처음에 강사의 도커 이미지로 컨테이너 만들 때 8081->8080 이렇게 만들었다.  
1. pc에서 8081로 요청하면  ssh 서버 컨테이너 내부에서 8080이 응답한다.
2. 컨테이너 내부에서 8080요청이 들어오면 컨테이너에 있는 또 다른 컨테이너 8080이 응답한다.
```

### 컨테이너 중지, 삭제  명령어
`# docker stop mytomcat`  
`# docker rm mytomcat`