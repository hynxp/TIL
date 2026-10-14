## 1. **기존: DB에 Refresh Token 저장 방식**

### **1) 사용자가 로그인했을 때**
- 사용자가 소셜 로그인(예: 카카오) 성공 → 서버에서 access token, refresh token 발급
- **refresh token을 DB(예: Member 테이블)에 저장**
  - 예: `member.updateRefreshToken(refreshToken); memberRepository.save(member);`
- 클라이언트에 access token, refresh token 반환

### **2) 리프레시 토큰 갱신 요청 시**
- 클라이언트가 기존 refresh token을 서버에 전달
- 서버가 **DB에서 해당 refresh token을 가진 사용자를 조회**
  - 예: `memberRepository.findByRefreshToken(refreshToken)`
- 유효성/소유자 검증 후, 새 access/refresh token 발급
- **DB에 새 refresh token으로 갱신**
  - 예: `member.updateRefreshToken(newRefreshToken); memberRepository.save(member);`
- 클라이언트에 새 토큰 반환

---

## 2. **현재: Redis에만 Refresh Token 저장 방식**

### **1) 사용자가 로그인했을 때**
- 사용자가 소셜 로그인(예: 카카오) 성공 → 서버에서 access token, refresh token 발급
- **refresh token을 Redis에 저장**
  - 예: `refreshTokenService.save(refreshToken, userId, expiresAt);`
- 클라이언트에 access token, refresh token 반환

### **2) 리프레시 토큰 갱신 요청 시**
- 클라이언트가 기존 refresh token을 서버에 전달
- 서버가 **Redis에서 해당 refresh token의 유효성 및 소유자(userId) 확인**
  - 예: `refreshTokenService.isValid(refreshToken)`, `refreshTokenService.getUserId(refreshToken)`
- **기존 refresh token을 Redis에서 삭제**
  - 예: `refreshTokenService.delete(refreshToken)`
- 새 access/refresh token 발급
- **새 refresh token을 Redis에 저장**
  - 예: `refreshTokenService.save(newRefreshToken, userId, expiresAt);`
- 클라이언트에 새 토큰 반환

---

## **핵심 차이점**

| 구분         | DB 방식                                      | Redis 방식                                      |
|--------------|----------------------------------------------|------------------------------------------------|
| 저장 위치    | DB (Member 테이블 등)                        | Redis (key-value)                              |
| 저장 방법    | 엔티티 필드에 저장 후 save()                 | Redis set(key, value, TTL)                     |
| 조회 방법    | DB 쿼리로 findByRefreshToken                 | Redis get(key)                                 |
| 갱신 방법    | DB에서 기존 토큰 찾아서 새 토큰으로 update   | Redis에서 기존 토큰 삭제 후 새 토큰 저장        |
| 삭제/무효화  | DB에서 해당 필드 null 처리                   | Redis에서 해당 key 삭제                        |
| 속도/확장성  | 느림, 트랜잭션 필요, DB 부하                 | 빠름, 세션/토큰 관리에 최적, 분산에 유리        |
| 동시성/보안  | DB 락/트랜잭션 필요, 만료 직접 관리          | TTL로 자동 만료, 분산 환경에서 일관성 보장      |

---

## **정리**
- **로그인 시**:  
  - DB 방식: DB에 토큰 저장  
  - Redis 방식: Redis에 토큰 저장
- **리프레시 토큰 갱신 시**:  
  - DB 방식: DB에서 토큰 조회/갱신  
  - Redis 방식: Redis에서 토큰 조회/삭제/저장

**Redis 방식이 훨씬 빠르고, 확장성·보안·유지보수 측면에서 우수**합니