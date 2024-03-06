# 구조
![[sec3-20240305112647002.webp]]
**도커 네트워크 브릿지 확인**  
`docker network inspect bridge`

**host PC에서 각 도커에 접속할 때는 아래처럼 접속**  
`ssh root@localhost -p 20022`  
`ssh root@localhost -p 10022`

**도커 컨테이너 안에서 다른 컨테이너로 접속할 때**
`ssh root@172.17.0.2`  
`ssh root@172.17.0.4`

**ssh 키 등록**
ansible 서버에서 docker 서버로 접속할 때 계속 비밀번호를 치면 번거로우니
ssh키를 생성해서 docker서버에 전달하면 바로 접속이 가능하다
`ssh-keygen`
`ssh-copy-id root@172.17.0.4`(접속할 서버 IP)

## 기본 명령어
`-i (--inventory-file)`
적용 될 호스트들에 대한 파일 정보
`-m (--module-name)`
모듈 선택
`-k (--ask-pass)`
관리자 암호 요청
`-K (--ask-become-pass)`
관리자 권한 상승
`--list-hosts` 
적용되는 호스트 목록

Ansible은 멱등성을 보장한다
-> 같은 설정을 여러 번 적용하더라고 결과가 달라지지 않는다. (한 번만 적용된다)

# 모듈 사용
[모듈 목록](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html)

- **접속 확인**
 `ansible all -m ping`
 `ansible devops -m ping`
all -> hosts 파일에 있는 대상 전체 다
devops -> hosts 파일에 있는 그룹명
-m -> 모듈 사용
ping -> ping이라는 모듈을 사용한다.
```json
172.17.0.2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
172.17.0.4 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

- **메모리 사용량 정보 반환**
`ansible all -m shell -a "free -h"`
```shell
[root@ea3a4e776da2 ~]# ansible all -m shell -a "free -h"
172.17.0.2 | CHANGED | rc=0 >>
total        used        free      shared  buff/cache   available
Mem:           15Gi       1.8Gi       9.6Gi        30Mi       4.1Gi        13Gi
Swap:         4.0Gi          0B       4.0Gi
172.17.0.4 | CHANGED | rc=0 >>
total        used        free      shared  buff/cache   available
Mem:           15Gi       1.8Gi       9.6Gi        30Mi       4.1Gi        13Gi
Swap:         4.0Gi          0B       4.0Gi
```

- **패키지 설치**
`ansible devops -m yum -a "name=httpd state=present"`

# Playbook 사용하기
모듈에 대한 명령어를 작성해놓고 한꺼번에 실행할 수 있다.

- **blockinfile 모듈 사용** (마커 선으로 둘러싸인 여러 줄의 텍스트 블록 삽입)
1. first-playbook.yml 생성
```yml
---
  - name: Add an ansible hosts
    hosts: localhost
    tasks:
      - name: Add a ansible hosts
        blockinfile:
          path: /etc/ansible/hosts
          block: |
            [mygroup]
            172.17.0.5
```
2. `# ansible-playbook first-playbook.yml` 실행
```shell
[root@ea3a4e776da2 ~]# cat /etc/ansible/hosts 
[devops]
172.17.0.2 
172.17.0.4
# BEGIN ANSIBLE MANAGED BLOCK
[mygroup]
172.17.0.5
# END ANSIBLE MANAGED BLOCK
```

2번 과정을 한 번 더 한다고 해도 host가 또 추가되지 않는다.
아까 말했던 **멱등성**때문이다!!! 

- **copy 모듈 사용**(파일 복사)
```yml
- name: Ansible Copy Example Local to Remtoe 
  hosts: devops
  tasks:
    - name: copying file with playbook
      copy:
        src: ~/sample.txt
        dest: /tmp
        owner: root
        mode: 0644
```

- **file, get_url 모듈 사용**(url을 가져와서 파일 다운로드)
```yml
---
- name: Download Tomcat9 from tomcat.apache.org
  hosts: all
  #become: yes
  # become_user: root
  tasks:
   - name: Create a Directory /opt/tomcat9
     file:
       path: /opt/tomcat9
       state: directory
       mode: 0755
   - name: Download the Tomcat checksum
     get_url:
       url: https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.86/bin/apache-tomcat-9.0.86.tar.gz.sha512
       dest: /opt/tomcat9/apache-tomcat-9.0.86.tar.gz.sha512
   - name: Register the checksum value
     shell: cat /opt/tomcat9/apache-tomcat-9.0.86.tar.gz.sha512 | grep apache-tomcat-9.0.86.tar.gz | awk '{ print $1 }'
     register: tomcat_checksum_value
   - name: Download Tomcat using get_url
     get_url:
       url: https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.86/bin/apache-tomcat-9.0.86.tar.gz
       dest: /opt/tomcat9
       mode: 0755
       checksum: sha512:{{ tomcat_checksum_value.stdout }}"
```

# Jenkins + Playbook 사용하기
```shell
- hosts: all
#   become: true  

  tasks:
  - name: build a docker image with deployed war file
    command: docker build -t cicd-project-ansible .
    args: 
        chdir: /root

  - name: create a container using cicd-project-ansible image
    command: docker run -d --name my_cicd_project -p 8080:8080 cicd-project-ansible
```
### 여기서 8080:8080은 어떻게 설정하느냐?
host PC(실제 컴퓨터)에서 ansible server를 **8082:8080**로 실행시켰다.
(localhost:8082로 요청이 들어오면 ansible server 내부에서 8080포트가 응답하겠다는 의미다.)

즉 8080포트가 응답하기 때문에 ansible server내에 컨테이너에서는 8080의 요청을 받아야 한다.
8080:8080 , 8080:8081 앞에만 8080이면 된다.

--- 

1. **Jenkins 관리 설정**
SSH Server에 ansible-server 등록
![[Pasted image 20240306111524.png]]

2.  **프로젝트 설정**
해당 프로젝트 빌드 후 조치에 war 파일이 빌드되면 ansible-server에서 ansible-playbook.yml에 등록한 명령어가 자동 실행되도록 설정한다.
![[Pasted image 20240306111443.png]]

3. **localhost:8082로 접속해서 확인**
![[Pasted image 20240306112926.png]]

---
이 상태에서 같은 이름의 컨테이너가 이미 존재하면 UNSTABLE 에러가 발생하기 때문에 
playbook 파일에서 기존에 컨테이너가 존재하면 중지하고 삭제하는 명령어를 추가한다.

```ad-note
~~~shell
- hosts: all
#   become: true  

  tasks:
  - name: stop current running container
    command: docker stop my_cicd_project
    ignore_errors: yes

  - name: remove stopped cotainer
    command: docker rm my_cicd_project
    ignore_errors: yes

  - name: remove current docker image
    command: docker rmi cicd-project-ansible
    ignore_errors: yes

  - name: build a docker image with deployed war file
    command: docker build -t cicd-project-ansible .
    args: 
        chdir: /root

  - name: create a container using cicd-project-ansible image
    command: docker run -d --name my_cicd_project -p 8080:8080 cicd-project-ansible
~~~
```
