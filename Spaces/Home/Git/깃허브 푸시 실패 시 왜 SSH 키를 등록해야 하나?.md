최근에 깃허브에서 **푸시(push) 오류**를 해결 하기 위해 SSH 키를 등록했었는데, 이때까지는 검색이나 AI가 알려준대로 했는데 궁금해져서 알아보았다.


## 오류
처음 내가 만난 에러 메시지는 아래와 같다.
```bash
remote: Permission to grow-together-study/system-design-interview.git denied to hynxp.
fatal: unable to access 'https://github.com/grow-together-study/system-design-interview.git/': The requested URL returned error: 403
```
이 오류가 뜨는 이유는 깃허브가 **누구인지 확인할 수 없는 사용자에게는 권한을 주지 않기 때문**이라고 한다.
즉, 인증 실패다.

깃허브에서 레포지토리에 푸시(push)하거나 clone을 할 때 가장 자주 발생하는 문제는 인증 문제다.
깃허브는 “이 요청을 보낸 사용자가 이 레포에 접근할 권한이 있는가?”를 항상 체크한다.

예전에는 GitHub 계정의 ID와 비밀번호로 HTTPS 방식 인증을 했지만, 요즘은 Personal Access Token(PAT)을 쓰거나, SSH 키 방식을 쓰는 경우가 많다

SSH 방식에서는 로컬 컴퓨터에 있는 **SSH 키**와 깃허브 계정에 등록된 **공개키**가 쌍으로 맞아야 통과할 수 있다.


## 왜 SSH 키를 등록해야 하나?
SSH 방식은 비밀번호나 토큰을 직접 입력하지 않아도 되는 **공개키/비밀키(Public/Private Key)** 방식의 인증이다. 

개발자라면 아래 명령어로 SSH 키를 등록한 경험이 한 번쯤은 있을 것이다.
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

깃허브는 크게 두 가지 인증 방식을 제공한다

1. HTTPS → 깃허브 ID + 비밀번호 또는 PAT(Personal Access Token)

2. SSH → 공개키/비밀키 기반의 SSH 인증
  

### SSH 작동 방식
SSH 방식은 비밀번호나 토큰 없이, 로컬에 저장된 `비밀키(private key)`로 깃허브가 가진 `공개키(public key)`와 암호화 통신해 신원을 인증한다.

흐름은 아래와 같다.

1. 로컬에서 SSH 접속을 시도하면, 비밀키로 인증 응답을 암호화해서 보냄
    
2. GitHub는 등록된 공개키로 해당 응답을 검증함
    
3. 일치하면 “권한 있음”으로 통과

이를 위해서는

- 로컬 컴퓨터에서 ssh-keygen으로 SSH 키 쌍을 생성하고

- 깃허브 계정에 .pub 파일(공개키)을 등록해줘야 한다.

등록이 안 되어 있다면 깃허브는 
“너 누구냐?” → “몰라, 입구컷” → Permission denied (publickey) 에러 발생 두둥!


## HTTPS vs SSH

|**구분**|**HTTPS**|**SSH**|
|---|---|---|
|인증 방식|GitHub ID + 비밀번호 또는 PAT|SSH 공개키/비밀키|
|인증 입력|비밀번호/PAT 입력 (혹은 캐시 사용)|최초 1회 SSH 키 등록 후 자동 인증|
|보안성|안전하지만 비밀번호/PAT 관리 필요|키 기반 인증, 비밀번호 노출 위험 없음|
|추천 용도|개인/GUI 위주 작업|CLI-heavy, 협업, 자동화 작업|
SourceTree 같은 GUI 툴은 HTTPS로 쓰는 경우가 많고, 터미널에서 git push/pull을 자주 하는 개발자는 SSH 방식이 훨씬 편하다.


## 그냥 원격 주소를 SSH로 변경하면 안될까?
처음에는 아래랑 뭐가 다르지? 했다.
```bash
git remote set-url origin git@github.com:ORG/REPO.git
```
이렇게 git remote set-url로 원격 주소를 SSH로 바꾸면 안되나?

#### git remote set-url origin ...
이건 깃이 연결할 원격 주소를 바꾸는 명령어고, 주소 형식만 바꿀 뿐, 실제 인증은 따로 해결해야 한다.

#### SSH 키 등록
이 작업은 SSH 방식에서 “내가 누구인지”를 증명하는 인증 수단이다.
깃허브에 등록된 공개키와 로컬의 비밀키가 맞아야 SSH 연결이 가능하다.

즉, remote 주소만 SSH로 바꾼다고 자동으로 인증 문제가 해결되는 것이 아니다.
반드시 SSH 키 등록까지 완료해야 SSH 방식이 정상 작동한다.

