# **ORACLE Advanced SQL Cheat Sheet**

## **Date** : **`Oct 6, 2020`**   
## **Tags** : **`CSE 215-216 DATABASE`**

&nbsp;
# **Some Miscellaneous Queries**

```sql
SELECT LPAD(SALARY,10,0)
FROM HR.EMPLOYEES;

SELECT SUBSTR(FIRST_NAME,1,2) -- prints substring[l...r] : 1 based indexing
FROM HR.EMPLOYEES;

SELECT LENGTH(FIRST_NAME)
FROM HR.EMPLOYEES;

-- searches for the given substring
SELECT INSTR(FIRST_NAME,'Ellen')
FROM HR.EMPLOYEES;
```

```sql
SELECT (SYSDATE-HIRE_DATE) -- differnce in days
FROM HR.EMPLOYEES;

SELECT (SYSDATE-HIRE_DATE)/365 -- differnce in years
FROM HR.EMPLOYEES;

SELECT ROUND(HIRE_DATE, 'YEAR') -- rounding to nearest month
FROM HR.EMPLOYEES;

SELECT ADD_MONTHS(HIRE_DATE, 5) -- adding 5 months to hire date
FROM HR.EMPLOYEES;
```

# **Join**

This is like cartesian product (all possible combinations)

```sql
SELECT e.job_id 
FROM HR.EMPLOYEES e;

SELECT d.department_id
FROM HR.DEPARTMENTS d;

SELECT e.job_id , e.FIRST_NAME , d.DEPARTMENT_NAME ,d.department_id 
FROM HR.EMPLOYEES e , HR.DEPARTMENTS d
-- WHERE e.DEPARTMENT_ID = d.DEPARTMENT_ID
ORDER BY e.job_id , d.DEPARTMENT_ID , d.DEPARTMENT_NAME;
```

# **Inner Join : `JOIN ... ON` / `INNER JOIN ... ON`**

Common from both table

```sql
-- using : where
SELECT e.job_id , d.department_id , d.DEPARTMENT_NAME
FROM HR.EMPLOYEES e , HR.DEPARTMENTS d
WHERE e.DEPARTMENT_ID = d.DEPARTMENT_ID -- using other operators other than =
ORDER BY e.job_id , d.DEPARTMENT_ID;

-- using : join on
SELECT e.job_id , d.department_id , d.DEPARTMENT_NAME
FROM HR.DEPARTMENTS d JOIN  HR.EMPLOYEES e
ON e.DEPARTMENT_ID = d.DEPARTMENT_ID -- using other operators other than =
ORDER BY e.job_id , d.DEPARTMENT_ID;
```

# **Outer Join : `LEFT JOIN ... ON` / `...(+) = ...` , `RIGHT JOIN ... ON` / `... = ... (+)`**

Either keeps Full Left Table / Full Right Table

```sql
-- Left Outer Join (Keeps all rows from Left Table)
-- using : where
SELECT e.job_id , d.department_id , d.DEPARTMENT_NAME
FROM HR.EMPLOYEES e , HR.DEPARTMENTS d
WHERE e.DEPARTMENT_ID(+) = d.DEPARTMENT_ID
ORDER BY e.job_id , d.DEPARTMENT_ID;

-- using : join on
SELECT e.job_id , d.department_id , d.DEPARTMENT_NAME
FROM HR.DEPARTMENTS d LEFT JOIN  HR.EMPLOYEES e
ON e.DEPARTMENT_ID = d.DEPARTMENT_ID
ORDER BY e.job_id , d.DEPARTMENT_ID;

-- Right Outer Join (Keeps all rows from Right Table)
SELECT e.job_id , d.department_id , d.DEPARTMENT_NAME
FROM HR.EMPLOYEES e , HR.DEPARTMENTS d
WHERE e.DEPARTMENT_ID = d.DEPARTMENT_ID(+)
ORDER BY e.job_id , d.DEPARTMENT_ID;

-- using : join on
SELECT e.job_id , d.department_id , d.DEPARTMENT_NAME
FROM HR.DEPARTMENTS d RIGHT JOIN  HR.EMPLOYEES e
ON e.DEPARTMENT_ID = d.DEPARTMENT_ID
ORDER BY e.job_id , d.DEPARTMENT_ID;
```

# **Self Join**

