##  NOT, <>, !=의 부정형 연산

### 부정형 연산 사용을 지양하자
**NOT, <>, !=의 부정형 연산은 [[인덱스(B-Tree, B+Tree)#인덱스란?|인덱스]] 사용이 불가**함으로 >= 연산으로 변경하여,name 인덱스를 활용하도록 개선한다.

**before**
```sql
SELECT code, NAME 
FROM athlete 
WHERE NAME IS NOT null
```

**after**
```sql
SELECT code, NAME 
FROM athlete 
WHERE NAME >= ''
```

## 컬럼에 함수 사용
### 조회 조건에 함수 사용을 지양하자

**game_date 컬럼에 TO_CHAR 함수를 사용하여 인덱스 사용이 불가**하다.
조회 대상이 되는 컬럼에 함수를 사용 할 경우, 해당 컬럼에 인덱스가 있어도 활용이 불가하다.
-> 2000년 1월부터 12월까지 조회하도록 변경

```sql
CREATE INDEX i_game_game_date ON game(game_date);

SELECT athlete_code
FROM game
WHERE TO_CHAR(game_date,'yyyy')='2000'
```

```sql
SELECT athlete_code 
FROM game 
WHERE game_date BETWEEN TO_DATE('200001','yyyymm') 
AND TO_DATE('200012','yyyymm')
```

## 인덱스 순서
### 조회 조건에 따른 인덱스 순서를 고려하자

인덱스가 이렇게 있을 때, 조회조건을 name으로 검색 할 경우 인덱스 사용이 불가하다.

**before**
```sql
CREATE INDEX idx01_athlete ON athlete(nation_code, name);
```
```sql
SELECT *
FROM athlete 
WHERE NAME LIKE 'Fernandez%'
```
**인덱스 생성 순서를 name, nation_code 순서로 변경하여 생성**한다.

**after**
```sql
CREATE INDEX idx01_athlete ON athlete(name, nation_code);

SELECT *
FROM athlete
WHERE NAME LIKE 'Fernandez%'
```

## 단일 인덱스 vs 다중 인덱스
### 조건에 따른 인덱스 개수를 고려하자

아래와 같은 쿼리가 있을 때 인덱스는 어떻게 잡아야 할까?
```sql
select * from game where stadium_code=‘30115' and nation_code=‘KOR'; 
```

단일 인덱스 여러 개의 구조 보다는 multi-column 인덱스를 생성하여 쿼리 성능을 향상할 수 있다.

**before**
```sql
create index i_game_1 on game(stadium_code);
create index i_game_2 on game(nation_code);
```

**after**
```sql
create index i_game_1 on game(stadium_code, nation_code);
```

## cost가 중요한가? 아니!
cubrid의 Q&A에 따르면 cost 보다는 trace를 보는 게 정확하다고 한다.
실제로 질문도 cost이 더 높은데 정작 수행 시간은 더 길다는 내용이었다.
![[IMG-20250716225803761.webp]]

실제로는 `show trace;`해서 trace를 보는 게 정확하다고 한다.

## trace 보는 법
최근 통계쿼리 작성하면서 이것저것 바꿔가며 trace를 확인해 보았다.

**1번 쿼리**
```sql
SELECT grp.schl_nm, 
       SUM(CASE WHEN bbs.attach_cd = 'N' THEN atch.file_sz ELSE 0 END),
       SUM(CASE WHEN bbs.attach_cd = 'M' THEN atch.file_sz ELSE 0 END),
       SUM(CASE WHEN bbs.attach_cd = 'R' THEN atch.file_sz ELSE 0 END),
       SUM(CASE WHEN bbs.attach_cd = 'C' THEN atch.file_sz ELSE 0 END),
       SUM(atch.file_sz) - SUM(CASE WHEN bbs.attach_cd = 'N' THEN atch.file_sz ELSE 0 END) - 
                          SUM(CASE WHEN bbs.attach_cd = 'M' THEN atch.file_sz ELSE 0 END) - 
                          SUM(CASE WHEN bbs.attach_cd = 'R' THEN atch.file_sz ELSE 0 END) - 
                          SUM(CASE WHEN bbs.attach_cd = 'C' THEN atch.file_sz ELSE 0 END)
FROM tb_bbs bbs
JOIN tb_grp grp ON grp.grp_no = bbs.grp_no
JOIN tb_atch_file atch ON atch.atc_seq = bbs.atc_seq
WHERE (grp.grp_tp_cd = ? OR grp.grp_tp_cd = ?)
  AND (grp.schl_cd IN (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?))
  AND atch.file_path >= ? AND atch.file_path < ?
GROUP BY grp.schl_nm;

```

**2번 쿼리**
```sql
SELECT grp.schl_nm, 
       SUM(CASE WHEN bbs.attach_cd = 'N' THEN atch.file_sz ELSE 0 END),
       SUM(CASE WHEN bbs.attach_cd = 'M' THEN atch.file_sz ELSE 0 END),
       SUM(CASE WHEN bbs.attach_cd = 'R' THEN atch.file_sz ELSE 0 END),
       SUM(CASE WHEN bbs.attach_cd = 'C' THEN atch.file_sz ELSE 0 END),
       SUM(atch.file_sz) - SUM(CASE WHEN bbs.attach_cd = 'N' THEN atch.file_sz ELSE 0 END) - 
                          SUM(CASE WHEN bbs.attach_cd = 'M' THEN atch.file_sz ELSE 0 END) - 
                          SUM(CASE WHEN bbs.attach_cd = 'R' THEN atch.file_sz ELSE 0 END) - 
                          SUM(CASE WHEN bbs.attach_cd = 'C' THEN atch.file_sz ELSE 0 END)
FROM tb_atch_file atch
JOIN tb_bbs bbs ON atch.atc_seq = bbs.atc_seq
JOIN tb_grp grp ON grp.grp_no = bbs.grp_no
WHERE (grp.grp_tp_cd = ? OR grp.grp_tp_cd = ?)
  AND (grp.schl_cd IN (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?))
  AND atch.file_path >= ? AND atch.file_path < ?
GROUP BY grp.schl_nm;

```
차이는 조인의 순서다.

gpt한테 trace 분석 해달라고 했더니 아래와 같이 답변한다.
![[IMG-20250716225803792.webp]]