---
title: [쿼리테스트] 자동차 대여기록 별 대여 금액 구하기
categories:
- SQL
- MySQL
- Query Test
- DATEDIFF
- REGEXP_REPLACE
feature_image: "https://picsum.photos/2560/600?image=872"
---

# 자동차 대여 기록 별 대여 금액 구하기
[문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/151141)

프로그래머스 문제 중 Lv.4 에 해당하는 문제이며, 정답율은 **48%** 이다.

## 문제 설명
자동차 대여 회사의 자동차 정보를 담은 CAR_RENTAL_COMPANY_CAR 테이블과 자동차 대여 기록 정보를 담은 CAR_RENTAL_COMPANY_RENTAL_HISTORY 테이블, 그리고 자동차 종류 별 대여 기간 및 종류 별 할인 정책 정보를 담은 CAR_RENTAL_COMPANY_DISCOUNT_PLAN 테이블을 이용한다. 각 테이블의 컬럼 정보는 위 문제링크에서 확인할 수 있다.

문제는 이 세가지 테이블에서 자동차 종류가 '트럭' 인 자동차의 대여 기록에 대해서 대여 기록 별로 대여 금액을 구하고 대여 기록 ID, 대여 금액 리스트를 출력하는 것이 목표이다.  

## 문제 풀이
### 1. DISCOUNT 정보 가져오기
우선 문제를 보았을 때, 직관적으로 discount 정보 테이블을 따로 가져와서 이를 활용하는 것이 편할 것으로 생각이 되었다. 그래서 WITH 절을 사용해 임시적으로 트럭과 관련된 discount 정보를 우선 불러오기로 한다.

```SQL
WITH CAR_RENTAL_DISC AS (
    SELECT *
    FROM CAR_RENTAL_COMPANY_DISCOUNT_PLAN
    WHERE CAR_TYPE = '트럭'
)
...
```
이렇게 하면 트럭과 관련된 discount 정책이 반환되는데, 우리가 사용할 것은 'duration_type' 과 'discount_rate' 을 사용해야 한다. 해당 정보는 모두 CAR_RENTAL_DISC 테이블에 임시로 저장이 된다.

### 2. history 테이블에서 차량 대여 기간 계산
history 테이블에는 START_DATE 와 END_DATE 가 있으므로, MySQL 의 DATEDIFF 함수를 사용해 쉽게 대여 일수를 가져올 수 있다. 단, DATEDIFF 를 사용하여 도출한 수에 +1일을 해줘야 정확한 대여 일수가 된다 (만약 하루를 빌린다면 start_date = end_date 이므로 diff 값은 0이 되며, 다른 일수도 마찬가지로 diff 값은 실제 대여 일수보다 하루 적게 계산된다는 점을 파악해야 함!)

```SQL
DATEDIFF(end_date, start_date)
```

### 3. case 문을 사용, duration type 별로 나누어 fee 를 연산
다음으로 discount 율을 구하기 위해 case 문을 사용해 대여 기간을 일정 기간 단위로 나누어서 볼 것이다. 문제풀이 1번 부분에서 discount 정보를 가져왔는데 여기에는 DURATION_TYPE 으로 '7일 이상', '30일 이상', '90일 이상' 등으로 나누어져 있다. 이를 CASE 문으로 분류해보면 다음과 같다:

```SQL
SELECT DISTINCT history_id,
CASE 
    WHEN DATEDIFF(h.end_date, h.start_date) >= 89 
        THEN ...
    WHEN DATEDIFF(h.end_date, h.start_date) >= 29 AND DATEDIFF(h.end_date, h.start_date) < 89 
        THEN ...
    WHEN DATEDIFF(h.end_date, h.start_date) >= 7 AND DATEDIFF(h.end_date, h.start_date) < 29 
        THEN ...
    ELSE ...
END FEE
...
```

### 4. HISTORY 정보와 차량 정보 JOIN
CASE 문을 완성하기 전에, History 테이블과 차량 정보 테이블을 조인해서 차량 타입이 트럭인 경우만을 먼저 필터링 해야한다. 
```SQL
SELECT *
FROM CAR_RENTAL_COMPANY_RENTAL_HISTORY h
JOIN CAR_RENTAL_COMPANY_CAR c ON h.CAR_ID = c.CAR_ID
WHERE c.CAR_TYPE = '트럭'
ORDER BY 2 DESC, 1 DESC;
```
이렇게 하면 CAR_TYPE 이 트럭인 정보만 불러올 수 있다.

### 5. CASE 문 마무리
마지막으로 CASE 문에서 각 구간 별로 discount 된 값을 구해주면 된다.  
우선 차량 정보 테이블에는 DAILY_FEE 컬럼으로 하루 이용료만 존재하므로, 각 history 별로 기간 * Daily fee 를 해주어야 한다.

```SQL
c.DAILY_FEE * (DATEDIFF(h.end_date, h.start_date)+1)
```
위의 식으로 계산된 값이 discount 가 없다했을 때의 차량 대여료에 해당한다. 이제 마지막으로 각 케이스에 맞게 discount 만 계산해주면 된다. 먼저 DISCOUNT_RATE 컬럼은 5%, 7% 등과 같이 % 가 붙은 string 타입이다. 여기서 우리는 숫자만을 추출해야 하기 때문에 SQL 정규식 연산을 활용해서 숫자만을 추출할 수 있다.

