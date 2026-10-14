---
aliases:
  - Outer Join 순서 조정으로 I/O 90 % 절감
---
**Outer Join 순서 조정으로 I/O 90 % 절감**

- **문제 상황**
    - 숙제 목록 조회 SQL에서 `tb_hmwkpp`(제출내역) 테이블을 **가장 먼저 조인**하며 Full Scan 발생
    - 필터 조건이 충분하지 않은 상태에서 조기 접근되어, **불필요한 Fetch Page 138,625건 발생**
    - 최대 수행 시간 6초 이상으로 확인됨
- **해결 방법**
    - 실행 계획(Explain Plan)을 분석해 `tb_hmwkpp` 접근 순서가 비효율적임을 확인
    - Outer Join 순서를 재배치해 **선택성 높은 테이블들(hmwk, mbr 등)을 먼저 조인**하도록 조정
    - `tb_hmwkpp`는 가장 마지막에 조인해 **필터 후 접근**되도록 구조 변경
- **결과**
    - **Fetch Page 138,625 → 13,969**로 약 90% 가까이 감소
    - 응답 속도도 **6초 → 0.2초**로 대폭 개선


6 sec(문제 발생시점 최대 수행속도)
최근 고도화 프로젝트에서 아래처럼 REPLACE문을 거의 1n중으로 감싼 하드코딩 쿼리를 발견했다.

이 프로젝트는 기간이 굉장히 타이트해서 계속 야근하느라 빨리 구현해야 됐었다.

이 쿼리 또한 AS-IS 프로젝트의 쿼리 로직을 그대로 사용해도 되는거였으나 차마 그냥 쓰기가 싫어서 함수로 만들었다.

 공통 테이블 기반의 함수들은 만들어두면 다른 개발자들도 사용할 수 있어서 유용하다.

간단한 예시로 포스팅하고자 한다.

레쓰고

```
SELECT REPLACE(REPLACE(REPLACE(your_column_name, 'APPLE', '애플'), 'BANANA', '바나나'), 'MANGO', '망고') AS FRUITS
FROM comm_cd
```

이 쿼리는 \`APPLE,BANANA,MANGO\` 이렇게 저장된 문자열의 값을 '애플,바나나,망고' 로 치환시켜주는 쿼리다.

이런 쿼리는 남이 보기에 이해하기도 힘들뿐더러, 대체 닫는 괄호가 어디를 닫는건지도 파악하기 힘들다.

또 나중에 과일 종류가 추가된다면 개수마다 REPLACE문을 추가해줘야 한다.

정말 생산성이 떨어진다..

간단한 예시로 이 값들을 공통 테이블에 넣어 함수로 치환해 보자.

## TABLE

공통 테이블 \`COMM\_CD\`

[##_Image|kage@cL8OzF/btsuZUQvIvZ/AAAAAAAAAAAAAAAAAAAAABCDCfbg_rbeJNaE4ikjogqjL4IVLAb33mCrV1wfbsN8/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&amp;expires=1750417199&amp;allow_ip=&amp;allow_referer=&amp;signature=TQx97krpbmIdV0k%2BUdENSQy60pg%3D|CDM|1.3|{"originWidth":394,"originHeight":212,"style":"alignLeft","width":245,"height":132}_##]

데이터

[##_Image|kage@ydTpI/btsu17BPG5I/AAAAAAAAAAAAAAAAAAAAAIz3VPEkNZEyPRd6j-DdPdF40NudVIilxPGLKmJZByqj/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&amp;expires=1750417199&amp;allow_ip=&amp;allow_referer=&amp;signature=UMGRKSl4grmIcflzvsGRdhTM2Do%3D|CDM|1.3|{"originWidth":1203,"originHeight":185,"style":"alignLeft","width":632,"height":97}_##]

조회 테이블 \`TB\_FRUITS\`

[##_Image|kage@baUWbr/btsu7OPsuDF/AAAAAAAAAAAAAAAAAAAAAIAGa_l6BOF7PNPnrNb17cA4rHXjrx2JrJOkcnayx_5B/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&amp;expires=1750417199&amp;allow_ip=&amp;allow_referer=&amp;signature=UWTrC4k5oT8Z2lxidhpnd0DcaWE%3D|CDM|1.3|{"originWidth":378,"originHeight":128,"style":"alignLeft","width":236,"height":80}_##]

## 데이터

[##_Image|kage@dgA5RR/btsu7SdffsP/AAAAAAAAAAAAAAAAAAAAANH9yGFlCURQ8BurxoyW1i4bkenvdZtJjTvin38VUNtR/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&amp;expires=1750417199&amp;allow_ip=&amp;allow_referer=&amp;signature=SlAw7lCmlamMB2DLJFXHkQ2cHIE%3D|CDM|1.3|{"originWidth":1203,"originHeight":183,"style":"alignLeft","width":643,"height":98}_##]

공통 테이블과 조회 테이블이 이렇게 있다고 가정했을 때 함수는 이렇게 만들 수 있다.

아래에서 하나씩 이해해 보자. 

```
CREATE OR REPLACE FUNCTION fn_get_comm_code_nm(p_code text, p_code_id text)
 RETURNS text
 LANGUAGE plpgsql
