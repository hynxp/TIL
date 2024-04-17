## 인덱스 적용 조건
- GROUP BY 절에 명시된 컬럼이 **인덱스 컬럼의 순서와 같아야 한다**.
    - 아래 모든 케이스는 **인덱스가 적용 안된다**. (index: a,b,c)
    - `group by b`
    - `group by b, a`
    - `group by a, c, b`
- 인덱스 컬럼 중 **뒤에 있는 컬럼이 GROUP BY 절에 명시되지 않아도** 인덱스는 사용할 수 있다.
    - 아래 모든 케이스는 **인덱스가 적용된다**. (index: a,b,c)
    - `group by a`
    - `group by a, b`
    - `group by a, b, c`
- 반대로 인덱스 컬럼 중 **앞에 있는 컬럼이 GROUP BY 절에 명시되지 않으면** 인덱스를 사용할 수 없다
    - ex: (index: a,b,c), `group by b, c` 는 **인덱스 적용안됨**
- 인덱스에 없는 컬럼이 GROUP BY 절에 포함되어 있으면 인덱스가 적용되지 않는다.
    - ex: (index: a,b,c), `group by a,b,c,d` 는 **인덱스 적용안됨**


## 참고
[1. 커버링 인덱스 (기본 지식 / WHERE / GROUP BY)](https://jojoldu.tistory.com/476)