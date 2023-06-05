


> Written with [StackEdit](https://stackedit.io/).
> **Schema (PostgreSQL v15)**

    -- Create 'departments' table
    CREATE TABLE departments (
        id SERIAL PRIMARY KEY,
        name VARCHAR(50),
        manager_id INT
    );
    
    -- Create 'employees' table
    CREATE TABLE employees (
        id SERIAL PRIMARY KEY,
        name VARCHAR(50),
        hire_date DATE,
        job_title VARCHAR(50),
        department_id INT REFERENCES departments(id)
    );
    
    -- Create 'projects' table
    CREATE TABLE projects (
        id SERIAL PRIMARY KEY,
        name VARCHAR(50),
        start_date DATE,
        end_date DATE,
        department_id INT REFERENCES departments(id)
    );
    
    -- Insert data into 'departments'
    INSERT INTO departments (name, manager_id)
    VALUES ('HR', 1), ('IT', 2), ('Sales', 3);
    
    -- Insert data into 'employees'
    INSERT INTO employees (name, hire_date, job_title, department_id)
    VALUES ('John Doe', '2018-06-20', 'HR Manager', 1),
           ('Jane Smith', '2019-07-15', 'IT Manager', 2),
           ('Alice Johnson', '2020-01-10', 'Sales Manager', 3),
           ('Bob Miller', '2021-04-30', 'HR Associate', 1),
           ('Charlie Brown', '2022-10-01', 'IT Associate', 2),
           ('Dave Davis', '2023-03-15', 'Sales Associate', 3);
    
    -- Insert data into 'projects'
    INSERT INTO projects (name, start_date, end_date, department_id)
    VALUES ('HR Project 1', '2023-01-01', '2023-06-30', 1),
           ('IT Project 1', '2023-02-01', '2023-07-31', 2),
           ('Sales Project 1', '2023-03-01', '2023-08-31', 3);
           
           UPDATE departments
    SET manager_id = (SELECT id FROM employees WHERE name = 'John Doe')
    WHERE name = 'HR';
    
    UPDATE departments
    SET manager_id = (SELECT id FROM employees WHERE name = 'Jane Smith')
    WHERE name = 'IT';
    
    UPDATE departments
    SET manager_id = (SELECT id FROM employees WHERE name = 'Alice Johnson')
    WHERE name = 'Sales';
    
    

---

**Query #1**

    SELECT department_id, name, start_date, end_date, 
           (end_date - start_date) AS duration
    FROM projects
    WHERE end_date > CURRENT_DATE
    ORDER BY duration DESC;

| department_id | name            | start_date               | end_date                 | duration |
| ------------- | --------------- | ------------------------ | ------------------------ | -------- |
| 3             | Sales Project 1 | 2023-03-01T00:00:00.000Z | 2023-08-31T00:00:00.000Z | 183      |
| 1             | HR Project 1    | 2023-01-01T00:00:00.000Z | 2023-06-30T00:00:00.000Z | 180      |
| 2             | IT Project 1    | 2023-02-01T00:00:00.000Z | 2023-07-31T00:00:00.000Z | 180      |

---
**Query #2**

    SELECT *
    FROM employees
    WHERE id NOT IN (SELECT manager_id FROM departments);

| id  | name          | hire_date                | job_title       | department_id |
| --- | ------------- | ------------------------ | --------------- | ------------- |
| 4   | Bob Miller    | 2021-04-30T00:00:00.000Z | HR Associate    | 1             |
| 5   | Charlie Brown | 2022-10-01T00:00:00.000Z | IT Associate    | 2             |
| 6   | Dave Davis    | 2023-03-15T00:00:00.000Z | Sales Associate | 3             |

---
**Query #3**

    SELECT e.*
    FROM employees e
    INNER JOIN projects p ON e.department_id = p.department_id
    WHERE e.hire_date > p.start_date;

| id  | name       | hire_date                | job_title       | department_id |
| --- | ---------- | ------------------------ | --------------- | ------------- |
| 6   | Dave Davis | 2023-03-15T00:00:00.000Z | Sales Associate | 3             |

---
**Query #4**

    SELECT department_id, name, hire_date,
           RANK() OVER (PARTITION BY department_id ORDER BY hire_date) AS hire_rank
    FROM employees;

| department_id | name          | hire_date                | hire_rank |
| ------------- | ------------- | ------------------------ | --------- |
| 1             | John Doe      | 2018-06-20T00:00:00.000Z | 1         |
| 1             | Bob Miller    | 2021-04-30T00:00:00.000Z | 2         |
| 2             | Jane Smith    | 2019-07-15T00:00:00.000Z | 1         |
| 2             | Charlie Brown | 2022-10-01T00:00:00.000Z | 2         |
| 3             | Alice Johnson | 2020-01-10T00:00:00.000Z | 1         |
| 3             | Dave Davis    | 2023-03-15T00:00:00.000Z | 2         |

---
**Query #5**

    SELECT e1.name, e1.hire_date, e2.name, e2.hire_date,
           (e2.hire_date - e1.hire_date) AS duration
    FROM employees e1
    INNER JOIN employees e2 ON e1.department_id = e2.department_id
    WHERE e2.hire_date > e1.hire_date
    ORDER BY e1.department_id, e1.hire_date;

| name          | hire_date                | name          | hire_date                | duration |
| ------------- | ------------------------ | ------------- | ------------------------ | -------- |
| John Doe      | 2018-06-20T00:00:00.000Z | Bob Miller    | 2021-04-30T00:00:00.000Z | 1045     |
| Jane Smith    | 2019-07-15T00:00:00.000Z | Charlie Brown | 2022-10-01T00:00:00.000Z | 1174     |
| Alice Johnson | 2020-01-10T00:00:00.000Z | Dave Davis    | 2023-03-15T00:00:00.000Z | 1160     |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/xckGL9ZW73A6FWhsmPogm7/21)
