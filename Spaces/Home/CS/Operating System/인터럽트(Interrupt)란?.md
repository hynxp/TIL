인터럽트는 **CPU의 작업을 방해하는 신호**를 의미한다. 
CPU가 명령어를 실행하는 도중 예상치 못한 상황이 발생하면, 이를 처리하기 위해 인터럽트가 발생한다.
![[IMG-20250212133228899.png|300]]


## 동기 인터럽트(Synchronous Interrupt)
`동기 인터럽트`는 CPU에 의해 발생하는 인터럽트이다.

CPU가 프로그램 실행 도중 예상치 못한 상황이 발생하면 CPU가 해당 문제를 처리하기 위해 인터럽트를 발생시킨다.
![[IMG-20250212133228957.png|300]]

예를 들어, 프로그래밍상의 오류와 같은 예외적인 상황에서 발생하는 인터럽트라고 해서 동기 인터럽트는 `예외(exception)`라고도 부른다.

### 예외(Exception)의 종류
![[IMG-20250212133229043.png|400]]
예외가 발생하면 CPU는 하던 일을 중단하고 해당 예외를 처리한다.
이후 예외를 처리하고 원래 하던 작업으로 돌아와 실행을 재개한다.

예외는 발생 후 CPU가 **예외 발생 지점부터 실행을 재개하느냐**, **다음 명령어부터 실행을 재개하느냐**에 따라 아래와 같이 분류된다.

#### 1. 폴트(fault)
폴트는 예외를 처리한 직후 예외가 발생한 명령어부터 다시 실행하는 경우를 말한다.

만약 CPU가 A명령어를 실행하려고 하는데, 이 **명령어를 실행하기 위해 필요한 데이터가 메모리가 아닌 보조기억장치에 있다고 가정**해 보자.

프로그램이 실행되려면 반드시 메모리에 저장되어 있어야 하기때문에 CPU는 `폴트`를 발생시키고 보조기억장치에있는 데이터를 메모리로 가져온다.
데이터를 가져왔으니 CPU는 다시 실행을 해야한다.
이때 CPU는 폴트가 발생한 그 명령어부터 실행한다.

이렇게 예외 발생 직후 예외가 발생한 명령어부터 실행해 나가는 예외를 폴트라고 한다.

> 페이지 폴트(page fault)가 대표적인 예다.

#### 2. 트랩(trap)
`트랩`은 예외를 처리한 후 **예외가 발생한 명령어의 다음 명령어부터 실행**하는 경우를 말한다.

주로 디버깅할 때 사용된다!
개발자가 특정 코드가 실행될 때 프로그램을 멈추도록 설정하면, CPU는 해당 명령어를 실행한 후 인터럽트를 발생시키고 멈춘다. 이후 디버깅이 끝나면(트랩을 처리하면) **다음 명령어부터 실행을 이어간다.**


#### 3. 중단(abort)
`중단`은 CPU가 실행 중인 프로그램을 강제로 중단시킬 수밖에 없는 심각한 오류가 발생했을 때 발생하는 예외다.
하드웨어 오류나 메모리 손상이 `중단`에 해당한다.

#### 4. 소프트웨어 인터럽트(software interrupt)
소프트웨어 인터럽트는 [[커널, 이중모드, 시스템 호출#시스템 호출(system call)|시스템 호출]]을 실행할 때 발생하는 인터럽트다.
주로 파일을 읽거나, 네트워크 요청을 보낼 때 운영체제의 기능을 호출하는 과정에서 발생한다.



## 비동기 인터럽트(Asynchronous Interrupt)
`비동기 인터럽트`는 **CPU가 실행하는 명령어와 무관하게 발생하는 인터럽트**로, 주로 **입출력 장치(I/O Device)**에 의해 발생한다.

입출력장치에 의한 `비동기 인터럽트`는 세탁기 완료 알림, 전자레인지 조리 완료 알림과 같은 알림 역할을 한다.

![[IMG-20250212133229159.png|500]]


예를 들어, CPU가 프린터에 "출력해!"라고 명령을 내린 후, 프린터가 작업을 끝내면 "출력 완료!"라는 신호(인터럽트)를 CPU에 보낸다. 이를 통해 CPU는 **프린터가 작업을 완료할 때까지 대기하지 않고 다른 작업을 수행**할 수 있다.

일반적으로 `비동기 인터럽트`를 인터럽트라 칭하기도 하지만, 보통은 `하드웨어 인터럽트`라는 용어를 사용한다.

### 하드웨어 인터럽트의 역할
위에서 하드웨어 인터럽트는 알림 역할을 한다고 했다.
`하드웨어 인터럽트`는 **CPU가 효율적으로 명령어를 처리할 수 있도록 돕는 기능**을 한다.

#### 🤔 명령어를 효율적으로 처리하는거랑 하드웨어 인터럽트가 뭔 상관이지?
예를 들어 **CPU**가 프린터에 "출력해!"라고 명령했다고 가정해 보자.

입출력장치는 CPU보다 속도가 현저히 느리기 때문에 CPU는 입출력 작업의 결과를 바로 받아볼 수 없다.
이때 `하드웨어 인터럽트`를 사용하지 않는다면 CPU는 프린터가 언제 프린트를 끝낼지 모르기 때문에 주기적으로 프린터의 완료 여부를 확인해야 한다.
이로 인해 CPU는 다른 작업을 할 수 없으니 CPU 사이클이 낭비된다.
이는 우리가 전자레인지를 돌려놓고 가만히 전자레인지 앞에서 언제 끝나나 쳐다보고 있는 상황인 것이다ㅎㅎ

이때 `하드웨어 인터럽트`를 사용하면 CPU는 주기적으로 프린트가 끝났는지 확인할 필요 없이 프린터로부터 "프린트 끝났어!"라는 알림을 받을 때까지 다른 작업을 처리할 수 있다.
우리는 **전자레인지를 돌려놓고 다른 일을 하다가 "삐-" 소리를 들으면 음식을 꺼내면 된다.**



## 요약
|구분|설명|예제|
|---|---|---|
|**동기 인터럽트**|CPU가 실행하는 명령어 때문에 발생하는 인터럽트|프로그램 오류, 시스템 호출|
|**비동기 인터럽트**|CPU의 실행과 관계없이 입출력 장치 등에 의해 발생하는 인터럽트|프린터 출력 완료, 키보드 입력 감지|
|**폴트(Fault)**|예외 처리 후, 예외가 발생한 명령어부터 실행|페이지 폴트|
|**트랩(Trap)**|예외 처리 후, 다음 명령어부터 실행|디버깅 중단점(Breakpoint)|
|**중단(Abort)**|심각한 오류로 인해 프로그램 강제 종료|메모리 손상|
|**소프트웨어 인터럽트**|시스템 호출(System Call)에 의해 발생|파일 읽기, 네트워크 요청|
|**하드웨어 인터럽트**|입출력 장치가 작업을 완료한 후 CPU에 알리는 인터럽트|프린터 출력 완료 알림|




참고
[혼자 공부하는 컴퓨터 구조+운영체제](https://m.yes24.com/Goods/Detail/111378840)