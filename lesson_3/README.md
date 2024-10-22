1. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1 млн строк
    ```sql
    thai=# CREATE TABLE note (
        record   TEXT
    );
    CREATE TABLE
    thai=# INSERT INTO note SELECT i::TEXT
    FROM generate_series(1,1000000) s(i);
    INSERT 0 1000000
    thai=# SELECT count(*) FROM note;
      count
    ---------
     1000000
    (1 row)
    ```
2. Посмотреть размер файла с таблицей
    ```sql
    thai=# SELECT oid FROM pg_class WHERE relname = 'note';
      oid
    -------
     16586
    (1 row)

    thai=# SELECT oid FROM pg_database WHERE datname = 'thai';
      oid
    -------
     16388
    (1 row)

    thai=# SHOW data_directory;
           data_directory
    -----------------------------
     /var/lib/postgresql/16/main
    (1 row)

    thai=# \! ls -lh /var/lib/postgresql/16/main/base/16388/16586
    -rw------- 1 postgres postgres 35M Oct 15 18:51 /var/lib/postgresql/16/main/base/16388/16586
    ```
3. 5 раз обновить все строчки и добавить к каждой строчке любой символ
    ```sql
    thai=# UPDATE note SET record = record || '1';
    UPDATE note SET record = record || '2';
    UPDATE note SET record = record || '3';
    UPDATE note SET record = record || '4';
    UPDATE note SET record = record || '5';
    UPDATE 1000000
    UPDATE 1000000
    UPDATE 1000000
    UPDATE 1000000
    UPDATE 1000000
    thai=# select * from note limit 1;
     record 
    --------
     112345
    (1 row)
    ```
4. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил
автовакуум
    ```sql
    thai=# select n_live_tup, n_dead_tup, relname from pg_stat_all_tables where relname = 'note';
     n_live_tup | n_dead_tup | relname 
    ------------+------------+---------
              0 |    4999788 | note
    (1 row)

    select relname, last_vacuum, last_autovacuum, last_analyze, last_autoanalyze from pg_stat_user_tables where relname = 'note';
     relname | last_vacuum |        last_autovacuum        | last_analyze |       last_autoanalyze        
    ---------+-------------+-------------------------------+--------------+-------------------------------
     note    |             | 2024-10-22 15:18:38.672917+03 |              | 2024-10-22 15:18:39.727385+03
    (1 row)
    ```
5. Подождать некоторое время, проверяя, пришел ли автовакуум
    ```sql
    thai=# select n_live_tup, n_dead_tup, relname from pg_stat_all_tables where relname = 'note';
     n_live_tup | n_dead_tup | relname 
    ------------+------------+---------
         995207 |          0 | note
    (1 row)
    ```
6. 5 раз обновить все строчки и добавить к каждой строчке любой символ

    ✅
7. Посмотреть размер файла с таблицей
    ```bash
    \! ls -lh /var/lib/postgresql/16/main/base/16388/16586
    -rw------- 1 postgres postgres 456M Oct 22 15:21 /var/lib/postgresql/16/main/base/16388/16586
    ```
8. Отключить Автовакуум на конкретной таблице
    ```sql
    ALTER TABLE note SET (autovacuum_enabled = false);
    ALTER TABLE
    ```
9. 10 раз обновить все строчки и добавить к каждой строчке любой символ
    ```sql
    DO $$
    BEGIN
       for i in 1..10 LOOP
          UPDATE note SET record = record || i::TEXT;
       END LOOP;
    END $$;
    DO
    ```
10. Посмотреть размер файла с таблицей
    ```sql
    thai=# \! ls -lh /var/lib/postgresql/16/main/base/16388/16586
    -rw------- 1 postgres postgres 615M Oct 22 15:47 /var/lib/postgresql/16/main/base/16388/16586
    ```
11. Объясните полученный результат
    ```text
    Скопилось много не удаленых таплов
    ```
    ```sql
    thai=# select n_live_tup, n_dead_tup, relname from pg_stat_all_tables where relname = 'note';
     n_live_tup | n_dead_tup | relname 
    ------------+------------+---------
         998508 |   10000000 | note
    (1 row)
    ```
12. Не забудьте включить автовакуум)
    ```sql
    ALTER TABLE note SET (autovacuum_enabled = true);
    ALTER TABLE
    ```

Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла.