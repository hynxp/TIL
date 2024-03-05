## **구조**
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

**실행 옵션**
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

