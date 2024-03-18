# SonarQube 구축
### 1. 도커 이미지 다운로드
`# docker pull sonarqube`

### 2. 도커 컨테이너 생성
`# docker run --rm -p 9000:9000 --name sonarqube sonarqube`

### 3. Jenkins 플러그인 설치
pom.xml에 플러그인 추가
```xml
<plugin>
	<groupId>org.sonarsource.scanner.maven</groupId>
	<artifactId>sonar-maven-plugin</artifactId>
	<version>3.11.0.3922</version>
</plugin>
```

### 4. user token 생성
`squ_6655709c5267c1a236a799e1aefe21bc6f21a834`
![[sec6_1.png]]

### 5. Maven에서 SonarQube 빌드
![[sec6_2.png|300]]
인텔리제이에서 빌드 명령어 입력
`mvn sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=squ_6655709c5267c1a236a799e1aefe21bc6f21a834`

sonarqube에서 프로젝트가 연결된 것을 확인할 수 있다.
![[sec6_3.png]]


# 예제1 - Bad code 조사하기

**before**
```java
@GetMapping("/")  
public String index(Model model) {  
    logger.debug("Welcome to njonecompany.com..."); 
    return "index";  
}
```

**after**
```java
@GetMapping("/")  
public String index(Model model) {  
    // logger.debug("Welcome to njonecompany.com...");
    System.out.println(model.getAttribute("today")); 
    return "index";  
}
```

코드를 위와 같이 변경하고 
### 1. 일반 빌드
`mvn clean compile package -DskipTests=true` 

그냥 빌드하게 되면 아무 이상 없이 빌드가 성공한다.
![[sec6_4.png]]

### 2. sonarqube 빌드
`mvn sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=squ_6655709c5267c1a236a799e1aefe21bc6f21a834

sonarqube를 통해 빌드하게 되면 Failed가 뜬다.
사용하지 않는 logger 필드를 제거하고 sout를 logger로 대체하라는 경고가 표시된다.
![[sec6_10.png]]![[sec6_5.png]]
# Jenkins + SonarQube 연동
### 1. Jenkins 플러그인 설치
`SonarQube Scanner for Jenkins`  설치

### 2. Jenkins credential 등록
![[sec6_6.png]]

### 3. Sonarqube 서버 등록

💡 Dashboard - Jenkins 관리 - System

> [!warning] 현재 젠킨스가 도커 컨테이너 내에서 기동되고 있기 때문에 http://localhost:9000 라고 작성하면 안 된다.

![[sec6_7.png]]

# 예제2 - Jenkins Pipeline + SonarQube 사용

### 파이프라인 스크립트 등록
이때 `withSonarQubeEnv('Sonarqube-server')`에서 위에서 등록한 SonarQube 서버 이름과 똑같은지 확인해야 한다.
```yml
pipeline {
    agent any
    tools { 
      maven 'Maven3.8.5'
    }
    stages {
        stage('github clone') {
            steps {
                git branch: 'main', url: 'https://github.com/joneconsulting/cicd-web-project.git'; 
            }
        }
        
        stage('build') {
            steps {
                sh '''
                    echo build start
                    mvn clean compile package -DskipTests=true
                '''
            }
        }
        
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('Sonarqube-server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        // stage('deploy') {
        //     steps {
        //         deploy adapters: [tomcat9(credentialsId: 'deployer_user', path: '', url: 'http://192.168.6.93:8088')], contextPath: null, war: '**/*.war'
        //     }
        // }
        
        // stage('ssh publisher') {
        //     steps {
        //         sshPublisher(publishers: [sshPublisherDesc(configName: 'docker-server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'docker build --tag hyuxp/devops_exam1 -f Dockerfile .', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        //     }
        // }
    }
}
```

# Jenkins Multi nodes 구성
### 구조
![[sec6_8.png]]

### 1. 새로운 Jenkins 서버1 생성
`# docker run --privileged --name jenkins-node1 -itd -p 30022:22 -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup --cgroupns=host edowon0623/docker:latest /usr/sbin/init`

### 2. JDK 설치
`yum install -y ncurses git`
`yum list java*jdk-devel`
`yum install -y java-11-openjdk-devel.aarch64`

### 3. Master -> node1 key 생성 및 접속
**ssh 키 생성**
`ssh-keygen`
`ssh-copy-id root@172.17.0.5` <- node1의 IP 입력

**접속**
`ssh root@172.17.0.5`

### Jenkins - node 추가

- **credential 추가**
	- ID : root
	- PWD : P@ssw0rd
![[sec6_9.png|400]]