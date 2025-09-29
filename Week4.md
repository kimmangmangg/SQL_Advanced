# SQL_ADVANCED 4주차 정규 과제 

## Week 4 : TOP-N 쿼리

📌**SQL_ADVANCED 정규과제**는 매주 정해진 주제에 따라 **MySQL 공식 문서 또는 한글 블로그 자료를 참고해 개념을 정리한 후, 프로그래머스, SolveSql, LeetCode 중에서 SQL 문제 4문제**와 **추가 확인문제**를 직접 풀어보며 학습하는 과제입니다. 

이번 주는 아래의 **SQL_ADVANCED_4th_TIL**에 나열된 주제를 중심으로 개념을 학습하고, 주차별 **학습 목표**에 맞게 정리해주세요. 정리한 내용은 GitHub에 업로드한 후, **스프레드시트의 'SQL' 시트에 링크를 제출**해주세요. 



**(수행 인증샷은 필수입니다.)** 

> 프로그래머스 문제를 풀고 '정답입니다' 문구를 캡쳐해서 올려주시면 됩니다. 



## SQL_ADVANCED_4th

### 15.2.13. SELECT Statement

- `ORDER BY, LIMIT, LIMIT and Subqueries` 중심으로 학습해주세요. 



## 🏁 강의 수강 (Study Schedule)

| 주차  | 공부 범위               | 완료 여부 |
| ----- | ----------------------- | --------- |
| 1주차 | 서브쿼리 & CTE          | ✅         |
| 2주차 | 집합 연산자 & 그룹 함수 | ✅         |
| 3주차 | 윈도우 함수             | ✅         |
| 4주차 | Top N 쿼리              | ✅         |
| 5주차 | 계층형 질의와 셀프 조인 | 🍽️         |
| 6주차 | PIVOT / UNPIVOT         | 🍽️         |
| 7주차 | 정규 표현식             | 🍽️         |



### 공식 문서 활용 팁

>  **MySQL 공식 문서는 영어로 제공되지만, 크롬 브라우저에서 공식 문서를 열고 이 페이지 번역하기에서 한국어를 선택하면 번역된 버전으로 확인할 수 있습니다. 다만, 번역본은 문맥이 어색한 부분이 종종 있으니 영어 원문과 한국어 번역본을 왔다 갔다 하며 확인하거나, 교육팀장의 정리 예시를 참고하셔도 괜찮습니다.**



# 1️⃣ 학습 내용

> 아래의 링크를 통해 *MySQL 공식문서*로 이동하실 수 있습니다.
>
> - 15.2.13 SELECT Statement : MySQL 공식문서 
>
> https://dev.mysql.com/doc/refman/8.0/en/select.html#order-by-optimization
>
> (한국어 버전) https://dart-b-official.github.io/posts/mysql-TopN/
>



<br>

<!-- 여기까진 그대로 둬 주세요-->

---

# 2️⃣ 학습 내용 정리하기

## 1. TOP N 쿼리

~~~
✅ 학습 목표 :
* LIMIT 와 ORDER BY를 이용한 TOP-N 쿼리 작성이 가능하다.
* SubQuery나 RANK 대신 LIMIT으로 간단한 순위 집계가 가능함을 이해한다. 
~~~

물론입니다! 아래는 마크다운 문법을 **정확히 적용한 형태**로 작성한 `2️⃣ 학습 내용 정리하기` 항목입니다.
`깃허브 Markdown 문서`에 그대로 붙여 넣어도 잘 작동합니다.

