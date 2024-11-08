1. Сгенерировать таблицу с 1 млн JSONB документов
    ```sql
    thai=# CREATE TABLE json_data (
        id SERIAL PRIMARY KEY,
        data JSONB
    );
    CREATE TABLE
    thai=# 
    thai=# INSERT INTO json_data (data)
    SELECT jsonb_build_object(
        'name', md5(random()::text),
        'age', (random() * 100)::int,
        'city', md5(random()::text)
    )
    FROM generate_series(1, 1000000);
    INSERT 0 1000000
    ```
2. Создать индекс
    ```sql
    thai=# CREATE INDEX idx_json_data_name ON json_data ((data->>'name'));
    CREATE INDEX
    ```
3. Обновить 1 из полей в json
    ```sql
    thai=# UPDATE json_data
    SET data = jsonb_set(data, '{age}', ((random() * 100)::int)::text::jsonb);
    UPDATE 1000000
    ```
4. Убедиться в блоатинге TOAST
    ```sql
    thai=# SELECT oid::regclass AS heap_rel,
           pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
           reltoastrelid::regclass AS toast_rel,
           pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
    FROM pg_class WHERE relname = 'json_data';
     heap_rel  | heap_rel_size |        toast_rel        | toast_rel_size 
    -----------+---------------+-------------------------+----------------
     json_data | 284 MB        | pg_toast.pg_toast_40982 | 0 bytes
    (1 row)
    ```
5. Придумать методы избавится от него и проверить на практике
    ```sql
    thai=# VACUUM FULL json_data;
    VACUUM
    thai=# SELECT oid::regclass AS heap_rel,
           pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
           reltoastrelid::regclass AS toast_rel,
           pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
    FROM pg_class WHERE relname = 'json_data';
     heap_rel  | heap_rel_size |        toast_rel        | toast_rel_size 
    -----------+---------------+-------------------------+----------------
     json_data | 142 MB        | pg_toast.pg_toast_40982 | 0 bytes
    (1 row)
    ```
6. Не забываем про блоатинг индексов*
    ```sql
    thai=# REINDEX TABLE json_data;
    REINDEX
    ```