```SQL
REGEXP_REPLACE(DISCOUNT_RATE, '[^0-9]+', '') AS FLOAT
```
이처럼 MySQL 에서는 REGEXP 함수를 사용하면 문자열에서 숫자만 추출할 수 있다. 위의 정규식에서 ```[^0-9]+```  부분은 숫자가 아닌 모든 문자만 선택하는 정규식이고, 함수의 세번째 인자인 ```''``` 가 나타내는 바는 이러한 숫자가 아닌 문자들을 모두 빈 문자열로 대체하겠다는 의미이다. 결과적으로 위 정규식 함수를 통해 숫자를 추출할 수 있고 우리는 이 숫자를 다시 형변환하여 사용할 수 있다. 결과적으로 DISCOUNT 율을 다음과 같이 연산할 수 있다.
```SQL
CAST(REGEXP_REPLACE(DISCOUNT_RATE, '[^0-9]+', '') AS FLOAT) * 0.01
```
이제 저 DISCOUNT_RATE 의 경우 위에서 WITH 절로 가지고 있던 DISCOUNT 테이블에서 선택해주고 이를 활용해서 ```원본 차량 대여료 * (1 - DISCOUNT_RATE)``` 식으로 discount 된 최종 대여료를 반환할 수 있다. 이를 CASE 에 적용하면 최종적으로 아래와 같은 CASE 문을 완성할 수 있다.

```SQL
(앞부분 생략)...
CASE 
    WHEN DATEDIFF(h.end_date, h.start_date) >= 89 
        THEN c.DAILY_FEE * (DATEDIFF(h.end_date, h.start_date)+1) * (1 - (SELECT CAST(REGEXP_REPLACE(DISCOUNT_RATE, '[^0-9]+', '') AS FLOAT) * 0.01 AS numbers FROM CAR_RENTAL_DISC WHERE DURATION_TYPE = '90일 이상'))
    WHEN DATEDIFF(h.end_date, h.start_date) >= 29 AND DATEDIFF(h.end_date, h.start_date) < 89 
        THEN c.DAILY_FEE * (DATEDIFF(h.end_date, h.start_date)+1) * (1 - (SELECT CAST(REGEXP_REPLACE(DISCOUNT_RATE, '[^0-9]+', '') AS FLOAT) * 0.01 AS numbers FROM CAR_RENTAL_DISC WHERE DURATION_TYPE = '30일 이상'))
    WHEN DATEDIFF(h.end_date, h.start_date) >= 7 AND DATEDIFF(h.end_date, h.start_date) < 29 
        THEN c.DAILY_FEE * (DATEDIFF(h.end_date, h.start_date)+1) * (1 - (SELECT CAST(REGEXP_REPLACE(DISCOUNT_RATE, '[^0-9]+', '') AS FLOAT) * 0.01 AS numbers FROM CAR_RENTAL_DISC WHERE DURATION_TYPE = '7일 이상'))
    ELSE c.DAILY_FEE * (DATEDIFF(h.end_date, h.start_date)+1)
END FEE
...(뒷부분 생략)
```
복잡해 보이지만, 원리대로 차근차근히 풀어보면 쉽게 풀어낼 수 있다. 

그래서 얻어낸 전체 쿼리문은 다음과 같다:
```SQL
WITH CAR_RENTAL_DISC AS (
    SELECT *
    FROM CAR_RENTAL_COMPANY_DISCOUNT_PLAN
    WHERE CAR_TYPE = '트럭'
)
SELECT DISTINCT history_id,
CASE 
    WHEN DATEDIFF(h.end_date, h.start_date) >= 89 
        THEN c.DAILY_FEE * (DATEDIFF(h.end_date, h.start_date)+1) * (1 - (SELECT CAST(REGEXP_REPLACE(DISCOUNT_RATE, '[^0-9]+', '') AS FLOAT) * 0.01 AS numbers FROM CAR_RENTAL_DISC WHERE DURATION_TYPE = '90일 이상'))
    WHEN DATEDIFF(h.end_date, h.start_date) >= 29 AND DATEDIFF(h.end_date, h.start_date) < 89 
        THEN c.DAILY_FEE * (DATEDIFF(h.end_date, h.start_date)+1) * (1 - (SELECT CAST(REGEXP_REPLACE(DISCOUNT_RATE, '[^0-9]+', '') AS FLOAT) * 0.01 AS numbers FROM CAR_RENTAL_DISC WHERE DURATION_TYPE = '30일 이상'))
    WHEN DATEDIFF(h.end_date, h.start_date) >= 7 AND DATEDIFF(h.end_date, h.start_date) < 29 
        THEN c.DAILY_FEE * (DATEDIFF(h.end_date, h.start_date)+1) * (1 - (SELECT CAST(REGEXP_REPLACE(DISCOUNT_RATE, '[^0-9]+', '') AS FLOAT) * 0.01 AS numbers FROM CAR_RENTAL_DISC WHERE DURATION_TYPE = '7일 이상'))
    ELSE c.DAILY_FEE * (DATEDIFF(h.end_date, h.start_date)+1)
END FEE
FROM CAR_RENTAL_COMPANY_RENTAL_HISTORY h
JOIN CAR_RENTAL_COMPANY_CAR c ON h.CAR_ID = c.CAR_ID
WHERE c.CAR_TYPE = '트럭'
ORDER BY 2 DESC, 1 DESC;
```

끝.  
정리하는 것도 힘들다..