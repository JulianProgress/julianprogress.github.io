---
title: (SQL) IN / EXISTS 정리
categories:
- A/B Test
- SQL
- 데이터 분석
feature_image: "https://picsum.photos/2560/600?image=872"
---

참고: [Tigercow 블로그](https://doorbw.tistory.com/222)

이번 포스트에서는 IN, EXISTS 문에 대한 상세한 내용을 정리한다.

## 1. 

SELECT * FROM TB_FOOD fWHERE f.number IN (SELECT c.number FROM TB_COLOR c);