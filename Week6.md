# SQL_ADVANCED 6주차 정규 과제

## **Week 6 : PIVOT / UNPIVOT**

📌**SQL_ADVANCED 정규과제**는 매주 정해진 주제에 따라 **MySQL 공식 문서 또는 한글 블로그 자료를 참고해 개념을 정리한 후, 프로그래머스 SQL 문제 3문제**와 **추가 확인문제**를 직접 풀어보며 학습하는 과제입니다.

이번 주는 아래의 **SQL_ADVANCED_6th_TIL**에 나열된 주제를 중심으로 개념을 학습하고, 주차별 **학습 목표**에 맞게 정리해주세요. 정리한 내용은 GitHub에 업로드한 후, **스프레드시트의 ‘SQL’ 시트에 링크를 제출**해주세요.

**(수행 인증샷은 필수입니다.)**

> 프로그래머스 문제를 풀고 ‘정답입니다’ 문구를 캡쳐해서 올려주시면 됩니다.

------

## **SQL_ADVANCED_6th_TIL**

- MySQL 공식문서에는 `PIVOT / UNPIVOT`에 대한 **전용 문법을 제공하지 않고 있습니다.** 따라서 이번 주차에서는 PIVOT / UNPIVOT에 대한 개념을 이해하고, 이를 `CASE WHEN + GROUP BY` 또는 `UNION ALL` 등의 조합으로 수동 구현할 수 있는 방법을 학습하면 됩니다. 



## **🏁 강의 수강 (Study Schedule)**

| **주차** | **공부 범위**           | **완료 여부** |
| -------- | ----------------------- | ------------- |
| 1주차    | 서브쿼리 & CTE          | ✅             |
| 2주차    | 집합 연산자 & 그룹 함수 | ✅             |
| 3주차    | 윈도우 함수             | ✅             |
| 4주차    | Top N 쿼리              | ✅             |
| 5주차    | 계층형 질의 & 셀프 조인 | ✅             |
| 6주차    | PIVOT / UNPIVOT         | ✅             |
| 7주차    | 정규 표현식             | 🍽️             |

<br>



# 1️⃣ 학습 내용

**참고할 자료를 아래에 같이 첨부합니다.**

<!-- 꼭 아래의 자료를 참고하지 않고, 개인적인 학습 방법으로 진행하셔도 좋습니다. -->

1. **PIVOT / UNPIVOT 개념 학습 Blog**

https://m.blog.naver.com/regenesis90/222205833002

https://blog.naver.com/regenesis90/222205964866

2. **MySQL 로 PIVOT 구현하기**

https://shxrecord.tistory.com/181



<br>

---

# 2️⃣ 학습 내용 정리하기

## 1. PIVOT

```
✅ 학습 목표 :
* MySQL에서 직접적인 `PIVOT` 문법은 없으므로, `CASE WHEN`, `GROUP BY`, `MAX()` 등으로 대체 구현할 수 있다.
* 데이터를 행 → 열 방향으로 전개하는 기본 로직을 이해한다.
```

<!-- 새롭게 배운 내용을 자유롭게 정리해주세요. -->

- 피벗(PIVOT)은 행(Row) 데이터를 열(Column) 데이터로 전환하여, 데이터를 2차원 표 형태로 요약해주는 기능

### 1) 목적

- 데이터를 특정 기준으로 요약
- 집계함수(합계, 평균, 개수 등)를 이용해 행을 열로 전환
- 가독성 높고 분석에 용이한 구조로 데이터 표현

> 예시: 직책별(job_id) & 부서별(department_id) 직원 수를 테이블 형태로 표현

### 2) 기본 원리

- 기존 집계: GROUP BY + 집계함수를 통해 다중 행으로 결과 출력
- **PIVOT**: 집계된 데이터를 행과 열 축을 바꿔 보기 쉽게 정리
<img width="753" height="354" alt="Image" src="https://github.com/user-attachments/assets/36953192-a135-4b25-a2ff-9bca2c06caf9" />

> 일반 집계 예시 (PIVOT 적용 전)
```sql
SELECT job_id,
       department_id,
       COUNT(employee_id) AS count
FROM employees
GROUP BY job_id, department_id;
```
<img width="422" height="431" alt="Image" src="https://github.com/user-attachments/assets/48eae52f-399e-4879-a543-71a7ce10c368" />

### ✅ 3) 오라클 SQL의 PIVOT 문법
```sql
WITH temp AS (
    SELECT job_id,
           department_id,
           employee_id
    FROM employees
)
SELECT *
FROM temp
PIVOT (
    COUNT(employee_id) AS c
    FOR department_id IN (
        10 AS d10,
        20 AS d20,
        30 AS d30,
        ...
        NULL AS none
    )
);
```
<img width="696" height="369" alt="Image" src="https://github.com/user-attachments/assets/65edce16-c44a-4759-b2e6-2b9f38967420" /><br>
- temp: 서브쿼리 형태로 원본 데이터 지정
- COUNT(employee_id): 집계함수
- FOR department_id IN (...): 피벗할 기준 열과 표시할 열 값들
- AS c: 결과 컬럼에 붙는 접미사 (예: D10_C, D20_C)




