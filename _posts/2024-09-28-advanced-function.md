---
layout: post
title: (MySQL) 쿼리문 심화 (윈도우 함수)
date:   2024-09-28
description: SQL 심화, MySQL, 데이터 분석
---
<p class="intro"><span class="dropcap">S</span>QL 쿼리문 사용 시 자주 사용되는 형태 중 하나는 윈도우 함수이다. 데이터의 특정 범위에 대해 여러가지 연산을 수행하고자 할 때 윈도우 함수를 사용할 수 있는데, 특히 누적합, 순위 계산, 이동 평균 등을 계산할 때 사용될 수 있다. 이번 포스트에서는 윈도우 함수의 개념부터 리텐션 분석을 예시로 윈도우 함수를 어떻게 이용하는지 정리한다.</p>

## 기본 윈도우 함수 구조
윈도우 함수는 기본적으로 ```OVER ()``` 절을 활용한다. 아래는 기본적인 윈도우 함수의 구조이다.

{%- highlight SQL -%}
SELECT 
    column1, 
    SUM(column2) OVER (PARTITION BY column3 ORDER BY column4) AS sum_col2
FROM (table);
{%- endhighlight -%}

```OVER ()``` 는 윈도우 함수가 작용하는 범위를 정의하는 절이다. 상황에 따라 ```PARTITION BY```와 ```ORDER BY``` 를 함께 적절하게 사용할 수 있어야 한다.

## PARTITION BY
```PARTITION BY``` 는 윈도우 함수와 함께 사용되어 데이터를 특정 범주로 **분할(파티션)**하고, 각 행마다 해당 범주의 집계 결과를 반환한다. 중요한 점은, ```PARTITION BY``` 는 데이터를 그룹화하지만, 그 그룹을 유지하면서 행마다 집계 결과를 표시한다는 것이다. 즉, 행마다 집계 결과가 반복적으로 표시되며, 그룹별로 요약된 데이터가 아닌 각 행의 데이터와 함께 집계된 값을 반환하게 된다.  
**용도**: 데이터 분석 시 특정 범주별로 별도의 집계 연산 시 유용하다. 예를 들어, 제품 카테고리별로 매출 합계를 계산하거나 부서별 평균 급여 계산 시 사용한다.

예시) 부서 별 평균 급여 계산

{%- highlight SQL -%}
SELECT 
    department,
    employee_name,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS avg_department_salary
FROM employees;
{%- endhighlight -%}

### ```PARTITION BY``` vs ```GROUP BY```
```PARTITION BY``` 는 윈도우 함수에 사용되는 절이며 ```GROUP BY``` 와 비슷하지만, 각 그룹에 대한 결과를 행마다 유지한다는 점에서 차이를 보인다. 즉, 윈도우 함수를 사용하면 결과가 각 행마다 출력되며 **그 행이 속하는 집합에 해당하는 전체 집계 정보가 중복되어 표시**된다.  
반면 ```GROUP BY``` 를 사용하면 각 그룹 별로 계산해서 **한 행에 하나의 그룹**과 그 집계 결과가 출력된다.

먼저 윈도우 함수(```PARTITION BY```) 를 사용한 경우는 다음과 같다.  

{%- highlight SQL -%}
SELECT 
    category,
    amount,
    price,
    SUM(amount * price) OVER (PARTITION BY category) AS total_sales_category
FROM sales;
{%- endhighlight -%}

결과적으로 윈도우 함수를 사용했을 때, 각 행마다 ```category``` 가 동일한 제품의 매출 합계가 **모든 행에 표시**된다. 따라서 위와 같은 예시에서는 같은 카테고리에 혹한 각 행마다 해당 카테고리의 매출 총액이 반복되어 출력된다. 이 방법은 행단위로 추가적인 매출 계산이 필요한 경우에 유용할 수 있다.  

|category|amount|price|total_sales_category|
|:---|:---|:---|:---|
|A|10|50|5000|
|A|20|50|5000|
|A|30|100|5000|
|B|15|30|4500|
|B|30|30|4500|


```GROUP BY``` 는 데이터를 그룹화해서 각 그룹 별로 하나의 결과 행만을 반환한다. 즉, 각 그룹에 대해 집계 함수 (```SUM```, ```COUNT```, ```AVG``` 등) 를 적용 후, 그룹별로 하나의 결과만을 생성한다. 이 방식은 그룹화 된 데이터를 요약해서 보여줄 때 유용하다.

{%- highlight SQL -%}
SELECT 
    category,
    SUM(amount * price) AS total_sales
FROM sales
GROUP BY category;
{%- endhighlight -%}

위 쿼리는 같은 category 값을 가진 행들을 그룹화한 후, 각 그룹에 대해 ```SUM(amount * price)``` 를 계산하고 그 결과를 하나의 행으로 반환하게 된다. 

|category|total_sales|
|:---|:---|
|A|5000|
|B|4500|

## ORDER BY
```ORDER BY``` 는 데이터를 정렬한 후 누적합, 순위, 이동 평균 등 계산 시 사용된다. 윈도우 함수가 각 행을 처리할 때 어떤 순서로 적용될지를 정의한다.

- **용도**: 누적합, 순위, 이동 평균 등과 같이 순서정보가 포함되는 연산 시 주로 사용된다.

예시) 매일의 매출에 대해 날짜순으로 누적 매출 합계 구하기

{%- highlight SQL -%}
SELECT 
    sales_date,
    sales_amount,
    SUM(sales_amount) OVER (ORDER BY sales_date) AS cum_sales
FROM sales
GROUP BY sales_date; # 또는 DATE(sales_date) 사용
{%- endhighlight -%}

이 쿼리는 매일의 매출(sales_amount) 에 대해 날짜순으로 누적 매출(cum_sales) 를 계산한다. 그 결과는 아래 예시와 같이 나타나며, 날짜별로 매출을 누적한 순서를 정의한다.

|sales_date|sales_amount|cum_sales|
|:---|:---|:---|
|2024-01-01|1000|1000|
|2024-01-02|1500|2500|
|2024-01-03|2000|4500|

## PARTITION BY & ORDER BY
```PARTITION BY``` 와 ```ORDER BY``` 를 함께 사용하면 데이터를 그룹으로 나누고 각 그룹 내에서 지정한 순서대로 윈도우 함수가 적용된다. 즉, **그룹 별로 정렬된 데이터를 기반으로 누적합, 순위, 이동 평균 등을 도출**할 수 있다. 
- **용도**: 특정 그룹 내에서 순차적으로 데이터를 분석할 때 사용된다.

예시) 부서 별 누적 매출 구하기

{%- highlight SQL -%}
SELECT
    department,
    sales_date,
    sales_amount,
    SUM(sales_amount) OVER (PARTITION BY department ORDER BY sales_date) AS cum_sales_by_department
FROM department_sales
ORDER BY 1, 2; 
{%- endhighlight -%}

위 쿼리는 부서 별로 데이터를 나눈 뒤, 부서 내에서 날짜별로 매출을 누적하여 계산한다. ```PARTITION BY department``` 는 부서별로 데이터를 나누고 ```ORDER BY sales_date``` 는 각 부서 내에서 날짜 순 매출을 누적하는 역할을 한다.

|department|sales_date|sales_amount|cum_sales_by_department|
|:---|:---|:---|:---|
|HR|2024-01-01|1000|1000|
|HR|2024-01-02|1500|2500|
|Sales|2024-01-01|1000|1000|
|Sales|2024-01-02|1200|2200|