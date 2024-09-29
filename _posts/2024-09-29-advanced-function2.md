---
layout: post
title: (MySQL) 쿼리문 심화 - 윈도우 함수 (2)
date: 2024-09-29
tags: [SQL 심화, MySQL, 데이터 분석]
categories: DataAnalysis
---

지난 포스팅에서 기본 윈도우 함수와 사용 예시 등에 대해 알아보았다. 본 포스팅에서는 주요 윈도우 함수들에 대해 알아보려고 한다.

# 주요 윈도우 함수
## 1. ```ROW_NUMBER()```
각 행에 고유한 번호를 부여하는 함수로, 순서대로 행 번호를 매기는 역할을 한다. **순위와 상관없이 각 행에 순번을 매길 때** 사용한다.  
**예시)** 판매 테이블에서 각 카테고리 내에서 판매가 발생한 순서대로 행 번호를 매기고 싶을 때

```SQL
SELECT 
    category, 
    sales_date, 
    sales_amount, 
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales_date) AS row_num
FROM sales
ORDER BY category, sales_date;
```
- ```PARTITION BY category``` 로 카테고리 별로 데이터를 나누고, 각 카테고리 내에서 ```ORDER BY sales_date``` 로 정렬하여 판매 발생 순서대로 번호를 매긴다.

**결과 예시)**

|category|sales_date|sales_amount|row_num|
|:---|:---|:---|:---|
|A|2024-01-01|1000|1|
|A|2024-01-02|1500|2|
|A|2024-01-03|1200|3|
|B|2024-01-01|2000|1|
|B|2024-01-02|1800|2|

---
## 2. ```RANK()```
동일한 값이 있을 때는 같은 순위를 부여하고 그다음 순위는 건너뛰는 방식으로 순위를 매긴다.

**예시)** 각 카테고리별로 매출 금액 순위를 매기는 경우

```SQL
SELECT 
    category, 
    product_id, 
    sales_amount, 
    RANK() OVER (PARTITION BY category ORDER BY sales_amount DESC) AS sales_rank
FROM sales
ORDER BY category, sales_rank;
```

- ```PARTITION BY category``` 로 카테고리 별로 데이터를 나누고, 매출 금액을 기준으로 내림차순 정렬해서 순위를 매긴다.

**결과 예시)**

|category|product_id|sales_amount|sales_rank|
|:---|:---|:---|:---|
|A|101|2000|1|
|A|102|2000|1|
|A|103|1500|3|
|B|201|1800|1|
|B|202|1600|2|

---
## 3. ```DENSE_RANK()```
```RANK()``` 와 유사하지만 동일 값이 있을 때 순위 간격을 건너뛰지 않고 연속적으로 순위를 부여한다.

**예시)** 동일 매출이 있을 때에도 연속된 순위 유지를 하고 싶을 때

```SQL
SELECT 
    category, 
    product_id, 
    sales_amount, 
    DENSE_RANK() OVER (PARTITION BY category ORDER BY sales_amount DESC) AS dense_sales_rank
FROM sales
ORDER BY category, dense_sales_rank;
```

- ```PARTITION BY category``` 로 카테고리 별로 데이터를 나누고, 매출 금액에 따라 연속된 순위를 부여한다.

**결과 예시)**
|category|product_id|sales_amount|sales_rank|
|:---|:---|:---|:---|
|A|101|2000|1|
|A|102|2000|1|
|A|103|1500|2|
|B|201|1800|1|
|B|202|1600|2|

---

## 4. ```LEAD()```
행 간의 비교가 필요할 때 사용하는 함수로, 현재 행의 다음 행에 있는 값(즉, **미래의 값**)을 참조할 때 사용된다.

**예시)** 특정 제품 매출이 다음날 매출과 얼마나 차이가 나는지 비교하는 경우

```SQL
SELECT
    product_id,
    sales_date,
    sales_amount,
    LEAD(sales_amount, 1) OVER (PARTITION BY product_id ORDER BY sales_date) AS next_day_sales,
    sales_amount - LEAD(sales_amount, 1) OVER (PARTITION BY product_id ORDER BY sales_date) AS sales_diff
FROM sales
ORDER BY product_id, sales_date; 
```
- ```LEAD(sales_amount, 1)``` 함수는 현재 행의 다음 행에 있는 매출 값을 가져오는 역할을 한다.

**결과 예시)** 

|product_id|sales_date|sales_amount|next_day_sales|sales_difference|
|:---|:---|:---|:---|:---|
|101|2024-01-01|1000|1500|-500|
|101|2024-01-02|1500|1200|500|
|101|2024-01-03|1200|NULL|NULL|

## 5. ```LAG()```
마찬가지로 행간 비교 함수로 현재 행의 이전 행에 있는 값(**과거의 값**)을 참조할 때 사용한다.

**예시)** 각 제품의 매출이 전날과 비교했을 때 얼마나 변동했는지 확인하는 경우

```SQL 
SELECT 
    product_id,
    sales_date,
    sales_amount,
    LAG(sales_amount, 1) OVER (PARTITION BY product_id ORDER BY sales_date) AS prev_day_sales,
    sales_amount - LAG(sales_amount, 1) OVER (PARTITION BY product_id ORDER BY sales_date) AS sales_diff
FROM sales
ORDER BY product_id, sales_date; 
```

```LAG(sales_amount, 1)``` 함수는 현재 행의 이전 행에 있는 매출 값을 가져온다.

**결과예시)** 
|product_id|sales_date|sales_amount|next_day_sales|sales_difference|
|:---|:---|:---|:---|:---|
|101|2024-01-01|1000|NULL|NULL|
|101|2024-01-02|1500|1000|500|
|101|2024-01-03|1200|1500|-300|

## 그 외
MAX, MIN, AVG, SUM, COUNT 함수들은 각각 선별된 그룹 별로 최대값, 최소값, 평균값, 합계, 집계 를 계산하는 함수이며, GROUP BY 와 함께 사용하는 사례도 많으니 잘 기억해둬야 한다.

# 윈도우 함수 사용 사례
다음으로 이와같은 윈도우 함수들을 적극적으로 사용하는 사례들을 몇가지 보려고 한다.

## 1. top k 유저 접근 수 집계
```Users``` 테이블에는 일별 유저 접근 수, 일자, 성별 등의 유저 정보가 담겨있다고 하자. 이 때, 일 별 접근 수로 top 3 유저 아이디를 가져오는 쿼리문을 짜보면 아래와 같이 짤 수 있다.

```SQL
WITH RankedUsers AS (
  SELECT 
    u.u_id, 
    u.day, 
    RANK() OVER (PARTITION BY u.day ORDER BY u.access_cnt DESC) AS ranking
  FROM Users u
)
SELECT 
  r.u_id, 
  r.day, 
  r.ranking
FROM RankedUsers r
WHERE r.ranking <= 3
ORDER BY r.day, r.ranking;
```
**결과예시)**

|u_id|day|ranking|
|:---|:---|:---|
|101|2024-01-01|1|
|102|2024-01-01|2|
|103|2024-01-01|3|
|104|2024-01-02|1|
|105|2024-01-02|2|
|106|2024-01-02|3|