AS $function$
DECLARE
    v_result text := '';         -- 결과값
    v_delimiter text := ',';     -- 구분자
    v_split_cnt int := 0;        -- 낱개 갯수
BEGIN
    -- 하나의 컬럼에 구분자(',')로 묶여져 있는 코드를 공통코드에서 찾아서 코드명으로 변경
    -- ex) 'APPLE,BANANA' → '사과, 망고'
    -- p_code : 입력텍스트
    -- p_code_id : 그룹코드

    IF p_code IS NULL OR TRIM(p_code) = ''
    THEN
        RETURN '';
    ELSE
        SELECT COUNT(CODE_NM) INTO v_split_cnt
        FROM COMM_CD
        WHERE CODE_ID = ANY(string_to_array(p_code_id, v_delimiter))
        AND CODE = ANY(string_to_array(p_code, v_delimiter));

        IF v_split_cnt > 0
        THEN
            SELECT replace(
                        array_to_string(
                            array_agg(
                                array(
                                        SELECT CODE_NM
                                        FROM COMM_CD
                                        WHERE CODE_ID = ANY(string_to_array(p_code_id, v_delimiter))
                                        AND CODE = ANY(string_to_array(p_code, v_delimiter))
                                )
                            ), v_delimiter
                        ), v_delimiter, ', '
                    ) INTO v_result;
        END IF;

    END IF;

    RETURN v_result;
END;
$function$
```

## 과정

1\. 입력된 \`p\_code\`가 NULL이거나 공백 문자열인지 확인하여 빈 문자열을 반환한다.

```
IF p_code IS NULL OR TRIM(p_code) = ''
    THEN
        RETURN '';
```

2\. 공백이 아닐 경우, 공통 코드 테이블(COMM\_CD)에서 주어진 \`p\_code\`와 \`p\_code\_id\`를 사용하여 코드명을 검색한다.

3\. \`COUNT(CODE\_NM)\`를 통해 코드의 수를 확인하고 그 값을 \`v\_split\_cnt\` 변수에 저장한다.

```
SELECT COUNT(CODE_NM) INTO v_split_cnt
FROM COMM_CD
WHERE CODE_ID = ANY(string_to_array(p_code_id, v_delimiter))
AND CODE = ANY(string_to_array(p_code, v_delimiter));
```

4\. \`v\_split\_cnt\`가 1 이상이면 \`p\_code\`로 들어온 'APPLE,BANANA,MANGO'의 값이 \`code\`에 있고, \`p\_code\_id\`로 들어온 'FRIUT'인 코드가 공통 테이블에 있는지 조회한다.

```
SELECT CODE_NM
FROM COMM_CD
WHERE CODE_ID = ANY(string_to_array(p_code_id, v_delimiter))
AND CODE = ANY(string_to_array(p_code, v_delimiter))
```

| 사과 |
| --- |
| 바나나 |
| 망고 |

5.\`array\`함수를 사용하여 데이터를 배열 형태로 그룹화한다. 

```
 array(
        SELECT CODE_NM
        FROM COMM_CD
        WHERE CODE_ID = ANY(string_to_array(p_code_id, v_delimiter))
        AND CODE = ANY(string_to_array(p_code, v_delimiter))
)
```

{사과, 바나나, 망고}

6\. \`array\_agg\`함수를 사용하여 코드명들을 하나의 문자열로 결합하고,\`v\_delimiter\` 즉 ',' 로 구분된 \`APPLE,BANANA,MANGO\`를 배열로 만든 다음 \`array\_to\_string\`함수를 사용해 문자열로 바꾼다.

```
SELECT replace(
                array_to_string(
                    array_agg(
                        array(
                                SELECT CODE_NM
                                FROM COMM_CD
                                WHERE CODE_ID = ANY(string_to_array(p_code_id, v_delimiter))
                                AND CODE = ANY(string_to_array(p_code, v_delimiter))
                        )
                    ), v_delimiter
                ), v_delimiter, ', '
            ) INTO v_result;
