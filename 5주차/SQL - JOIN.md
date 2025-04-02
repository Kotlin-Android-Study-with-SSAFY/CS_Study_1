## 박상윤

### JOIN이란?

- 둘 이상의 테이블에서 관련 데이터를 결합하여 하나의 결과 세트로 만드는 데 사용되는 명령어
- 데이터베이스 내에서 여러 테이블에 분산되어 저장된 정보를 효과적으로 조회할 수 있음

---

### 내부 조인(Inner Join)

![image](https://github.com/user-attachments/assets/df69d235-6824-4bda-9ad6-acf8cc27840a)

- 두 테이블에서 지정된 조건(보통 공통 열)이 일치하는 행만 반환
- 기존 테이블과 조인 테이블 모두에 조인 컬럼 데이터가 존재해야 조회됨(ON절)

```sql
SELECT 조회할 컬럼
FROM 테이블1
(INNER) JOIN 테이블2
ON 테이블1.컬럼 = 테이블2.컬럼
[WHERE 추가조건]
```

- 예시 - 각 판매 건에 대해 해당 판매가 발생한 국가의 이름을 함께 조회하시오.
    
    ![image1](https://github.com/user-attachments/assets/1c16bbf1-0861-498c-9382-a08dd38ce163)


---

### 자연 조인(Natural Join)

- 내부 조인에 속함
- 두 테이블 간에 동일한 이름을 가진 컬럼들을 기준으로 자동으로 조인 조건을 생성하여 데이터를 결합하는 방법 → 명시적으로 on 절을 작성할 필요 없이 두 테이블에서 이름이 같은 컬럼들을 찾아 그 값이 일치하는 행끼리 결합
- 주의점 - 두 테이블에 원치 않은 공통 컬럼이 있을 경우 의도하지 않은 결과가 발생할 수 있으므로, 사용 전 공통 컬럼을 잘 확인해야함

```sql
SELECT 조회할 컬럼
FROM 테이블1
NATURAL JOIN 테이블2
[WHERE 조건문]
```

![image2](https://github.com/user-attachments/assets/5bcae592-f655-4b48-8db2-d95c28a6a468)

---

### 전체 외부 조인 (Full Outer Join)

![image3](https://github.com/user-attachments/assets/4e08e98c-1f44-46a4-a763-aed6b4fc25e5)

- 양쪽 테이블의 모든 행을 반환하며, 어느 한쪽에 일치하는 데이터가 없으면 NULL을 채워줌
- MySQL에서는 FULL OUTER JOIN을 지원하지 않으므로 LEFT OUTER JOIN 결과와 RIGHT OUTER JOIN 결과를 UNION 하여 사용해야 함

```sql
SELECT 조회할 컬럼
FROM 테이블1 
FULL OUTER JOIN 테이블2
ON 조건문
[WHERE 추가조건문]
```

- 예시 - **모든 교수와 모든 강의 정보**를 한 번에 조회하시오
    
    ![image4](https://github.com/user-attachments/assets/02e22c69-6a1e-4ec9-bfba-dd503a3b487d)


---

### 왼쪽 외부 조인(LEFT OUTER JOIN)

![image5](https://github.com/user-attachments/assets/ad065170-1b9f-47e6-b15e-049cd8ea68ad)

- 왼쪽 테이블의 모든 행을 반환하고, 오른쪽 테이블에서 일치하는 값이 없으면 NULL을 반환

```sql
SELECT 조회할 컬럼
FROM 기준테이블1 
LEFT OUTER JOIN 테이블2
ON 조건문
[WHERE 추가조건문]
```

- 예시 - 모든 교수 정보와 그 교수가 맡고 있는 **강의 정보**를 함께 조회하시오
    
    ![image6](https://github.com/user-attachments/assets/2809672d-ae1f-447d-b032-728121a1068b)


---

### 오른쪽 외부 조인(RIGHT OUTER JOIN)

![image7](https://github.com/user-attachments/assets/2bd6a1d2-2321-40d6-8b96-c937868387e4)

- LEFT OUTER JOIN의 반대 개념으로, 오른쪽 테이블의 모든 행을 반환하고, 왼쪽 테이블에서 일치하는 값이 없으면 NULL을 반환

```sql
SELECT 조회할 컬럼
FROM 테이블1 
RIGHT OUTER JOIN 기준테이블2
ON 조건문
[WHERE 추가조건문]
```

- 예시 - 현재 개설된 모든 강의를 강의를 맡고 있는 **담당 교수**와 함께 조회하시오.
    
    ![image8](https://github.com/user-attachments/assets/e42c29e8-722a-4fd5-8731-98500ee2dc27)


---

### 상호 조인(CROSS JOIN)

![image9](https://github.com/user-attachments/assets/79346e97-8b98-46fe-9369-5468ef81a02e)

- 한쪽 테이블의 모든 행과 다른 쪽 테이블의 모든 행을 조인(곱집합)
- 두 테이블의 모든 가능한 행 조합 → **카테시안 곱(CARTESIAN PRODUCT)**

```sql
SELECT 조회할컬럼
FROM 테이블1
CROSS JOIN 테이블2
[WHERE 추가조건문]
```

- 예시 - **각 제품과 모든 색상 조합**을 만들어 새로운 목록을 작성하시오.
    
    ![image10](https://github.com/user-attachments/assets/af0d4670-8ab7-4eae-b369-cb9e2b518aa9)


---

### 자체 조인(SELF JOIN)

![image11](https://github.com/user-attachments/assets/4408abe1-2c46-4e6f-aa31-5613bf8f2c67)

- 같은 테이블을 두 번 이상 사용하여 자기 자신과 조인
- 테이블에 별칭이 무조건 필요함

```sql
SELECT 조회할컬럼
FROM 테이블1 AS 별칭1
INNER JOIN 테이블1 AS 별칭2
ON 조건문
[WHERE 추가조건문]

SELECT 조회할컬럼
FROM 테이블1 AS 별칭1
LEFT JOIN 테이블1 AS 별칭2
ON 조건문
[WHERE 추가조건문]

SELECT 조회할컬럼
FROM 테이블1 AS 별칭1
RIGHT JOIN 테이블1 AS 별칭2
ON 조건문
[WHERE 추가조건문]
```

- 예시
    
    ![image12](https://github.com/user-attachments/assets/22be6e34-f647-4bb9-aa73-df82264b13fb)

    - INNER JOIN - 상사가 있는 직원만 조회하시오.
    
    ![image13](https://github.com/user-attachments/assets/8e075500-a5d6-42e1-9ef6-55d9ab151691)

    - LEFT JOIN - 모든 직원의 상사 정보를 조회하는데, 상사가 없으면 비워서 조회하시오.
    
    ![image14](https://github.com/user-attachments/assets/cf449a35-c68b-4137-8428-86c5743adf5e)

    - RIGHT JOIN - 모든 상사 정보를 조회하는데, 부하가 없으면 비워서 조회하시오.
 
    ![image15](https://github.com/user-attachments/assets/f454dea3-ca27-48f3-b93f-7834fc43c8aa)
