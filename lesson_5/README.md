Развернуть асинхронную реплику (можно использовать 1 ВМ, просто рядом кластер
развернуть и подключиться через localhost):
❖ тестируем производительность по сравнению с сингл инстансом

```text
Стенд: 2 докер контейнера на базе ubuntu:20.04, внутри postgres 17
```
1. Бенч с репликой:
    ```sql
    Первый прогон:
    postgres@primary:~$ pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
    pgbench (17.0 (Ubuntu 17.0-1.pgdg20.04+1))
    transaction type: /var/lib/postgresql/workload2.sql
    scaling factor: 1
    query mode: simple
    number of clients: 8
    number of threads: 4
    maximum number of tries: 1
    duration: 10 s
    number of transactions actually processed: 89019
    number of failed transactions: 0 (0.000%)
    latency average = 0.887 ms
    initial connection time = 134.156 ms
    tps = 9020.567368 (without initial connection time)
    ```
    ```sql
    Через несколько прогонов (прогрелся кэш):
    postgres@primary:~$ pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
    pgbench (17.0 (Ubuntu 17.0-1.pgdg20.04+1))
    transaction type: /var/lib/postgresql/workload2.sql
    scaling factor: 1
    query mode: simple
    number of clients: 8
    number of threads: 4
    maximum number of tries: 1
    duration: 10 s
    number of transactions actually processed: 217723
    number of failed transactions: 0 (0.000%)
    latency average = 0.367 ms
    initial connection time = 5.326 ms
    tps = 21780.753110 (without initial connection time)
    ```
2. Бенч с отключенной репликой
    ```sql
    postgres@primary:~$ pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
    pgbench (17.0 (Ubuntu 17.0-1.pgdg20.04+1))
    transaction type: /var/lib/postgresql/workload2.sql
    scaling factor: 1
    query mode: simple
    number of clients: 8
    number of threads: 4
    maximum number of tries: 1
    duration: 10 s
    number of transactions actually processed: 252148
    number of failed transactions: 0 (0.000%)
    latency average = 0.317 ms
    initial connection time = 6.235 ms
    tps = 25228.299663 (without initial connection time)
    ```

```text
    Разница с включеной асинхронной репликой вышла в районе ~16%
```