## 2. UNPIVOT

```
✅ 학습 목표 :
* UNPIVOT 역시 직접 문법이 없으므로, `UNION ALL`과 `JOIN`, JSON 등의 방법으로 열 → 행 형태로 변환하는 과정을 익힌다.
* `UNION ALL`로 수동 구현 시의 컬럼 이름 통일과 데이터 병합 과정을 익힌다.
```

<!-- 새롭게 배운 내용을 자유롭게 정리해주세요. -->

- UNPIVOT은 기존에 열(Column)로 나뉘어 있던 데이터를 다시 행(Row)으로 되돌리는 연산
- PIVOT의 반대 역할을 하며, “넓은(wide)” 형태의 데이터를 “긴(long)” 형태로 바꾸는 것.

### 1) 목적
- 분석이나 전처리를 위해 데이터를 다시 집계 전 형태로 정규화(normalize) 해야 할 때 사용
- PIVOT으로 데이터를 시각적으로 보기 쉽게 정리했더라도, 다시 분석 도구에 넣기 위해선 원래 구조(행 기반)로 바꿔야 할 때가 많음
- 머신러닝, 통계 분석, 외부 시스템 전송 등에서는 열 기반 데이터 구조가 불편할 수 있기 때문.

### ✅ 2) 오라클 SQL에서의 기본 문법

```sql
SELECT * FROM 테이블이름A
UNPIVOT (
  새_값_컬럼명 FOR 기준_컬럼명 IN (
    기존컬럼1 AS 기준값1,
    기존컬럼2 AS 기준값2,
    ...
  )
);
```
> UNPIVOT 예제 코드
```sql
select * from pivot_sample
UNPIVOT (
count_employee for department_id in (d10_c as 10,
                                     d20_c as 20,
                                     d30_c as 30,
                                     d40_c as 40,
                                     d50_c as 50,
                                     d60_c as 60,
                                     d70_c as 70,
                                     d80_c as 80,
                                     d90_c as 90,
                                     d100_c as 100,
                                     none_c as null)
);
```
<img width="422" height="431" alt="Image" src="https://github.com/user-attachments/assets/48eae52f-399e-4879-a543-71a7ce10c368" />

---

# 3️⃣ 실습 문제

## Leetcode 문제 

https://leetcode.com/problems/reformat-department-table/description/

> LeetCode. Reformat Department Table
>
> 학습 포인트 : MySQL 에서는 PIVOT을 쉽게 구할 수 있는 방법이 없다. 
>
> - 수동으로 구하기 : CASE WHEN + 집계함수 / GROUP BY + 조건 분기 사용



## 문제 인증란

<!-- 이 주석을 지우고 여기에 문제 푼 인증사진을 올려주세요. -->



## 확인 문제 

### 문제 1

> **🧚지희는 매월 각 매장의 월별 매출 데이터가 담긴 테이블을 가공하려 합니다. 아래와 같은 테이블이 있다고 가정합시다.**

| **branch** | **Jan_sales** | **Feb_sales** | **Mar_sales** |
| ---------- | ------------- | ------------- | ------------- |
| A          | 100           | 120           | 130           |
| B          | 90            | 110           | 140           |

> **Q. 이 테이블을 아래와 같은 형태로 바꾸고 싶습니다. SQL에서 UNION ALL을 활용하여 UNPIVOT 구조를 수동으로 구현해보세요.**

| **branch** | **month** | **sales** |
| ---------- | --------- | --------- |
| A          | Jan       | 100       |
| A          | Feb       | 120       |
| A          | Mar       | 130       |
| B          | Jan       | 90        |
| B          | Feb       | 110       |
| B          | Mar       | 140       |

```
여기에 SQL 쿼리를 작성해주세요!
```



### 문제 2

> **🧚태연이는 지점별로 월별 매출을 한 눈에 보기 위해, 아래와 같이 매출 데이터가 저장된 데이터를 가공하려고 합니다.**

| **branch** | **month** | **sales** |
| ---------- | --------- | --------- |
| A          | Jan       | 100       |
| A          | Feb       | 120       |
| A          | Mar       | 130       |
| B          | Jan       | 90        |
| B          | Feb       | 110       |
| B          | Mar       | 140       |

> **이 데이터를 아래와 같이 월별 매출 컬럼이 각각 존재하도록 PIVOT 형태로 바꾸고 싶습니다.MySQL에서는 PIVOT 문법이 없기 때문에, CASE WHEN, GROUP BY, MAX() 또는 SUM() 등을 이용해 수동으로 구현해보세요.**

- 원하는 결과 

| **branch** | **Jan_sales** | **Feb_sales** | **Mar_sales** |
| ---------- | ------------- | ------------- | ------------- |
| A          | 100           | 120           | 130           |
| B          | 90            | 110           | 140           |

~~~
여기에 SQL 쿼리를 작성해주세요!
~~~







### **🎉 수고하셨습니다.**
