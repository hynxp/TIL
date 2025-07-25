##  트랜잭션 개발 가이드
1. 데이터 오너십 원칙
2. 실패한 트랜잭션의 처리
	1. 트랜잭션의 취소/재시도
3. 트랜잭션 격리성 보완
	1. 레코드/트랜잭션 잠금
4. 신뢰할 수 있는 이벤트 전송

### 1. 데이터 오너십 원칙
Service에서 차마조
1. 테이블
2. DAO(Repo)
3. ServiceImpl

모듈화 컨셉 3 > 2 > 1
3번이 익숙한 사람이 MSA에 좀 더 익숙?할 것?

### 2. 실패한 트랜잭션의 처리
트랜잭션이 길 경우 트랜잭션을 나눈다
서비스간 트랜잭션은 롤백 기능이자동이 아니기 때문에 보상 트랜잭션을 구현을 해줘야 된다.

Pivot Transaction
: 트랜잭션 중 에러가 발생했을 때 재시도할지 취소할지를 결정하는 기준

실패하면 트랜잭션 취소(피봇 트랜잭션이 수행되기 전에)
- 사용자는 응답 대기
- 재시도는 사용자가 주관

피봇 트랜잭셩 이후에는
- 실패하면 이벤트로 재시도
- 계속 실패하면 시스템 담당자가 처리

! API로 재시도 처리하면 위험할 수 있음

#### 수강 신청 예시
1. 수강 신청
2. 신청 내역 검토
3. 신청 정보 생성
4. 커뮤니티 멤버 등록
5. 실습 리소스 생성
6. 커밋

모놀리식이면 한 트랜잭션으로 해도 ㄱㅊ
MSA에서 5번에 실패가 났다고 수강 신청을 거절하는 게 맞나?
3번까지를 피봇으로 두고, 그 이후거는 이벤트로 분리

사가 패턴


### 3. 트랜잭션 격리성 보완
팬텀 리드 같은 문제들 해결법
1. Semantic Lock
2. TCC(Try, Confirm, Cancel)
	1. Try Commit/Confirm Commit 분리하는 방법
3. Offline Lock

### 4. 신뢰할 수 있는 이벤트 전송
보통 Transactional Outbox Pattern 사용

진짜 실패, 가짜 실패 구분
- 오류를 정확하게 던지면 진짜 실패
- 가짜 실패 복구하는 법
	- 멱등성
	- 실패 정보의 유실 방지

실패 정보의 유실 방지
중간에 AOP 둬서 API 로그를 남기자
- API 요청 기록 Commit
- API 실패 기록 Commit

멱등성
1. 중복 실행
	1. 유니크 값으로 실행
	2. 버전 정보를 함께 전송
		1. 이전 버전보다 지금 버전이 작아야 후에 온거니까 커밋
		2. V2>V0(O) V1<V2(X)

