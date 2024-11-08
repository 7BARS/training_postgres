1. Создать таблицу с продажами.
    ```sql
    thai=# CREATE TABLE sales (
        sale_id INT PRIMARY KEY,
        sale_date DATE,
        amount DECIMAL(10, 2)
    );
    CREATE TABLE
    thai=# INSERT INTO sales (sale_id, sale_date, amount) VALUES
        (1, '2024-01-15', 100.00),
        (2, '2024-04-10', 150.00),
        (3, '2024-05-22', 200.00),
        (4, '2024-08-05', 250.00),
        (5, '2024-09-17', 300.00),
        (6, '2024-12-25', 350.00);
    ```
2. Реализовать функцию выбор трети года (1-4 месяц - первая треть, 5-8 - вторая и тд)
    ```sql
    thai=# CREATE OR REPLACE FUNCTION get_year_part_case(sale_date DATE) 
    RETURNS INT AS $$
    BEGIN
        RETURN CASE 
            WHEN sale_date IS NULL THEN NULL
            WHEN EXTRACT(MONTH FROM sale_date) BETWEEN 1 AND 4 THEN 1
            WHEN EXTRACT(MONTH FROM sale_date) BETWEEN 5 AND 8 THEN 2
            WHEN EXTRACT(MONTH FROM sale_date) BETWEEN 9 AND 12 THEN 3
            ELSE NULL
        END;
    END;
    $$ LANGUAGE plpgsql;
    ```
3. Вызвать эту функцию в SELECT из таблицы с продажами, уведиться, что всё отработало
    ```sql
    thai=# select *, get_year_part_case(sale_date) from sales;
     sale_id | sale_date  | amount | get_year_part_case 
    ---------+------------+--------+--------------------
           1 | 2024-01-15 | 100.00 |                  1
           2 | 2024-04-10 | 150.00 |                  1
           3 | 2024-05-22 | 200.00 |                  2
           4 | 2024-08-05 | 250.00 |                  2
           5 | 2024-09-17 | 300.00 |                  3
           6 | 2024-12-25 | 350.00 |                  3
    (6 rows)
    ```
