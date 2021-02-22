# **Oracle Basic SQL Cheat Sheet**

## **Date** : **`Oct 2, 2020`**   
## **Tags** : **`CSE 215-216 DATABASE`**

&nbsp;
# **Some Random Operations : `Concatenation` ,  `as` ,  `null` arithmetic**

```sql
SELECT employee_id , (first_name || ' ' || last_name) as name , (salary+salary*commission_pct) SAL
FROM HR.EMPLOYEES;
```

# **Duplication Removal : `DISTINCT`**

```sql
SELECT DISTINCT job_id , department_id
FROM HR.EMPLOYEES;
```

# **Condition : `WHERE`**

```sql
SELECT last_name , salary
FROM HR.EMPLOYEES
WHERE employee_id = 100;
```

# **List Traversal : **`IN`** , **`NOT IN`****
```sql
SELECT DISTINCT job_id
FROM HR.EMPLOYEES
WHERE job_id IN ('AC_MGR');

SELECT DISTINCT job_id
FROM HR.EMPLOYEES
WHERE job_id NOT IN ('AC_ACCOUNT' , 'AC_MGR');
```

# **Operators**

**`<>`** not equal

**`BETWEEN value1 AND value2`**   

**`IN (value 1,value 2, ... )`**

```sql
SELECT last_name , salary
FROM HR.EMPLOYEES
WHERE employee_id <> 100 AND salary BETWEEN 10000 AND 17000;
```

