1. открыть консоль и зайти по ssh на ВМ

    ✅
2. открыть вторую консоль и также зайти по ssh на ту же ВМ (можно в докере 2 сеанса)

    ✅
3. запустить везде psql из под пользователя postgres

    ✅
4. сделать в первой сессии новую таблицу и наполнить ее данными
    ```sql
    thai=# SELECT current_database();
    CREATE TABLE test (i serial, amount int);
    INSERT INTO test(amount) VALUES (1000);
    INSERT INTO test(amount) VALUES (2000);
    SELECT * FROM test;
     current_database
    ------------------
     thai
    (1 row)

    CREATE TABLE
    INSERT 0 1
    INSERT 0 1
     i | amount
    ---+--------
     1 |   1000
     2 |   2000
    (2 rows)
    ```
5. посмотреть текущий уровень изоляции:
    ```sql
    thai=# show transaction isolation level;
     transaction_isolation
    -----------------------
     read committed
    (1 row)
    ```
6. начать новую транзакцию в обеих сессиях с дефолтным (не меняя) уровнем
изоляции
    ```sql
    thai=# BEGIN;
    BEGIN
    thai=*#
    ```
7. в первой сессии добавить новую запись
    ```sql
    thai=*# INSERT INTO test(amount) VALUES (100500);
    INSERT 0 1
    ```
8. сделать запрос на выбор всех записей во второй сессии
    ```sql
    thai=# BEGIN;
    BEGIN
    thai=*# SELECT * FROM test;
     i | amount
    ---+--------
     1 |   1000
     2 |   2000
    (2 rows)
    ```
9. видите ли вы новую запись и если да то почему? После задания можете сверить
правильный ответ с эталонным (будет доступен после 3 лекции)
    ```text
    Записи не видно в другой транзакции, потому что при уровне изоляции read committed, видно только записи с успешно завершеных транзакций.
    ```
10. завершить транзакцию в первом окне
    ```sql
    thai=*# COMMIT;
    COMMIT
    ```
11. сделать запрос на выбор всех записей второй сессии
    ```sql
    thai=*# SELECT * FROM test;
     i | amount
    ---+--------
     1 |   1000
     2 |   2000
     3 | 100500
    (3 rows)
    ```
12. видите ли вы новую запись и если да то почему?
    ```text
    Запись появилась, потому что предыдушая транзакиция завершилась успешно и все ее изменения были записаны в БД
    ```
13. завершите транзакцию во второй сессии
    ```sql
    thai=*# COMMIT;
    COMMIT
    ```
14. начать новые транзакции, но уже на уровне repeatable read в ОБЕИХ сессиях
    ```sql
    thai=# BEGIN;
    BEGIN
    thai=*# set transaction isolation level repeatable read;
    SET
    thai=*# show transaction isolation level;
     transaction_isolation
    -----------------------
     repeatable read
    (1 row)
    ```
15. в первой сессии добавить новую запись
    ```sql
    thai=*# INSERT INTO test(amount) VALUES (808);
    INSERT 0 1
    ```
16. сделать запрос на выбор всех записей во второй сессии
    ```sql
    thai=*# select * from test;
     i | amount
    ---+--------
     1 |   1000
     2 |   2000
     3 | 100500
    (3 rows)
    ```
17. видите ли вы новую запись и если да то почему?
    ```text
    Записи не видно в другой транзакции, потому что при уровне изоляции repeatable read, видно только записи на момент начала транзакции, видно слепок данных.
    ```
18. завершить транзакцию в первом окне
    ```sql
    thai=*# COMMIT;
    COMMIT
    ```
19. сделать запрос во выбор всех записей второй сессии
    ```sql
    thai=*# select * from test;
     i | amount
    ---+--------
     1 |   1000
     2 |   2000
     3 | 100500
    (3 rows)
    ```
20. видите ли вы новую запись и если да то почему?
    ```text
    Нет, потому что при уровне изоляции repeatable read вижно только слепок данных на момент старта транзакции
    ```
