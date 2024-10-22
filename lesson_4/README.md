1. Создать таблицу accounts(id integer, amount numeric);
    ```sql
    thai=# CREATE table accounts(id integer, amount numeric);
    CREATE TABLE
    ```
2. Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).
    ```sql
    thai=# INSERT INTO accounts VALUES(1, 123);
    INSERT 0 1
    thai=# INSERT INTO accounts VALUES(2, 456);
    INSERT 0 1
    ```
    ```sql
    1 terminal:
    thai=# BEGIN;
    BEGIN
    thai=*# UPDATE accounts SET amount = amount + 100 WHERE id = 1;
    UPDATE 1
    ```
    ```sql
    2 terminal:
    thai=# BEGIN;
    BEGIN
    thai=*# UPDATE accounts SET amount = amount + 200 WHERE id = 2;
    UPDATE 1
    ```
    ```sql
    1 terminal:
    thai=*# UPDATE accounts SET amount = amount + 200 WHERE id = 2;
    ```
    ```sql
    2 terminal:
    thai=*# UPDATE accounts SET amount = amount + 100 WHERE id = 1;
    ERROR:  deadlock detected
    DETAIL:  Process 1037 waits for ShareLock on transaction 1041; blocked by process 78.
    Process 78 waits for ShareLock on transaction 1042; blocked by process 1037.
    HINT:  See server log for query details.
    CONTEXT:  while updating tuple (0,1) in relation "accounts"
    ```
3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.
    ```log
    2024-10-22 16:21:42.308 MSK [1037] postgres@thai ERROR:  relation "accounts" does not exist at character 8
    2024-10-22 16:21:42.308 MSK [1037] postgres@thai STATEMENT:  UPDATE accounts SET amount = amount + 200 WHERE id = 2;
    2024-10-22 16:23:08.866 MSK [1037] postgres@thai ERROR:  deadlock detected
    2024-10-22 16:23:08.866 MSK [1037] postgres@thai DETAIL:  Process 1037 waits for ShareLock on transaction 1041; blocked     by process 78.
    	Process 78 waits for ShareLock on transaction 1042; blocked by process 1037.
    	Process 1037: UPDATE accounts SET amount = amount + 100 WHERE id = 1;
    	Process 78: UPDATE accounts SET amount = amount + 200 WHERE id = 2;
    2024-10-22 16:23:08.866 MSK [1037] postgres@thai HINT:  See server log for query details.
    ```