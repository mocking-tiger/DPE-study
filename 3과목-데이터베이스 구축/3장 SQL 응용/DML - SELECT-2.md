 ## 일반 형식

 ```
SELECT [PREDICATE] [테이블명.]속성명 [AS 별칭][, [테이블명.]속성명, ...]
[, 그룹합수(속성명) [AS 별칭]]
[, WINDOW함수 OVER (PARTITION BY 속성명1, 속성명2, ...
            ORDER BY 속성명3, 속성명4, ...) [AS 별칭]]
FROM 테이블명[, 테이블명 , ...]
[WHERE 조건]
[GROUP BY 속성명, 속성명, ...]
[HAVING 조건]
[ORDER BY 속성명 [ASC|DESC]];
```

- 그룹함수: GROUP BY절에 지정된 그룹별로 속성의 값을 집계할 함수를 기술한다.
- WINDOW함수: GROUP BY절을 이용하지 않고 속성의 값을 집계할 함수를 기술한다.
  - PARTITION BY: WINDOW함수가 적용될 범위로 사용할 속성을 지정한다.
  - ORDER BY: PARTITION 안에서 정렬 기준으로 사용할 속성을 지정한다.
- GROUP BY절: 특정 속성을 기준으로 그룹화하여 검색할 때 사용한다. 일반적으로 GROUP BY절은 그룹 함수와 함께 사용된다.
- HAVING절: GROUP BY와 함께 사용되며, 그룹에 대한 조건을 지정한다.

## WINDOW함수 이용 검색

GROUP BY절을 이용하지 않고 함수의 인수로 지정한 속성을 범위로 하여 속성의 값을 집계한다.

<상여금>

![image](https://github.com/user-attachments/assets/587f6623-c63c-46e0-be97-45d61ce127df)

예제1) 상여금 테이블에서 '상여내역'별로 '상여금'에 대한 일련번호를 구하시오. (단 순서는 내림차순이며 속성명은 'NO'로 할 것)

```
SELECT 상여내역, 상여금.
  ROW_NUMBER() OVER(PARTITION BY 상여내역 ORDER BY 상여금 DESC) AS NO
FROM 상여금;
```

<결과>

![image](https://github.com/user-attachments/assets/f5eff5cb-cd8f-4d5c-a85c-6730e803f15d)

---

예제2) 상여금 테이블에서 '상여내역'별로 '상여금'에 대한 순위를 구하시오. (단, 순서는 내림차순이며, 속성명은 '상여금순위'로 하고, RANK()함수를 이용할 것)

```
SELECT 상여내역,상여금,
  RANK() OVER(PARTITION BY 상여내역 ORDER BY 상여금 DESC) AS 상여금순위
FROM 상여금;
```

<결과>

![image](https://github.com/user-attachments/assets/7c7d3a1d-538a-4847-a076-c6e6ab70531f)

## 그룹 지정 검색

GROUP BY절에 지정한 속성을 기준으로 자료를 그룹화하여 검색한다.

예제1) 상여금 테이블에서 '부서'별 '상여금'의 평균을 구하시오.

```
SELECT 부서, AVG(상여금) AS 평균
FROM 상여금
GROUP BY 부서;
```

<결과>

![image](https://github.com/user-attachments/assets/a88bbc68-2a4b-4087-9800-0b55e10782fb)

---

예제2) 상여금 테이블에서 부서별 튜플 수를 검색하시오.

```
SELECT 부서, COUNT(*) AS 사원수
FROM 상여금
GROUP BY 부서;
```

<결과>

![image](https://github.com/user-attachments/assets/c4d80e11-88fb-45e0-94c7-7d1165fa8eaf)

---

예제3) 상여금 테이블에서 '상여금'이 100 이상인 사원이 2명 이상인 '부서'의 튜플 수를 구하시오.

```
SELECT 부서, COUNT(*) AS 사원수
FROM 상여금
WHERE 상여금 >= 100
GROUP BY 부서
HAVING COUNT(*) >= 2;
```

<결과>

![image](https://github.com/user-attachments/assets/a6d63759-3ff5-4110-a949-d114c8ee11d2)

---

예제4) 상여금 테이블의 '부서', '상여내역', 그리고 '상여금'에 대해 부서별 상여내역별 소계와 전체 합계를 검색하시오.(단, 속성명은 '상여금합계'로 하고, ROLLUP 함수를 사용할 것)

```
SELECT 부서, 상여내역, SUM(상여금) AS 상여금합계
FROM 상여금
GROUP BY ROLLUP(부서,상여내역);
```

<결과>

![image](https://github.com/user-attachments/assets/4f0c3e4f-70d3-4d5e-aa09-22d58720d3b5)

---

예제5) 상여금 테이블의 '부서', '상여내역', 그리고 '상여금'에 대해 부서별 상여내역별 소계와 전체 합계를 검색하시오. (단, 속성명은 '상여금합계'로 하고, CUBE 함수를 사용할 것)

```
SELECT 부서, 상여내역, SUM(상여금) AS 상여금합계
FROM 상여금
GROUP BY CUBE(부서, 상여내역);
```

<결과>

![image](https://github.com/user-attachments/assets/8a9aad48-9874-40d3-acbb-8f767001f370)

---

## 집합 연산자를 이용한 통합 질의

집합 연산자를 사용하여 2개 이상의 테이블의 데이터를 하나로 통합한다.

### 표기 형식

```
SELECT 속성명1, 속성명2, ...
FROM 테이블명
UNION|UNION ALL|INTERSECT|EXCEPT
SELECT 속성명1, 속성명2,...
FROM 테이블명
[ORDER BY 속성명 [ASC|DESC]];
```

- 두 개의 SELECT문에 기술한 속성들은 개수와 데이터 유형이 서로 동일해야 한다.
- 집합 연산자의 종류(통합 질의의 종류)
- UNION(합집합)
  - 두 SELECT문의 조회 결과를 통합하여 출력한다.
  - 중복된 행은 한 번만 출력한다.
- UNION ALL(합집합)
  - 두 SELECT문의 조회 결과를 통합하여 모두 출력한다.
  - 중복된 행도 그대로 출력한다.
- INTERSECT(교집합)
  - 두 SELECT문의 조회 결과 중 공통된 행만 출력한다.
- EXCEPT(차집합)
  - 첫 번째 SELECT문의 조회 결과에서 두 번째 SELECT문의 조회 결과를 제외한 행을 출력한다.

<사원>

![image](https://github.com/user-attachments/assets/7218a2c6-540e-4924-8414-888036e94b8a)

<직원>

![image](https://github.com/user-attachments/assets/4e68b1cc-e248-4f70-8dbd-977d9269d3d5)

예제1) 사원 테이블과 직원 테이블을 통합하는 질의문을 작성하시오.(단, 같은 레코드가 중복되어 나오지 않게 하시오.)

```
SELECT *
FROM 사원
UNION
SELECT *
FROM 직원;
```

<결과>

![image](https://github.com/user-attachments/assets/1a8f091f-b0b5-4380-8972-e16491ce0618)

---

예제2) 사원 테이블과 직원 테이블에 공통으로 존재하는 레코드만 통합하는 질의문을 작성하시오.

```
SELECT *
FROM 사원
INTERSECT
SELECT  *
FROM 직원;
```

<결과>

![image](https://github.com/user-attachments/assets/023ced62-c988-4755-bddd-0b93abfe3baa)

---





