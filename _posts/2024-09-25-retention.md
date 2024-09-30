---
layout: post
title:  "[데이터 분석 SQL] 리텐션 분석 SQL 로 해보기"
date:   2024-09-25
tags: [SQL, Retention Analysis]
categories: DataAnalysis
---

# 리텐션 분석이란?
리텐션 분석 (Retention Analysis)은 특정 기간 동안 고객 또는 사용자가 서비스를 유지하는지 여부를 추적하는 분석기법이다. 이 분석기법은 사용자 유지율, 이탈율, 코호트(Cohort: 일정한 특성을 공유하는 사람들의 집단) 별 유지 등을 분석하여 비즈니스 성과를 측정하고, 분석 결과를 토대로 이탈을 최소화하면서 이용자의 참여를 극대화하는 전략 수립에 도움을 줄 수 있다.  
리텐션 분석은 일반적으로 아래와 같은 질문에 답을 하는 기법이다:
- 특정 시점에 가입한 사용자 중 얼마나 많은 사용자가 여전히 서비스를 이용중인가?
- 제품/서비스의 사용자 유지율이 어떤가?
- 특정 이벤트 후 사용자 행동이 어떻게 변화하는가?

리텐션 분석에 자주 사용되는 지표로는 Day 1 Retention, Day 7 Retention, Day 30 Retention 등과 같이 시간에 따른 유지율과 관련이 깊다. 이를 통해 사용자가 서비스에 얼마나 머무는지를 평가할 수 있다.

---
# SQL 분석 예시
## 1. Day 1 Retention 분석
특정 날짜에 신규 가입한 사용자의 Day 1 Retention 비율을 계산하려고 한다. 예시 table 로 user_events 에는 아래와 같은 정보가 담겨있다고 하자:

|user_id|event_name|event_timestamp|
|:---|:---|:---|
|유저 고유 id|유저 행동 종류 (first_visit, sign_in, click 등)|이벤트 발생 datetime|

이 때, Day 1 Retention 분석을 위한 쿼리를 만들어보면 다음과 같다.

```SQL
WITH retention_data AS (
    SELECT
        user_id,
        MIN(DATE(event_timestamp)) OVER (PARTITION BY user_id) AS first_visit_date,
        DATE(event_timestamp) AS event_date
    FROM user_events
    WHERE event_name = 'first_visit' OR event_name IN (
        'sign_in', 'page_view', 'scroll'
    )
)
SELECT first_visit_date AS signup_date,
    COUNT(DISTINCT user_id) AS total_signups,
    COUNT(DISTINCT CASE
        WHEN event_date = DATE_ADD(first_visit_date, INTERVAL 1 DAY) THEN user_id END
    ) AS day_1_retained,
    (
        COUNT(DISTINCT CASE
        WHEN event_date = DATE_ADD(first_visit_date, INTERVAL 1 DAY) THEN user_id END
        ) / COUNT(DISTINCT user_id) * 100 AS day_1_retention_rate
    )
FROM retention_data
GROUP BY first_visit_date;
```
- CTE (Common Table Expression)

  - ```WITH retention_date AS ()```: 윈도우 함수 ```MIN() OVER  (PARTITION BY ...)``` 를 사용해 사용자 별 첫 방문 날짜를 계산 (```event_date``` 도 함께 가져옴)
  - 필요한 이벤트 필터링을 위해 WHERE 절을 통해 first_visit 과 주요 리텐션 분석에 사용할 이벤트 종류를 추출함
- 메인 쿼리
  - ```first_visit_date``` 를 기준으로 사용자 별로 Day 1 Retention을 유지한 사용자를 계산
  - ```signup_date``` 별로 Day 1 Retention 비율을 계산

결과적으로 각 가입 날짜 별로 총 첫 방문자 수 및 Day 1 Retention 유지한 사용자 수(```day_1_retained```) 그리고 리텐션 비율(```day_1_retention_rate```) 을 구할 수 있다.

이렇게 리텐션 비율을 first_visit 을 기준으로 몇일 혹은 몇달 후인지를 비교하는 쿼리는 구조화를 할 수 있다:

```SQL
COUNT(DISTINCT CASE
    WHEN event_date = DATE_ADD(first_visit_date, INTERVAL XX) THEN user_id
END)
```


## 2. 1 Week Retention 분석
일주일 후 여전히 활동 중인 사용자 비율을 확인하는 쿼리는 다음과 같다.

```SQL
SELECT 
    DATE(first_visit_date) AS signup_date,
    COUNT(DISTINCT user_id) AS total_signups,
    COUNT(DISTINCT CASE
        WHEN event_date = DATE_ADD(signup_date, INTERVAL 7 DAY) THEN user_id END
    ) AS week_retained,
    (
        COUNT(DISTINCT CASE
            WHEN event_date = DATE_ADD(signup_date, INTERVAL 7 DAY) THEN user_id END
        ) / COUNT(DISTINCT user_id) * 100 AS week_retention_rate
    )
FROM users
WHERE signup_date BETWEEN '2024-09-01' AND '2024-09-30'
GROUP BY signup_date;
```
Day 1 리텐션 비율과 비슷하지만 이번에는 ```INTERVAL``` 을 ```7 DAY``` 로 두어 주별 리텐션 분석이 가능하다.

