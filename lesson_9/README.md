1. Проанализировать данные о зарплатах сотрудников с использованием оконных функций.
а) На сколько было увеличение с предыдущей зарплатой
б) если это первая зарплата - вместо NULL вывести 0
    https://www.db-fiddle.com/f/eQ8zNtAFY88i8nB4GRB65V/0
    ```sql
    SELECT
        e.id AS employee_id,
        e.first_name,
        e.last_name,
        s.from_date,
        s.to_date,
        s.amount AS current_salary,
        COALESCE(s.amount - LAG(s.amount) OVER (PARTITION BY s.fk_employee ORDER BY s.from_date), 0) AS salary_increase,
        g.value AS grade
    FROM
        salary s
    JOIN
        employee e ON s.fk_employee = e.id
    JOIN
        grade g ON s.fk_grade = g.id
    ORDER BY
        e.id, s.from_date;
    ```
    result:
    ---
    | employee_id | first_name | last_name | from_date  | to_date    | current_salary | salary_increase | grade  |
    | ----------- | ---------- | --------- | ---------- | ---------- | -------------- | --------------- | ------ |
    | 1           | Eugene     | Aristov   | 2024-01-01 | 2024-01-31 | 100000         | 0               | junior |
    | 1           | Eugene     | Aristov   | 2024-02-01 | 2024-02-29 | 200000         | 100000          | middle |
    | 1           | Eugene     | Aristov   | 2024-03-01 | 2099-12-31 | 300000         | 100000          | senoir |
    | 2           | Ivan       | Ivanov    | 2023-01-01 | 2024-01-31 | 200000         | 0               | middle |
    | 3           | Petr       | Petrov    | 2024-03-01 | 2024-01-31 | 200000         | 0               | middle |
    ---