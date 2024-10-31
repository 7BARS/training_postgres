1. Развернуть ВМ (Linux) с PostgreSQL

    ✅
2. Залить Тайские перевозки
https://github.com/aeuge/postgres16book/tree/main/database

    ✅
3. Проверить скорость выполнения сложного запроса (приложен в конце файла скриптов)
    ```sql
    thai=# EXPLAIN ANALYZE WITH all_place AS (
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
                                                                                                     QUERY  PLAN                                                                                                            
    ------------------------------------------------------------------------------------------------------------------------    -------------------------------------------------------------------------------------
     Limit  (cost=329031.23..329031.25 rows=10 width=56) (actual time=671.781..671.838 rows=10 loops=1)
       ->  Sort  (cost=329031.23..329391.19 rows=143984 width=56) (actual time=658.748..658.805 rows=10 loops=1)
             Sort Key: r.startdate
             Sort Method: top-N heapsort  Memory: 25kB
             ->  Group  (cost=323400.06..325919.78 rows=143984 width=56) (actual time=624.540..647.930 rows=144000 loops=1)
                   Group Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
                   ->  Sort  (cost=323400.06..323760.02 rows=143984 width=56) (actual time=624.517..632.185 rows=144000     loops=1)
                         Sort Key: r.id, (((bs.city || ', '::text) || bs.name)), (count(t.id)), (count(s_1.id))
                         Sort Method: external merge  Disk: 7872kB
                         ->  Hash Join  (cost=257338.83..306139.34 rows=143984 width=56) (actual time=429.820..607.946  rows=144000 loops=1)
                               Hash Cond: (r.fkbus = s_1.fkbus)
                               ->  Nested Loop  (cost=257333.71..304715.98 rows=143984 width=84) (actual time=429.768..588. 996 rows=144000 loops=1)
                                     ->  Hash Join  (cost=257333.57..301289.39 rows=143984 width=24) (actual time=429.746.. 561.816 rows=144000 loops=1)
                                           Hash Cond: (s.fkroute = br.id)
                                           ->  Hash Join  (cost=257331.22..300882.38 rows=143984 width=24) (actual time=429.    731..548.302 rows=144000 loops=1)
                                                 Hash Cond: (r.fkschedule = s.id)
                                                 ->  Merge Join  (cost=257287.82..300459.91 rows=143984 width=24) (actual   time=429.585..532.517 rows=144000 loops=1)
                                                       Merge Cond: (r.id = t.fkride)
                                                       ->  Index Scan using ride_pkey on ride r  (cost=0.42..4534.42    rows=144000 width=16) (actual time=0.008..11.326 rows=144000 loops=1)
                                                       ->  Finalize GroupAggregate  (cost=257287.40..293765.69 rows=143984  width=12) (actual time=429.572..501.225 rows=144000 loops=1)
                                                             Group Key: t.fkride
                                                             ->  Gather Merge  (cost=257287.40..290886.01 rows=287968   width=12) (actual time=429.540..470.736 rows=432000 loops=1)
                                                                   Workers Planned: 2
                                                                   Workers Launched: 2
                                                                   ->  Sort  (cost=256287.38..256647.34 rows=143984     width=12) (actual time=420.205..428.196 rows=144000     loops=3)
                                                                         Sort Key: t.fkride
                                                                         Sort Method: external merge  Disk: 3672kB
                                                                         Worker 0:  Sort Method: external merge  Disk:  3672kB
                                                                         Worker 1:  Sort Method: external merge  Disk:  3672kB
                                                                         ->  Partial HashAggregate  (cost=218947.44..241487.    15 rows=143984 width=12) (actual time=331.925..399. 747 rows=144000 loops=3)
                                                                               Group Key: t.fkride
                                                                               Planned Partitions: 4  Batches: 5  Memory    Usage: 8241kB  Disk Usage: 24336kB
                                                                               Worker 0:  Batches: 5  Memory Usage: 8241kB      Disk Usage: 27608kB
                                                                               Worker 1:  Batches: 5  Memory Usage: 8241kB      Disk Usage: 27568kB
                                                                               ->  Parallel Seq Scan on tickets t  (cost=0. 00..80532.27 rows=2160627 width=12) (actual  time=0.020..103.472 rows=1728502 loops=3)
                                                 ->  Hash  (cost=25.40..25.40 rows=1440 width=8) (actual time=0.140..0.141  rows=1440 loops=1)
                                                       Buckets: 2048  Batches: 1  Memory Usage: 73kB
                                                       ->  Seq Scan on schedule s  (cost=0.00..25.40 rows=1440 width=8)     (actual time=0.003..0.068 rows=1440 loops=1)
                                           ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.010..0.011 rows=60    loops=1)
                                                 Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                                 ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual     time=0.004..0.006 rows=60 loops=1)
                                     ->  Memoize  (cost=0.15..0.36 rows=1 width=68) (actual time=0.000..0.000 rows=1    loops=144000)
                                           Cache Key: br.fkbusstationfrom
                                           Cache Mode: logical
                                           Hits: 143990  Misses: 10  Evictions: 0  Overflows: 0  Memory Usage: 2kB
                                           ->  Index Scan using busstation_pkey on busstation bs  (cost=0.14..0.35 rows=1   width=68) (actual time=0.002..0.002 rows=1 loops=10)
                                                 Index Cond: (id = br.fkbusstationfrom)
                               ->  Hash  (cost=5.05..5.05 rows=5 width=12) (actual time=0.045..0.046 rows=5 loops=1)
                                     Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                     ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=0.042..0.043 rows=5  loops=1)
                                           Group Key: s_1.fkbus
                                           Batches: 1  Memory Usage: 24kB
                                           ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=0.011..    0.019 rows=200 loops=1)
     Planning Time: 0.348 ms
     JIT:
       Functions: 84
       Options: Inlining false, Optimization false, Expressions true, Deforming true
       Timing: Generation 1.448 ms, Inlining 0.000 ms, Optimization 1.089 ms, Emission 23.248 ms, Total 25.785 ms
     Execution Time: 674.913 ms
    (59 rows)
    ```
4. Навесить индексы на внешние ключ
    ```sql
    thai=# CREATE INDEX idx_seat_fkbus ON book.seat (fkbus);
    CREATE INDEX
    thai=# CREATE INDEX idx_tickets_fkride ON book.tickets (fkride);
    CREATE INDEX
    thai=# CREATE INDEX idx_ride_fkschedule ON book.ride (fkschedule);
    CREATE INDEX
    thai=# CREATE INDEX idx_schedule_fkroute ON book.schedule (fkroute);
    CREATE INDEX
    thai=# CREATE INDEX idx_busroute_fkbusstationfrom ON book.busroute (fkbusstationfrom);
    CREATE INDEX
    ```
5. Проверить, помогли ли индексы на внешние ключи ускориться
    ```text
    В данном запросе индексы не помогают, из-за ORDER BY r.startdate, приходится перечитывать всю строки
    ```