**`IS NULL`** / **`IS NOT NULL`** **( can't use other operators on NULL , syntactically correct though : will return nothing )**

```sql
SELECT last_name , salary
FROM HR.EMPLOYEES
WHERE commission_pct IS NOT NULL;
```

***Date Comparison and Date Operations***

```sql
SELECT last_name , salary , hire_date
FROM HR.EMPLOYEES
WHERE hire_date < '01-JAN-2003';
```

- **Better practice** is to convert to char (**`to_char`**) when displaying and convert to date (**`to_date`**) when comapring

    ```sql
    SELECT last_name , salary , to_char(hire_date,'dd-Mon-yyyy')
    FROM HR.EMPLOYEES
    WHERE hire_date < to_date('01-JAN-2003','dd-Mon-yyyy');
    ```


- **`SYSDATE`** : builtin ORACLE function for current system date . Oracle stores dates in an internal numeric format: century, year, month, day, hours, minutes, seconds. **SYSDATE** is a function that returns the current date and time
- If you add or subtract a **DATE** and a **NUMBER**, the number is
interpreted as an _**interval expressed in days**_

    ```sql
    SELECT HIRE_DATE , SYSDATE - HIRE_DATE , HIRE_DATE + 30
    FROM HR.EMPLOYEES;
    ```
- **DUAL** : a dummy table used to view `SYSDATE`.
  ```sql
  SELECT SYSDATE FROM dual;
  ```

# **Sorting : `ORDER BY` ASC / DESC [ Default : **ASC** ]**

```sql
SELECT last_name , salary , hire_date
FROM HR.EMPLOYEES
ORDER BY hire_date DESC;
```

- ***Multiple** `ORDER BY`*

    works like cpp comparators : first by 1st parameter , if can't break tie then 2nd parameter and so on

    ```sql
    SELECT last_name , salary , hire_date
    FROM HR.EMPLOYEES
    ORDER BY hire_date ASC , last_name DESC;
    ```

- *`ORDER BY` using **Alias***

    ```sql
    SELECT  job_title , (max_salary-min_salary) as DIFF
    FROM HR.JOBS
    ORDER BY DIFF;
    ```

- **`ORDER BY`** with **NULL VALUES**  
  - **NULLS LAST** is the default for **_ascending_** (which itself is the default ordering sequence) order
  - **NULLS FIRST** is the default for **_descending_** order
  - **custom definition**
    ```sql
    SELECT commission_pct
    FROM HR.EMPLOYEES
    ORDER BY commission_pct DESC NULLS LAST;
    ```
# **Pattern Matching : `LIKE` / `REGEXP_LIKE`**

- **`Like`**

    **Case Sensitive**

    - **`%`** 0 or more [ Equivalent to Kleen Star of Regex `*` ]
    - **`_`** only one character

    ```sql
    SELECT last_name , salary
    FROM HR.EMPLOYEES
    WHERE last_name LIKE 'S%n';
    ```

- **`REGEXP_LIKE`**

    **Case Sensitive**

    - **`^`** matches the beginning of a string
    - **`$`** matches the end of a string
    - **`*`** matches zero or more occurrences
    - **`+`** matches one or more occurrences
    - **`|`** used like an "OR" to specify more than one alternative.
    - **`[]`** same as "OR" / union , just compact notation

    More details : [https://www.techonthenet.com/oracle/regexp_like.php](https://www.techonthenet.com/oracle/regexp_like.php)

    ```sql
    SELECT last_name , salary
    FROM HR.EMPLOYEES
    WHERE REGEXP_LIKE (last_name , '^Su([l])*y');
    ```

# **Data Aggregation : `SUM` `MIN` `MAX` `AVG` `COUNT` `GROUP BY`**

```sql
-- count() counts non null values
SELECT SUM(salary) , MAX(salary) , MIN(salary) , COUNT(salary) , AVG(salary)
FROM HR.EMPLOYEES
```

```sql
-- count(*) counts both null and non null values
SELECT job_id , COUNT(*)
FROM HR.EMPLOYEES
GROUP BY job_id;
```

- ****`DISTINCT`** with aggregation**

    ```sql
    SELECT SUM(salary) , MAX(salary) , MIN(salary) , COUNT(salary) , AVG(salary)
    FROM HR.EMPLOYEES;
    SELECT SUM( DISTINCT salary) , MAX(salary) , MIN(salary) , COUNT(DISTINCT salary) , AVG( DISTINCT salary)
    FROM HR.EMPLOYEES;
    ```

- Can't select column **which is not included** in the **`GROUP BY`** condition  
  All columns (**that are not GROUP function**) MUST be in the **`GROUP BY`** condition  
  But , the **`GROUP BY`** column doesn't have to be in the **`SELECT`** list

    ```sql
    SELECT job_id , SUM(salary) , MAX(salary) , MIN(salary) , COUNT(salary) , AVG(salary)
    FROM HR.EMPLOYEES
    GROUP BY job_id;
    ```

    ```sql
    SELECT job_id , COUNT(*)
    FROM HR.EMPLOYEES
    GROUP BY job_id
    ORDER BY COUNT(*) DESC;

    SELECT department_id , COUNT(*)
    FROM HR.EMPLOYEES
    GROUP BY department_id
    ORDER BY COUNT(*) DESC;

    SELECT job_id , department_id , COUNT(*)
    FROM HR.EMPLOYEES
    GROUP BY job_id , department_id
    ORDER BY COUNT(*) DESC;
    ```

- **with **`WHERE`** clause**

    ```sql
    SELECT department_id , count(*)
    FROM HR.EMPLOYEES
    WHERE salary > 3000
    GROUP BY department_id;
    ```

- **with **`HAVING`** clause**
    - Some fundamental difference with **`WHERE`** and **`HAVING`**
        - **`WHERE`** is executed before **`GROUP BY`** , and **`HAVING`** is after
        - **`WHERE`** **doesn't** need **`GROUP BY`** i.e can be used anywhere , whereas **`HAVING`** can't be used without **`GROUP BY`**
        - And the most important one : **`WHERE`** can't contain aggregate functions , but **`HAVING`** can

    ```sql
    -- both can be used

    SELECT department_id , SUM(salary) , MAX(salary) , MIN(salary) , COUNT(salary) , AVG(salary)
    FROM HR.EMPLOYEES
    WHERE department_id <> 80
    GROUP BY department_id;

    SELECT department_id , SUM(salary) , MAX(salary) , MIN(salary) , COUNT(salary) , AVG(salary)
    FROM HR.EMPLOYEES
    HAVING department_id <> 80
    GROUP BY department_id;
    ```

    ```sql
    -- only HAVING can be used

    SELECT department_id , SUM(salary) , MAX(salary) , MIN(salary) , COUNT(salary) , AVG(salary)
    FROM HR.EMPLOYEES
    GROUP BY department_id
    HAVING MAX(salary) > 5000;
    ```

- **Custom **`GROUP BY`** definition**

    ```sql
    SELECT TRUNC(salary/5000) as NEW_GROUP , COUNT(*)
    FROM HR.EMPLOYEES
    GROUP BY TRUNC(salary/5000);
    ```

    ```sql
    SELECT (TRUNC(salary/5000,0)) as NEW_GROUP , (TRUNC(salary/5000,0) * 5000) as LOWER  , ( (TRUNC(salary/5000,0) + 1 ) * 5000) as HIGHER, COUNT(*)
    FROM HR.EMPLOYEES
    GROUP BY TRUNC(salary/5000,0);
    ```

# **NULL Operations** : **`IS NULL`** , **`IS NOT NULL`** , **`NVL`** ,**`NVL2`**

```sql
SELECT department_id, job_id , MIN(hire_date) , MAX(hire_date) , AVG(salary)
FROM HR.EMPLOYEES
WHERE department_id IS NOT NULL
GROUP BY department_id, job_id
HAVING AVG(salary) >6000;
```
- **`SUM`** **`MAX`** **`MIN`** etc operations _**ignore**_ **NULL VALUES**
    ```sql
    <!-- here the ones with null or not counted in the denominator -->
    SELECT AVG(commission_pct)  
    FROM HR.EMPLOYEES

    <!-- here you can see the difference -->
    SELECT AVG(NVL(commission_pct,0))  
    FROM HR.EMPLOYEES;
    ```
- **`NVL`** ( **field_to_check** , **value_if_null** )
    ```sql
    SELECT AVG(NVL(commission_pct,0))  
    FROM HR.EMPLOYEES
    ```
- **`NVL2`** ( **field_to_check** , **value_if_NOT_null** , **value_if_null** )
    ```sql
    SELECT (last_name || ' , ' || first_name) as "Full Name" , (salary*12+NVL2(commission_pct,commission_pct,0)) as "Annul salary" , (salary*12+NVL2(commission_pct,commission_pct,0) - 0.05*(salary*12+NVL2(commission_pct,commission_pct,0))) as "Deduced Salary"
    FROM HR.EMPLOYEES
    WHERE last_name LIKE 'S%' or last_name LIKE 's%';
    ```
- **`GROUP BY`** using **NULL** or not
  ```sql
  SELECT NVL2(commission_pct,1,0) as "HAS COMMISSION" , COUNT(*)
  FROM HR.EMPLOYEES
  GROUP BY NVL2(commission_pct,1,0);
  ```

# **Execution Order**

- **`FROM`**
- **`WHERE`**
- **`GROUP BY`**
- **`HAVING`**
- **`SELECT`**
- **`ORDER BY`**