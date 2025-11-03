# SQL_ADVANCED 5주차 정규 과제 

## Week 5 : 계층형 질의 & 셀프 조인

📌**SQL_ADVANCED 정규과제**는 매주 정해진 주제에 따라 **MySQL 공식 문서 또는 한글 블로그 자료를 참고해 개념을 정리한 후, 프로그래머스 SQL 문제 3문제**와 **추가 확인문제**를 직접 풀어보며 학습하는 과제입니다. 

이번 주는 아래의 **SQL_ADVANCED_5th_TIL**에 나열된 주제를 중심으로 개념을 학습하고, 주차별 **학습 목표**에 맞게 정리해주세요. 정리한 내용은 GitHub에 업로드한 후, **스프레드시트의 'SQL' 시트에 링크를 제출**해주세요. 



**(수행 인증샷은 필수입니다.)** 

> 프로그래머스 문제를 풀고 '정답입니다' 문구를 캡쳐해서 올려주시면 됩니다. 



## SQL_ADVANCED_5th

### 15.2.20 WITH (Common Table Expressions)

- **재귀 CTE를 통한 계층형 구조 탐색 방법을 중심으로 학습해주세요.**

> Self Join은 따로 MySQL 공식문서가 없습니다. 다른 블로그나 유튜브 영상을 참고하여 스스로 학습하고, 넣어주세요. 



## 🏁 강의 수강 (Study Schedule)

| 주차  | 공부 범위               | 완료 여부 |
| ----- | ----------------------- | --------- |
| 1주차 | 서브쿼리 & CTE          | ✅         |
| 2주차 | 집합 연산자 & 그룹 함수 | ✅         |
| 3주차 | 윈도우 함수             | ✅         |
| 4주차 | Top N 쿼리              | ✅         |
| 5주차 | 계층형 질의와 셀프 조인 | ✅         |
| 6주차 | PIVOT / UNPIVOT         | 🍽️         |
| 7주차 | 정규 표현식             | 🍽️         |



### 공식 문서 활용 팁

>  **MySQL 공식 문서는 영어로 제공되지만, 크롬 브라우저에서 공식 문서를 열고 이 페이지 번역하기에서 한국어를 선택하면 번역된 버전으로 확인할 수 있습니다. 다만, 번역본은 문맥이 어색한 부분이 종종 있으니 영어 원문과 한국어 번역본을 왔다 갔다 하며 확인하거나, 교육팀장의 정리 예시를 참고하셔도 괜찮습니다.**



# 1️⃣ 학습 내용

> 아래의 링크를 통해 *MySQL 공식문서*로 이동하실 수 있습니다.
>
> - 15.2.20 WITH (Common Table Expressions) : MySQL 공식문서 
>
> https://dev.mysql.com/doc/refman/8.0/en/with.html
>
> (한국어 버전) https://dart-b-official.github.io/posts/mysql-RecursiveWith/



<br>

---

# 2️⃣ 학습 내용 정리하기

## 1. 계층형 질의 (WITH RECURSIVE)

~~~
✅ 학습 목표 :
* 'WITH RECURSIVE' 문법을 활용해 계층형 구조를 탐색할 수 있다.
~~~

* **재귀 CTE (Common Table Expression)**: 자기 자신을 참조하는 서브쿼리를 포함한 CTE.
* 사용 예:

```sql
WITH RECURSIVE cte (n) AS (
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte
  WHERE n < 5
)
SELECT * FROM cte;
```

* CTE가 자기 자신을 참조하면 반드시 `WITH RECURSIVE` 사용.
* `RECURSIVE` 생략 시: `ERROR 1146` 발생.

### 재귀 CTE의 두 부분
* **비재귀 SELECT**: 초기 시드값. CTE 이름 참조 없음.
* **재귀 SELECT**: 이전 결과 참조. CTE 이름을 `FROM`에 1회 참조.

### 열 타입과 NULL
* 종료 조건 없으면 무한 루프 발생 가능.
* 열 타입은 비재귀 SELECT 기준.
* 모든 열은 NULL 허용으로 간주.

### UNION 종류
* `UNION ALL`: 중복 허용
* `UNION DISTINCT`: 중복 제거 → 무한 루프 방지

* 재귀 SELECT는 직전 반복에서 생성된 행만을 입력으로 사용.
* 여러 쿼리 블록이 있을 경우 실행 순서는 정의되지 않음.
* 열은 위치가 아니라 이름 기준으로 참조됨.

### 컬럼 폭(CAST) 이슈
* 비재귀 SELECT에서 정의한 문자열 길이보다 재귀 SELECT 결과가 길면 잘릴 수 있음 ➡️ 해결: CAST()로 열 폭 미리 확장.
* 사용 예:

```sql
WITH RECURSIVE cte AS (
  SELECT 1 AS n, CAST('abc' AS CHAR(20)) AS str
  UNION ALL
  SELECT n + 1, CONCAT(str, str) FROM cte WHERE n < 3
)
SELECT * FROM cte;
```

### 문법 제약 (재귀 SELECT 내부)

* ❌ 사용 불가: `집계 함수`, `윈도 함수`, `GROUP BY`, `ORDER BY`, `DISTINCT`
* `LIMIT`: MySQL 8.0.19 이상부터 사용 가능

### 기타 제약

* 재귀 SELECT는 CTE를 `FROM`에서 1회만 참조 가능
* `LEFT JOIN`의 오른쪽에서 CTE 사용 불가

### 성능 및 안전장치

* `cte_max_recursion_depth`: 최대 재귀 깊이 제한
* `max_execution_time`: 실행 시간 제한
* 옵티마이저 힌트: `/*+ MAX_EXECUTION_TIME(1000) */`