```

## 결과

```
SELECT fn_get_comm_code_nm('FRUIT',fruit_nm)
FROM TB_FRUITS;
```

\-> '사과, 바나나, 망고' 

참고로 공통 그룹코드를 조회할 때도 \`WHERE CODE\_ID = ANY(string\_to\_array(p\_code\_id, v\_delimiter))\`

파라미터를 전부 조회하기 때문에 FRUIT외에도 다른 그룹코드의 코드도 조회할 수 있다.

ex) 

```
SELECT fn_get_comm_code_nm('FRUIT,FOOD,FLOWERS',code)
FROM TB_FRUITS;
```
138625 page -> 13969 page 
조인순서를 변경하여 Fetch되는 Page 수 감소(1/10)

```sql
SELECT
        hmwkinfo.*,
        (
                SELECT
                        tg.grp_nm
                FROM
                        tb_grp tg,
                        tb_grp_rel tgr
                WHERE
                        tgr.chdr_grp_no = hmwkinfo.modumNo
                        AND tgr.prnt_grp_no = tg.grp_no
        ) AS subjNm
FROM
        (
                SELECT
                        ORDERBY_NUM() AS rn,
                        hmwk.hmwk_seq,
                        hmwk.grp_no,
                        hmwk.hmwk_ttl,
                        hmwk.stat_cd,
                        COUNT(hmwkpp.hmwk_seq) AS hmwkpp_cnt,
                        hmwk.strt_ymd,
                        hmwk.end_ymd,
                        mbr.mbr_id AS userId,
                        mbr.mbr_nm AS userNm,
                        grp.grp_nm AS modumNm,
                        grp.grp_no AS modumNo,
                        hmwk.opng_cnfg_cd AS openingConfigCode,
                        hmwk.hmwk_repeat AS hmwkRepeat,
                        hmwk.hmwk_repeat_day AS repeatDay,
                        hmwk.modum_op AS modumOp,
                        hmwk.delay_submission AS delaySubmission,
                        hmwk.opng_cnfg AS openingConfig,
                        hmwk.upper_hmwk_seq AS upperHmwkSeq,
                        hmwk.type_code AS typeCode,
                        'Y' AS isHmwkppAtchFile,
                        NVL(category.category_nm, '미분류') AS hmwkCategoryName
                FROM
                        tb_hmwk hmwk
                        LEFT OUTER JOIN tb_hmwkpp hmwkpp ON hmwk.hmwk_seq = hmwkpp.hmwk_seq
                        AND hmwkpp.hmwkpp_stats != 'D'
                        LEFT OUTER JOIN tb_mbr mbr ON hmwk.regr_mbr_no = mbr.mbr_no
                        LEFT OUTER JOIN tb_grp grp ON grp.grp_no = hmwk.modum_no
                        LEFT OUTER JOIN tb_hmwk_category category ON hmwk.hmwk_category = category.category_seq
                WHERE
                        hmwk.grp_no = 'G000461136'
                        AND hmwk.strt_ymd IS NOT NULL
                        AND hmwk.end_ymd IS NOT NULL
                        AND hmwk.stat_cd != 'D'
                        AND hmwk.type_code IS NOT NULL
                        AND hmwk.type_code != ''
                        AND hmwk.type_code != 'RM'
                        AND hmwk.type_code != 'RC'
                        AND hmwk.hmwk_seq NOT IN (
                                SELECT
                                        A.hmwk_seq
                                FROM
                                        TB_HMWK A
                                                USE INDEX (idx_hmwk_grpno)
                WHERE
                        TYPE_CODE = 'R'
                        AND GRP_NO = 'G000461136'
                        AND STAT_CD != 'D'
                        AND NOT EXISTS (
                                SELECT
                                        1
                                FROM
                                        TB_HMWK
                                                USE INDEX (idx_tb_hmwk_05
                        )
                WHERE
                        UPPER_HMWK_SEQ = A.hmwk_seq
                        AND STAT_CD != 'D'
                        AND GRP_NO = 'G000461136'
        ) )
        AND hmwk.hmwk_seq NOT IN (
                SELECT
                        A.hmwk_seq
                FROM
                        TB_HMWK A
                                USE INDEX (idx_hmwk_grpno
        )
WHERE
        TYPE_CODE = 'M'
        AND GRP_NO = 'G000461136'
        AND STAT_CD != 'D'
        AND NOT EXISTS (
                SELECT
                        1
                FROM
                        TB_HMWK
                                USE INDEX (idx_tb_hmwk_05
        )
WHERE
        UPPER_HMWK_SEQ = A.hmwk_seq
        AND STAT_CD != 'D'
        AND GRP_NO = 'G000461136' ) )
        AND (
                hmwk.hmwk_seq IN (
                        SELECT
                                upper_hmwk_seq
                        FROM
                                TB_HMWK
                        WHERE
                                STAT_CD != 'D'
                                AND modum_no IN (
                                        SELECT
                                                auth.GRP_NO
                                        FROM
                                                TB_GRP_AUTH auth,
                                                TB_GRP grp
                                        WHERE
                                                auth.MBR_NO='M098793565'
                                                AND auth.grp_no = grp.grp_no
                                                AND grp.grp_tp_cd = 'L'
                                )
                                AND grp_no = 'G000461136'
                        UNION ALL
                        SELECT
                                upper_hmwk_seq
                        FROM
                                TB_HMWK
                        WHERE
                                modum_no = ''
                                AND GRP_NO = 'G000461136'
                                AND STAT_CD != 'D'
                        UNION ALL
                        SELECT
                                upper_hmwk_seq
                        FROM
                                TB_HMWK
                        WHERE
                                modum_no IS NULL
                                AND GRP_NO = 'G000461136'
                                AND STAT_CD != 'D'
                )
                OR hmwk.type_code = 'C'
        )
GROUP BY
        hmwk.hmwk_seq,
        hmwk.grp_no,
        hmwk.hmwk_ttl,
        hmwk.stat_cd,
        hmwk.strt_ymd,
        hmwk.end_ymd,
        mbr.mbr_id,
        mbr.mbr_nm,
        grp.grp_nm,
        grp.grp_no,
        hmwk.opng_cnfg_cd,
        hmwk.hmwk_repeat,
        hmwk.hmwk_repeat_day
ORDER BY
        hmwk.reg_ymdt DESC
LIMIT
        (0 + 20 + 1) ) hmwkinfo
