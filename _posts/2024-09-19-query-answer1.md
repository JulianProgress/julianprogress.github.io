---
layout: post
title:  (쿼리 테스트) 대여 가능한 자동차는?
date:   2024-09-19
description: SQL, Query Test, CASE, GROUP BY
---


# 자동차 대여 기록에서 대여중 / 대여 가능 여부 구분하기
[문제링크](https://school.programmers.co.kr/learn/courses/30/lessons/157340)

해당 문제는 프로그래머스 Lv.3. 문제이다.

## 문제 설명
자동차 대여 회사의 자동차 대여 기록 정보를 담은 'CAR_RENTAL_COMPANY_RENTAL_HISTORY' 테이블을 활용한다. 문제는 'CAR_RENTAL_COMPANY_RENTAL_HISTORY' 테이블에서 2022년 10월 16일에 대여 중인 자동차인 경우 '대여중' 으로 표시하고, 대여 중이지 않은 자동차의 경우 '대여 가능'을 표시하는 컬럼 (AVAILABILITY) 를 추가해서 자동차 ID 와 함께 제시하는 문제이다.  
언뜻 생각해보면 쉽다고 느껴지지만, 사실 고려해야될 사항이 있다.

이 문제는 CASE와 GROUP BY 를 사용해 차량 ID 별로 해당일에 대여중인지 확인을 해야 한다.

### 1. CASE 문
우선 차량이 해당 날짜에 대여중인지를 알기 위해서는 START_DATE와 END_DATE를 활용해야 한다.  
그래서 만약 START_DATE와 END_DATE 사이에 '2022-10-16' 이 위치하면 대여중, 그렇지 않으면 대여 가능으로 표기하면 된다.

```SQL
CASE
    WHEN '2022-10-16' BETWEEN START_DATE AND END_DATE THEN '대여중'
    ELSE '대여 가능'
END
```

### 2. GROUP BY
'CAR_RENTAL_COMPANY_RENTAL_HISTORY' 테이블은 모든 차량 대여 기록이 담겨 있기 때문에 특정 차량이 여러 번 대여되었을 가능성도 있다. 그렇기 때문에 차량 별로 대여중/대여 가능을 표기해야 하므로 GROUP BY 절을 사용해 CAR_ID 별로 정보를 묶어준다.

```SQL
SELECT DISTINCT CAR_ID,
    CASE
        WHEN '2022-10-16' BETWEEN START_DATE AND END_DATE THEN '대여중'
        ELSE '대여 가능'
    END AS AVAILABILITY
FROM CAR_RENTAL_COMPANY_RENTAL_HISTORY
GROUP BY CAR_ID
ORDER BY CAR_ID DESC;
```
---
### One more thing..
하지만 여기서 끝이 아니다. 이렇게만 끝내면 올바르게 표기된것으로 착각할만 하다. 문제는 '한 차량이 여러번 대여되었을 가능성' 이다. 그렇기 때문에 대여 기록 중 특정 차량이 한번이라도 2022년 10월 16일에 대여된 기록이 있다면, '대여중' 으로 표기 해야한다는 점이다.  
이를 해결하기 위해서는 ```WHEN '2022-10-16' BETWEEN START_DATE AND END_DATE THEN '대여중'``` 부분에 약간의 수정이 필요하다.
```SQL
CASE
    WHEN MAX(CASE WHEN '2022-10-16' BETWEEN START_DATE AND END_DATE THEN 1 ELSE 0 END) = 1
    THEN '대여중'
    ELSE '대여 가능'
END
```
이렇게 case 문을 바꾸게 되면 ```MAX(...)``` 내부 절에서 한번이라도 2022-10-16 이 start date 와 end date 사이에 있었을 때 1을 반환하게 된다. 다시말해 '대여 가능' 차량은 2022-10-16 이 START_DATE, END_DATE 사이에 한번도 위치해 있지 않았으므로, 0만 반환하게 되며, ```MAX(0...)``` 은 0이므로 '대여 가능' 으로 분류된다.


