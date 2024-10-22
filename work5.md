Развернуть асинхронную реплику (можно использовать 1 ВМ, просто рядом кластер развернуть и подключиться через localhost):
❖ тестируем производительность по сравнению с сингл инстансом


1. Поднял docker с postgres

2. Подключился к докер контенейру через терминал

```
docker exec -it 44d7e0946398 /bin/bash
```

3. Создал основной кластер

```bash
root@44d7e0946398:/# pg_createcluster 16 main
Creating new PostgreSQL cluster 16/main ...
```

4. Залил БД в main кластер
```bash

root@44d7e0946398:/# sudo su postgres

postgres@44d7e0946398:/$ wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql -U postgres < thai.sql
SET
SET
# ...
# Очень большой лог изменений
```

5. Создадим пользовтеля для репликации (копипаст из лекции)
```bash
postgres@44d7e0946398:/$ psql -c "CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'pass';"
CREATE ROLE
```

6. Cоздадим слот для устойчивости (копипаст из лекции)
```bash
postgres@44d7e0946398:/$ psql -p 5433 -c "SELECT pg_create_physical_replication_slot('test');"
 pg_create_physical_replication_slot 
-------------------------------------
 (test,)
(1 row)
```

7. Создадим второй кластер 

```bash
postgres@44d7e0946398:/$ pg_createcluster 16 main2
Creating new PostgreSQL cluster 16/main2 ...
16  main2   5434 down   postgres /var/lib/postgresql/16/main2 /var/log/postgresql/postgresql-16-main2.log

postgres@44d7e0946398:/$ rm -rf /var/lib/postgresql/16/main2

postgres@44d7e0946398:/$ pg_basebackup -h localhost -p 5433 -U replicator -R -S test -D /var/lib/postgresql/16/main2

# убеджаемся что создался в режиме recovery
postgres@44d7e0946398:/$ pg_lsclusters 
Ver Cluster Port Status        Owner    Data directory               Log file
16  main    5433 online        postgres /var/lib/postgresql/16/main  /var/log/postgresql/postgresql-16-main.log
16  main2   5434 down,recovery postgres /var/lib/postgresql/16/main2 /var/log/postgresql/postgresql-16-main2.log

postgres@44d7e0946398:/$ pg_ctlcluster 16 main2 start
```

8. Тестируем нагрузку на запись
``` bash
cat > ~/workload2.sql << EOL
INSERT INTO book.tickets (fkRide, fio, contact, fkSeat)
VALUES (
	ceil(random()*100)
	, (array(SELECT fam FROM book.fam))[ceil(random()*110)]::text || ' ' ||
    (array(SELECT nam FROM book.nam))[ceil(random()*110)]::text
    ,('{"phone":"+7' || (1000000000::bigint + floor(random()*9000000000)::bigint)::text || '"}')::jsonb
    , ceil(random()*100));

EOL

postgres@44d7e0946398:/$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5433 thai
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
transaction type: /var/lib/postgresql/workload2.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 67225
number of failed transactions: 0 (0.000%)
latency average = 1.189 ms
initial connection time = 10.536 ms
tps = 6725.698742 (without initial connection time)
```

9. Тестируем нагрузку на чтение
``` bash
postgres@44d7e0946398:/$ cat > ~/workload.sql << EOL

\set r random(1, 5000000) 
SELECT id, fkRide, fio, contact, fkSeat FROM book.tickets WHERE id = :r;

EOL

postgres@44d7e0946398:/$ /usr/lib/postgresql/16/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 5433 thai
pgbench (16.4 (Debian 16.4-1.pgdg120+2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 508291
number of failed transactions: 0 (0.000%)
latency average = 0.157 ms
initial connection time = 14.052 ms
tps = 50884.131188 (without initial connection time)

```