WHERE
        rn > 0;
```


13969 page
```sql
SELECT
        hmwkinfo.*,
        (
                SELECT
                        tg.grp_nm
                FROM
                        tb_grp tg,
                        tb_grp_rel tgr
                WHERE
                        tgr.chdr_grp_no = hmwkinfo.modumNo
                        AND tgr.prnt_grp_no = tg.grp_no
        ) AS subjNm
FROM
        (
                SELECT
                        ORDERBY_NUM() AS rn,
                        hmwk.hmwk_seq,
                        hmwk.grp_no,
                        hmwk.hmwk_ttl,
                        hmwk.stat_cd,
                        COUNT(hmwkpp.hmwk_seq) AS hmwkpp_cnt,
                        hmwk.strt_ymd,
                        hmwk.end_ymd,
                        mbr.mbr_id AS userId,
                        mbr.mbr_nm AS userNm,
                        grp.grp_nm AS modumNm,
                        grp.grp_no AS modumNo,
                        hmwk.opng_cnfg_cd AS openingConfigCode,
                        hmwk.hmwk_repeat AS hmwkRepeat,
                        hmwk.hmwk_repeat_day AS repeatDay,
                        hmwk.modum_op AS modumOp,
                        hmwk.delay_submission AS delaySubmission,
                        hmwk.opng_cnfg AS openingConfig,
                        hmwk.upper_hmwk_seq AS upperHmwkSeq,
                        hmwk.type_code AS typeCode,
                        'Y' AS isHmwkppAtchFile,
                        NVL(category.category_nm, '미분류') AS hmwkCategoryName
                FROM
                        tb_hmwk hmwk
--                        LEFT OUTER JOIN tb_hmwkpp hmwkpp ON hmwk.hmwk_seq = hmwkpp.hmwk_seq  /* 조인순서 변경 */
--                        AND hmwkpp.hmwkpp_stats != 'D'
                        LEFT OUTER JOIN tb_mbr mbr ON hmwk.regr_mbr_no = mbr.mbr_no
                        LEFT OUTER JOIN tb_grp grp ON grp.grp_no = hmwk.modum_no
                        LEFT OUTER JOIN tb_hmwk_category category ON hmwk.hmwk_category = category.category_seq
                        LEFT OUTER JOIN tb_hmwkpp hmwkpp ON hmwk.hmwk_seq = hmwkpp.hmwk_seq
                        AND hmwkpp.hmwkpp_stats != 'D'
                WHERE
                        hmwk.grp_no = 'G000461136'
                        AND hmwk.strt_ymd IS NOT NULL
                        AND hmwk.end_ymd IS NOT NULL
                        AND hmwk.stat_cd != 'D'
                        AND hmwk.type_code IS NOT NULL
                        AND hmwk.type_code != ''
                        AND hmwk.type_code != 'RM'
                        AND hmwk.type_code != 'RC'
                        AND hmwk.hmwk_seq NOT IN (
                                SELECT
                                        A.hmwk_seq
                                FROM
                                        TB_HMWK A
                                                USE INDEX (idx_hmwk_grpno)
                WHERE
                        TYPE_CODE = 'R'
                        AND GRP_NO = 'G000461136'
                        AND STAT_CD != 'D'
                        AND NOT EXISTS (
                                SELECT
                                        1
                                FROM
                                        TB_HMWK
                                                USE INDEX (idx_tb_hmwk_05
                        )
                WHERE
                        UPPER_HMWK_SEQ = A.hmwk_seq
                        AND STAT_CD != 'D'
                        AND GRP_NO = 'G000461136'
        ) )
        AND hmwk.hmwk_seq NOT IN (
                SELECT
                        A.hmwk_seq
                FROM
                        TB_HMWK A
                                USE INDEX (idx_hmwk_grpno)
WHERE
        TYPE_CODE = 'M'
        AND GRP_NO = 'G000461136'
        AND STAT_CD != 'D'
        AND NOT EXISTS (
                SELECT
                        1
                FROM
                        TB_HMWK
                                USE INDEX (idx_tb_hmwk_05
        )
WHERE
        UPPER_HMWK_SEQ = A.hmwk_seq
        AND STAT_CD != 'D'
        AND GRP_NO = 'G000461136' ) )
        AND (
                hmwk.hmwk_seq IN (
                        SELECT
                                upper_hmwk_seq
                        FROM
                                TB_HMWK
                        WHERE
                                STAT_CD != 'D'
                                AND modum_no IN (
                                        SELECT
                                                auth.GRP_NO
                                        FROM
                                                TB_GRP_AUTH auth,
                                                TB_GRP grp
                                        WHERE
                                                auth.MBR_NO='M098793565'
                                                AND auth.grp_no = grp.grp_no
                                                AND grp.grp_tp_cd = 'L'
                                )
                                AND grp_no = 'G000461136'
                        UNION ALL
                        SELECT
                                upper_hmwk_seq
                        FROM
                                TB_HMWK
                        WHERE
                                modum_no = ''
                                AND GRP_NO = 'G000461136'
                                AND STAT_CD != 'D'
                        UNION ALL
                        SELECT
                                upper_hmwk_seq
                        FROM
                                TB_HMWK
                        WHERE
                                modum_no IS NULL
                                AND GRP_NO = 'G000461136'
                                AND STAT_CD != 'D'
                )
                OR hmwk.type_code = 'C'
        )
GROUP BY
        hmwk.hmwk_seq,
        hmwk.grp_no,
        hmwk.hmwk_ttl,
        hmwk.stat_cd,
        hmwk.strt_ymd,
        hmwk.end_ymd,
        mbr.mbr_id,
        mbr.mbr_nm,
        grp.grp_nm,
        grp.grp_no,
        hmwk.opng_cnfg_cd,
        hmwk.hmwk_repeat,
        hmwk.hmwk_repeat_day
ORDER BY
        hmwk.reg_ymdt DESC
LIMIT
        (0 + 20 + 1)) hmwkinfo
WHERE
        rn > 0;
```