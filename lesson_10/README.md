1. Установить 16 ПГ.

    В качестве основы был взят докер образ с ubuntu:24
2. Залить средние Тайские перевозки

    ✅
3. Рядом поднять кластер 17 версии
    ```bash
    root@1109d6521da4:/# pg_lsclusters 
    Ver Cluster Port Status Owner    Data directory              Log file
    16  main    5432 online postgres /var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log
    17  main    5433 online postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
    ```
4. Протестировать скорость онлайн вариантов миграции (логическая репликация, postgres_fdw, pg_dump/pg_restore)
5. Один минимум, лучше 2+

    В качестве нагрузки взял запрос с join
    1. На 16 ПГ
    ```sql
    thai=# WITH all_place AS (
        SELECT count(s.id) as all_place, s.fkbus as fkbus
        FROM book.seat s
        group by s.fkbus
    ),
    order_place AS (
        SELECT count(t.id) as order_place, t.fkride
        FROM book.tickets t
        group by t.fkride
    )
    SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,  
          t.order_place, st.all_place
    FROM book.ride r
    JOIN book.schedule as s
          on r.fkschedule = s.id
    JOIN book.busroute br
          on s.fkroute = br.id
    JOIN book.busstation bs
          on br.fkbusstationfrom = bs.id
    JOIN order_place t
          on t.fkride = r.id
    JOIN all_place st
          on r.fkbus = st.fkbus
    GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
    ORDER BY r.startdate
    limit 10;
      id  | depart_date |      busstation       | order_place | all_place 
    ------+-------------+-----------------------+-------------+-----------
     1006 | 2000-01-01  | Poipet, Train station |          35 |        40
     1034 | 2000-01-01  | Bankgkok, Eastern     |          37 |        40
      329 | 2000-01-01  | Pattaya, Central      |          35 |        40
      207 | 2000-01-01  | Pattaya, North        |          37 |        40
      825 | 2000-01-01  | Surin, Central        |          35 |        40
     1394 | 2000-01-01  | Bankgkok, Eastern     |          38 |        40
      891 | 2000-01-01  | Changmai, Central     |          35 |        40
      791 | 2000-01-01  | Bankgkok, Eastern     |          37 |        40
      631 | 2000-01-01  | Pattaya, South        |          35 |        40
      575 | 2000-01-01  | Pattaya, Central      |          36 |        40
    (10 rows)

    Time: 7891.165 ms (00:07.891)
    ```
    2. На 17 ПГ Логическая репликация
    ```sql
    thai=# WITH all_place AS (
        SELECT count(s.id) as all_place, s.fkbus as fkbus
        FROM book.seat s
        group by s.fkbus
    ),
    order_place AS (
        SELECT count(t.id) as order_place, t.fkride
        FROM book.tickets t
        group by t.fkride
    )
    SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,  
          t.order_place, st.all_place
    FROM book.ride r
    JOIN book.schedule as s
          on r.fkschedule = s.id
    JOIN book.busroute br
          on s.fkroute = br.id
    JOIN book.busstation bs
          on br.fkbusstationfrom = bs.id
    JOIN order_place t
          on t.fkride = r.id
    JOIN all_place st
          on r.fkbus = st.fkbus
    GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
    ORDER BY r.startdate
    limit 10;
     id | depart_date |       busstation       | order_place | all_place 
    ----+-------------+------------------------+-------------+-----------
      2 | 2000-01-01  | Bankgkok, Suvarnabhumi |          36 |        40
      3 | 2000-01-01  | Bankgkok, Suvarnabhumi |          35 |        40
      4 | 2000-01-01  | Bankgkok, Eastern      |          35 |        40
      5 | 2000-01-01  | Bankgkok, Eastern      |          36 |        40
      6 | 2000-01-01  | Bankgkok, Eastern      |          37 |        40
      7 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        40
      8 | 2000-01-01  | Bankgkok, Chatuchak    |          39 |        40
      9 | 2000-01-01  | Bankgkok, Chatuchak    |          34 |        40
     10 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
      1 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        40
    (10 rows)
    
    Time: 10382.376 ms (00:10.382)
    ```
    3. На 17 ПГ postgres_fdw
    ```sql
    thai=# WITH all_place AS (
        SELECT count(s.id) as all_place, s.fkbus as fkbus
        FROM book.seat s
        group by s.fkbus
    ),
    order_place AS (
        SELECT count(t.id) as order_place, t.fkride
        FROM book.tickets t
        group by t.fkride
    )
    SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,  
          t.order_place, st.all_place
    FROM book.ride r
    JOIN book.schedule as s
          on r.fkschedule = s.id
    JOIN book.busroute br
          on s.fkroute = br.id
    JOIN book.busstation bs
          on br.fkbusstationfrom = bs.id
    JOIN order_place t
          on t.fkride = r.id
    JOIN all_place st
          on r.fkbus = st.fkbus
    GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
    ORDER BY r.startdate
    limit 10;
      id  | depart_date |       busstation       | order_place | all_place 
    ------+-------------+------------------------+-------------+-----------
      183 | 2000-01-01  | Bankgkok, Suvarnabhumi |          39 |        40
      721 | 2000-01-01  | Bankgkok, Suvarnabhumi |          36 |        40
      303 | 2000-01-01  | Bankgkok, Suvarnabhumi |          35 |        40
      602 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        40
     1141 | 2000-01-01  | Bankgkok, Suvarnabhumi |          37 |        40
     1381 | 2000-01-01  | Bankgkok, Suvarnabhumi |          39 |        40
     1263 | 2000-01-01  | Bankgkok, Suvarnabhumi |          35 |        40
      243 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        40
     1262 | 2000-01-01  | Bankgkok, Suvarnabhumi |          36 |        40
       61 | 2000-01-01  | Bankgkok, Suvarnabhumi |          36 |        40
    (10 rows)

    Time: 16059.624 ms (00:16.060)
    ```