## 인덱스 적용 조건
(인덱스 **(a,b,c)** 인 경우)

- `order by b, c`
    - 인덱스 첫번째 컬럼인 **a가 누락**되어 사용 불가
- `order by a, c`
    - 인덱스에 포함된 **b 컬럼이 a, c 사이에 미포함**되어 사용 불가
- `order by a, c, b`
    - 인덱스 컬럼과 order by 컬럼간 **순서 불일치**로 사용 불가
- `order by a, b desc, c`
    - b 컬럼의 `desc` 로 인해서 사용 불가
- `order by a, b, c, d`
    - 인덱스에 **존재하지 않는 컬럼** d로 인해 사용 불가

```sql
WHERE a = 1 
ORDER BY b, c

WHERE a = 1 and b = 'b'
ORDER BY c
```
이 쿼리는 인덱스 적용이 가능하다. a가 없는데 왜 되는걸까?
실제로 `where a = 1 ORDER BY a, b, c`와 동일한 쿼리이기 때문이다.

## 참고
[2. 커버링 인덱스 (WHERE + ORDER BY / GROUP BY + ORDER BY ))](https://jojoldu.tistory.com/481)