```sql
SET SESSION cte_max_recursion_depth = 10;
SET SESSION max_execution_time = 1000;

WITH RECURSIVE cte(n) AS (
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte LIMIT 10000
)
SELECT /*+ MAX_EXECUTION_TIME(1000) */ * FROM cte;
```

## 2. 셀프 조인

~~~
✅ 학습 목표 :
* 같은 테이블 내에서 상호 관계를 탐색할 수 있는 셀프 조인의 구조를 이해하고 사용할 수 있다. 
~~~

* 셀프 조인(Self Join) 은 하나의 테이블을 자기 자신과 조인하는 SQL 문법
* 한 테이블의 두 행 간의 관계(상하, 연결, 부모-자식 등) 를 찾고자 할 때 주로 사용
* 동일한 테이블을 두 개의 별칭(alias) 으로 나눠 사용

```sql
SELECT A.컬럼1, B.컬럼2
FROM 테이블명 AS A
JOIN 테이블명 AS B ON A.기준컬럼 = B.기준컬럼;
```

* 사용 예: 직원과 매니저 관계

```sql
SELECT emp.name AS 직원, mgr.name AS 매니저
FROM employees AS emp
JOIN employees AS mgr ON emp.manager_id = mgr.id;
```
* employees 테이블 내에서 manager_id를 통해 직원-매니저 관계를 찾는 것이 목적이라고 한다면,
* 위의 코드처럼 같은 테이블의 두 역할(직원/매니저)을 나눠서 조인해야 함

### 셀프 조인과 일반 조인의 차이점

| 구분    | 일반 조인       | 셀프 조인              |
| ----- | ----------- | ------------------ |
| 조인 대상 | 서로 다른 두 테이블 | 같은 테이블을 두 번 참조     |
| 별칭 사용 | 필수는 아님      | **별칭 필수** (혼동 방지용) |



<br>

<br>

---

# 3️⃣ 실습 문제

## 문제 

- https://leetcode.com/problems/employees-earning-more-than-their-managers/ 

> LeetCode 181. Employees  Earning More Than Their Managers
>
> 학습 포인트 : 동일 테이블을 두 번 조인 (왜 동일 테이블을 JOIN 해야하는 문제일까)

- https://leetcode.com/problems/tree-node/description/

> LeetCode 608. Tree Node 
>
> 학습 포인트 : id, parent_id 기반의 트리 구조에서 **부모 ~ 자식 관계 재귀 탐색**
>
> Hint : (문제 해석) 
>
> - 어떤 노드가 Root Node 이려면, 부모노드가 존재하지 않아야 한다. 
> - 어떤 노드가 Inner Node 이려면, 나를 부모로 가지는 노드가 하나 이상 존재하여야 한다.
>   - 그 외네는 모두 Leaf Node 이다. --> (CASE 문을 사용하는 것을 추천드립니다.)

- https://school.programmers.co.kr/learn/courses/30/lessons/144856

> 프로그래머스 : 저자 별 카테고리 별 매출액 집계하기 
>
> 학습 포인트 : 카테고리와 서브카테고리 계층 구조를 분석하는 로직, SELF JOIN / CTE를 다 활용할 수 있다.
>
> - 위에 2가지의 문제를 풀어보고 난 이후, 더 편리한 방법으로 문제를 풀어보세요.

---

## 문제 인증란

<!-- 이 주석을 지우고 여기에 문제 푼 인증사진을 올려주세요. -->



---

# 확인문제

## 문제 1

> **🧚윤서는 어떤 기업의 조직 구조를 분석하는 SQL 쿼리를 작성하고 있습니다. 각 직원은 상위 관리자 ID(manager_id)를 가지며, 조직도는 같은 Employees 테이블 내에서 계층적으로 연결됩니다. 윤서는 최상위 관리자부터 각 사원까지의 계층 깊이(depth)를 계산하고자 다음과 같은 SELF JOIN 기반 쿼리를 시도했습니다.** 

~~~sql
SELECT e1.id, e1.name, e2.name AS manager_name
FROM Employees e1
LEFT JOIN Employees e2 ON e1.manager_id = e2.id;
~~~

> **쿼리를 잘 작성했다고 생각을 했지만, 막상 실행을 해보니 1단계 매니저까지만 추적할 수 있어 계층 구조의 전체를  표현하는데 한계가 존재했습니다. 이에 여러분에게 다음과 같은 미션을 요청합니다. WITH RECURSIVE를 활용하여  최상위 관리자부터 시작해 각 직원까지의 조직 구조 계층 깊이(depth)를 구하고, 결과를 depth가 높은 순으로 정렬하는 쿼리를 작성하세요.**



~~~sql
WITH RECURSIVE org_chart AS (
  -- 비재귀: 최상위 관리자부터 시작
  SELECT 
    id, 
    name, 
    manager_id, 
    1 AS depth
  FROM Employees
  WHERE manager_id IS NULL

  UNION ALL

  -- 재귀: 각 직원의 하위 직원 탐색
  SELECT 
    e.id, 
    e.name, 
    e.manager_id, 
    oc.depth + 1
  FROM Employees e
  JOIN org_chart oc ON e.manager_id = oc.id
)

-- 최종 출력
SELECT * 
FROM org_chart
ORDER BY depth DESC;
~~~



---

### 참고자료

<!--셀프조인에 대해 학습하시기에 도움이 되도록 참고할말한 잘 설명된 블로그들을 같이 첨부하겠습니다. -->

https://step-by-step-digital.tistory.com/101



<br>

### 🎉 수고하셨습니다.
