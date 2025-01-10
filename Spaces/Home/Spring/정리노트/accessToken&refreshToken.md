
```
로그인을 하면 access token을 주겠죠?

access token의 유효기간은 1시간이라고 가정합니다.

사용자가 30분정도 웹페이지에 있다가, 로그아웃하지 않은 상태로 껐습니다.

그리고 2시간 후에 같은 사이트에 접속합니다.

그러면 access token은 유효기간이 만료됬네요. accessToken을 재발급 받아야 합니다.

재발급 받으려면 어떻게 해야하죠?

로그인 다시 해야죠.

2시간 마다 로그인 해야되는게 번거롭죠?

그렇다고 accessToken에 제한시간 안건다?

해커가 딱 한번만 탈취하면 해당 계정 평생 자유이용권입니다.

제한시간이 있으면, 30분이면 30분, 1시간이면 1시간 이기 때문에, 피해를 어느정도 줄여줄 수 있겠죠.

  
여하튼 이러한 이유로 1시간 만료되서 accessToken을 재발급 받으려고 다시 로그인 하면,

서버입장에서는

1. database에서 SELECT Member table 도 해야하고,
2. 찾은 Member를 UserDetails로 변환해 검증도 해야하고,
3. 검증이 끝났으면 spring security context에 해당 유저정보를 Principal로 저장도 해야하고,
4. jwt token도 만들어야 하고,
5. refresh token도 만들어야 하고
6. refresh token을 db에 저장해야 하고,
7. access token는 http response에, refresh token를 cookie에 담아 보내줘야 하고..

등, login 하는 작업에서 약간 할게 많죠.

  

그런데 만약에 유효기간이 1주일인 refresh token이 있으면,

굳이 번거로운 login process를 1시간 마다 거치지 않아도,

사용자가 http request시, 만료된 access Token과 아직 살아있는 refresh token을 보내면, 서버 입장에서는

1. database에 refresh token이 있는지 확인만 되면,
2. 바로 access token 만들어서 뿌립니다.  
    

처리하는 프로세스 량이 줄었죠.


근데 작성자님께서 질문하신 것 처럼, 아니 refresh token을 database에서 관리하고, 사용자 정보랑 토큰 있는데, 왜 굳이 accessToken 쓰지? 라는 의문이 드실 수 있는데,

jwt token은 STATELESS해서 씁니다. 만약 사용자 정보를 database에 저장해서 요청올 때마다 db에 io해서 authenticate 할거였으면, STATEFUL 방식인 세션을 썼겠죠? jwt token 방식을 안쓰고.

그럼 STATEFUL 방식을 놨두고 왜 STATELESS 방식을 쓰느냐?

요새 온프레미스에서 클라우드로 넘어가면서, 오토스케일 중에 horizontal scale 방식을 씁니다.

이게 뭐냐면, 사용자 1000명이었을 땐, 서버가 1개였다가, 갑자기 급증해서 10000명 되면 클라우드 플랫폼이 이를 감지해서 서버를 10개로 늘리는걸 horizontal scale이라고 합니다.

STATELESS 방식을 쓰는 이유 중 하나가 여기에 있습니다.

만약 서버1에서 STATEFUL 방식인 세션 방식을 쓰고 있다고 가정했을 때, 유저1부터 유저100까지 정보가 서버1에서 관리되겠죠?

근데 트래픽이 10000명으로 늘어서 서버가 10개로 증가하면, 서버 2부터 10까지는, 유저 1~100의 정보가 없잖아요?

그렇다고 유저1 부터 100이 서버1에 로그인 했으면, 100명의 정보를 서버2부터 10까지 공유한다?

사실상 어렵겠죠. 동기화 맞추는 것도 일일테구요.

  

그러니까, STATEFUL방식이 아닌 STATELESS방식을 써서,

유저 아이디 + 서버에 적어놓은 private key를 hash()함수 돌려서 나온 해시 값인 jwt token을 유저에게 보내고,

유저가 서버 1이든 서버 5든 10이든 http request를 jwt token으로 보내면,

서버에서는 간편하게 유저 아이디 + 서버에 적어놓은 private key를 hash()함수 돌려서 얻은 값과 String.equals() 비교만 하면, 굳이 database io나 스프링 시큐리티에 세션 객체를 만들지 않아도 authenticate()를 해주니까,

유저의 세션 정보를 서버 1부터 10까지 공유할 필요가 없어지죠.

그러니 STATELESS방식이 horizontal scale에 잘 받는 요즘 클라우드 환경 유용한 방식이구요.
```



okky 댓글에 달렸던 내용인데 너무 이해가 잘되서 긁어왔다.
다시 주소를 찾으려니 못찾겠음.. 