## 3. 월별 코호트 분석
아래 쿼리는 월별 코호트를 생성해 첫달 가입한 사용자가 1, 2, 3개월 후에도 얼마나 남아있는지를 분석한다:
```SQL
SELECT
    DATE_FORMAT(signup_date, '%Y-%m') AS cohort_month,
    COUNT(DISTINCT user_id) AS total_signups,
    COUNT(DISTINCT CASE WHEN DATE(last_active_date) BETWEEN signup_date AND DATE_ADD(signup_date, INTERVAL 30 DAY) THEN user_id END) AS month_1_retained,
    COUNT(DISTINCT CASE WHEN DATE(last_active_date) BETWEEN DATE_ADD(signup_date, INTERVAL 31 DAY) AND DATE_ADD(signup_date, INTERVAL 60 DAY) THEN user_id END) AS month_2_retained,
    COUNT(DISTINCT CASE WHEN DATE(last_active_date) BETWEEN DATE_ADD(signup_date, INTERVAL 61 DAY) AND DATE_ADD(signup_date, INTERVAL 90 DAY) THEN user_id END) AS month_3_retained
FROM users
WHERE signup_date BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY cohort_month;
```
- ```DATE_FORMAT()``` : 각 사용자의 가입일을 월 단위로 묶어서 각 코호트가 속하는 달을 정의한다.
- ```COUNT(DISTINCT user_id)```: 각 코호트의 첫 가입자 수를 계산한다.
- ```COUNT(DISTINCT CASE WHEN ... THEN user_id END)```: 첫 방문 후 특정 기간 (예: 1개월, 2개월, ...) 내에 다시 서비스를 사용한 사용자 수를 계산한다. 


### 코호트 분석 쿼리 기본구조
코호트 분석 쿼리의 기본구조는 아래와 같이 정의할 수 있다:

```SQL
WITH cohort_data AS (
    -- 각 사용자별 코호트 형성
    SELECT
        user_id,
        MIN(signup_date) AS signup_date,
        DATE_FORMAT(MIN(signup_date), '%Y-%m') AS cohort_month
    FROM users
    GROUP BY user_id
)
SELECT
    cohort_month,
    COUNT(DISTINCT user_id) AS total_signups,
    COUNT(DISTINCT CASE
        WHEN event_date = DATE_ADD(signup_date, INTERVAL 1 DAY) THEN user_id
    END) AS day_1_retained,
    COUNT(DISTINCT CASE
        WHEN event_date BETWEEN DATE_ADD(signup_date, INTERVAL 1 MONTH) AND DATE_ADD(signup_date, INTERVAL 2 MONTH) THEN user_id
    END) AS month_1_retained,
    COUNT(DISTINCT CASE
        WHEN event_date BETWEEN DATE_ADD(signup_date, INTERVAL 2 MONTH) AND DATE_ADD(signup_date, INTERVAL 3 MONTH) THEN user_id
    END) AS month_2_retained
FROM cohort_data cd
LEFT JOIN events e ON cd.user_id = e.user_id
GROUP BY cohort_month;
```
- ```WITH cohort_data```: 코호트 분석을 위한 기본 데이터를 생성한다. 여기서는 사용자별로 최초 가입일을 기준으로 코호트를 형성하고 있으나, 분석 기준에 따라 달라질 수 있다.
- COUNT(DISTINCT CASE ... END): 주어진 기간 동안 서비스에 재참여한 사용자를 계산한다. 각 조건에 따라 특정 기간 후에도 사용자 활동이 있었는가를 함께 확인한다.
- ```GROUP BY cohort_month```: 코호트 별로 리텐션 데이터를 집계한다.

## 4. 코호트 별 리텐션 비율 계산
코호트별로 1개월 차 리텐션 비율을 계산하는 쿼리는 아래와 같다:

```SQL
SELECT
    DATE_FORMAT(signup_date, '%Y-%m') AS cohort_month,
    COUNT(DISTINCT user_id) AS total_signups,
    (
        COUNT(DISTINCT CASE WHEN DATE(last_active_date) BETWEEN signup_date AND DATE_ADD(signup_date, INTERVAL 30 DAY) THEN user_id END) / COUNT(DISTINCT user_id)
    ) * 100 AS month_1_retention_rate
FROM users
WHERE signup_date BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY cohort_month;
```
- ```COUNT(DISTINCT CASE ...)```: 


---
# 결론
이번 포스팅에서는 리텐션 분석을 위한 쿼리문에 대해 몇가지 예제와 함께 그 구조를 살펴보았다. 