> ✅ 해당 정리는 [GPT 온라인](https://gptonline.ai/ko/)에서 제공하는 SQL 교육 콘텐츠와 연계하여 학습 효과를 높일 수 있습니다.


### 1. Top-N 쿼리

Top-N 쿼리는 **특정 기준으로 정렬된 데이터 중 상위 N개만 추출**하는 방식의 SQL 쿼리
MySQL에서는 `ORDER BY`와 `LIMIT`을 조합하여 간단히 구현할 수 있음.

---

### 2. 기본 구문

```sql
SELECT 컬럼1, 컬럼2, ...
FROM 테이블명
ORDER BY 기준컬럼 DESC
LIMIT N;
```

- `ORDER BY`: 정렬 기준 지정 (`ASC` 오름차순 / `DESC` 내림차순)
- `LIMIT`: 출력할 행의 수 제한
- `OFFSET`: 시작 위치 지정 (페이징에 활용)

> 예제 1: 급여 상위 5명 조회
```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 5;
```

> 예제 2: 6번째 ~ 10번째 급여 조회
```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 5 OFFSET 5;
```

> 예제 3: 동점자 처리를 위한 다중 정렬
```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC, id ASC
LIMIT 5;
```
> 예제 4: 그룹별 상위 N명은 LIMIT만으로는 불가
- `LIMIT`은 전체 행에 적용되므로, **그룹별 상위 N명 추출**에는 적합하지 않음
- 이 경우 **윈도우 함수**를 사용해야 함. (MySQL 8.0 이상)

```sql
SELECT *
FROM (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
  FROM employees
) AS ranked
WHERE rn <= 3;
```

### 3. SubQuery나 RANK 없이 가능한 단순 Top-N

- 단순한 Top-N 결과(전체 상위 N명)에는 `LIMIT`만으로 충분
- `ROW_NUMBER()`나 `RANK()` 없이도 정렬 + 제한으로 구현 가능
- 예시: 인기 게시글 상위 10개, 최신 공지 5개 등

### 4. 정리 표

| 기능           | `LIMIT + ORDER BY`로 가능 | 윈도우 함수 필요 |
| ------------ | ---------------------- | --------- |
| 전체 상위 N명 추출  | ✅ 가능                   | ❌ 불필요     |
| 그룹별 상위 N명 추출 | ❌ 어려움                  | ✅ 필요      |
| 페이징 처리       | ✅ 가능                   | ❌ 불필요     |
| 동점자 우선순위 설정  | ✅ 가능                   | ✅ 가능      |


<br>

<br>

---

# 3️⃣ 실습 문제

## 문제 

https://leetcode.com/problems/rank-scores/

> LeetCode 178. Rank Scores
>
> 학습 포인트 : DENSE_RANK( )를 활용하여 점수별 순위 부여, 동점자 처리, 윈도우 함수 복습 

https://school.programmers.co.kr/learn/courses/30/lessons/133027

> 프로그래머스 : 주문량이 많은 아이스크림들 조회하기 (Lev 4)
>
> Hint
>
> - 문제 핵심은 '총 주문량 합산' 입니다. 
>
> - 두 테이블을 '세로로' 합쳐야합니다. 
>   - 저희는 이 부분을 1주차에 `UNION ALL` 을 통해 방법을 배웠습니다. 
> - 합쳐진 테이블에서 FLAVOR 별로 그룹화해 주문량을 합산하세요. 
> - 상위 3개를 추출 = 주문량 기준으로 내림차순하여 이번에 학습한 것을 사용해야 합니다. 

---

## 문제 인증란

<!-- 이 주석을 지우고 여기에 문제 푼 인증사진을 올려주세요. -->



---

# 확인문제

## 문제 1

> **🧚영리는 지역별로 가장 인기 있는 식당 2곳씩을 뽑기 위해 다음과 같은 UNION ALL 기반 쿼리를 작성했습니다.**

~~~sql
(
  SELECT region, restaurant_name, review_count
  FROM Restaurants
  WHERE region = '서울'
  ORDER BY review_count DESC
  LIMIT 2
)
UNION ALL
(
  SELECT region, restaurant_name, review_count
  FROM Restaurants
  WHERE region = '부산'
  ORDER BY review_count DESC
  LIMIT 2
)
UNION ALL
(
  SELECT region, restaurant_name, review_count
  FROM Restaurants
  WHERE region = '대구'
  ORDER BY review_count DESC
  LIMIT 2
);
~~~

> **쿼리는 잘 작동하긴 하지만, 지역을 더 추가해달라는 재원이의 부탁으로 UNION ALL 블록을 계속 추가하게 되어 관리가 어려울 것 같아서 힘들어하고 있었습니다. 여러분들은 이 쿼리를 윈도우 함수로 변경하여 더 쉽게 리팩토링을 하려고 합니다. 민서를 도와서 UNION ALL 없이 RANK( ) 또는 ROW_NUMBER( ) 윈도우 함수를 사용해, 각 지역별로 리뷰 수가 가장 많은 상위 2개 식당을 추출하는 쿼리를 작성해보세요.**



~~~
여기에 답을 작성해주세요!
~~~



<br>

### 🎉 수고하셨습니다.