- [https://www.oracletutorial.com/oracle-basics/oracle-self-join/](https://www.oracletutorial.com/oracle-basics/oracle-self-join/)
- [https://www.sqlitetutorial.net/sqlite-self-join/](https://www.sqlitetutorial.net/sqlite-self-join/)

    Because you cannot refer to the same table more than one in a query, you need to use a table alias to assign the table a different name when you use self-join. The self-join compares values of the same or different columns in the same table. Only one table is involved in the self-join.

```sql
-- using : where
SELECT
		e.EMPLOYEE_ID,
    (e.first_name || '  ' || e.last_name) employee,
    (m.first_name || '  ' || m.last_name) manager,
    e.job_id
FROM employees e , employees m
WHERE m.EMPLOYEE_ID(+)  = e.MANAGER_ID
ORDER BY manager;

-- using : join on
SELECT
		e.EMPLOYEE_ID,
    (e.first_name || '  ' || e.last_name) employee,
    (m.first_name || '  ' || m.last_name) manager,
    e.job_id
FROM employees e LEFT JOIN employees m 
ON m.employee_id = e.manager_id
ORDER BY manager;
```

# **Multiple Table Join**

```sql
SELECT E.LAST_NAME, D.DEPARTMENT_NAME, L.CITY 
FROM EMPLOYEES E 
JOIN DEPARTMENTS D
ON (E.DEPARTMENT_ID = D.DEPARTMENT_ID) 
JOIN LOCATIONS L
ON (D.LOCATION_ID = L.LOCATION_ID);
```

# **Sub Queries**

```sql
SELECT LAST_NAME  , SALARY
FROM HR.EMPLOYEES
WHERE SALARY >
(
-- another query
)
```

# **Single Row Sub Query**

```sql
SELECT LAST_NAME  , SALARY
FROM HR.EMPLOYEES
WHERE SALARY >
(
	SELECT SALARY
	FROM HR.EMPLOYEES
	WHERE LAST_NAME = 'Abel'
)
```

```sql
SELECT LAST_NAME  , SALARY
FROM HR.EMPLOYEES
WHERE SALARY >
(
	SELECT MIN(AVG(SALARY))
	FROM HR.EMPLOYEES
	GROUP BY JOB_ID
)
```

- Multiple Single Row Sub Query

    ```sql
    SELECT LAST_NAME  , SALARY
    FROM HR.EMPLOYEES
    WHERE SALARY >
    (
    	SELECT MIN(AVG(SALARY))
    	FROM HR.EMPLOYEES
    	GROUP BY JOB_ID
    )
    AND SALARY >
    (
    	SELECT AVG(AVG(SALARY))
    	FROM HR.EMPLOYEES
    	GROUP BY JOB_ID
    )
    ```

# **Multiple Row Sub Query**

```sql
SELECT LAST_NAME  , SALARY
FROM HR.EMPLOYEES
WHERE SALARY > ANY
(
	SELECT AVG(SALARY)
	FROM HR.EMPLOYEES
	GROUP BY JOB_ID
)
AND SALARY > ALL
(
	SELECT MIN(SALARY)
	FROM HR.EMPLOYEES
	GROUP BY JOB_ID
);
```

# **Multiple Column Sub Query**

```sql
SELECT LAST_NAME  , SALARY
FROM HR.EMPLOYEES
WHERE (JOB_ID,DEPARTMENT_ID) = 
(
	SELECT JOB_ID,DEPARTMENT_ID
	FROM HR.EMPLOYEES
	WHERE LAST_NAME = 'Austin'
)
```

# **Set Operations : `UNION` **`UNION ALL` `INTERSECT` `MINUS`****

```sql
SELECT LAST_NAME  , SALARY
FROM HR.EMPLOYEES
WHERE JOB_ID ='PU_CLERK'
INTERSECT
(
SELECT LAST_NAME  , SALARY
FROM HR.EMPLOYEES
WHERE DEPARTMENT_ID = 30
);
```

# **Insert : `INSERT INTO ... VALUES ...`**

```sql
INSERT INTO suppliers
(supplier_id, supplier_name)
VALUES
(5000, 'Apple');

-- if inserting all the columns , can ignore column names
INSERT INTO suppliers
VALUES
(5000, 'Apple');
```

- Check if already not present

    ```sql
    INSERT INTO clients
    (client_id, client_name, client_type)
    SELECT supplier_id, supplier_name, 'advertising'
    FROM suppliers
    WHERE NOT EXISTS (SELECT *
                      FROM clients
                      WHERE clients.client_id = suppliers.supplier_id);
    ```

# **Exercise**

```sql
SELECT J.JOB_TITLE, D.DEPARTMENT_NAME, COUNT(*)
FROM DEPARTMENTS D, JOBS J , EMPLOYEES E
WHERE D.DEPARTMENT_ID = E.DEPARTMENT_ID 
AND J.JOB_ID = E.JOB_ID
GROUP BY J.JOB_TITLE, D.DEPARTMENT_NAME;
```

```sql
SELECT e1.EMPLOYEE_ID , COUNT(e2.EMPLOYEE_ID) , e1.LAST_NAME
FROM EMPLOYEES e1 , EMPLOYEES e2
WHERE e1.HIRE_DATE > e2.HIRE_DATE
GROUP BY e1.EMPLOYEE_ID, e1.LAST_NAME;
```

```sql
SELECT e1.LAST_NAME , e1.EMPLOYEE_ID
FROM EMPLOYEES e1
WHERE 3 < 
(
SELECT COUNT(*)
FROM EMPLOYEES e2
WHERE e1.SALARY > e2.SALARY
);
```