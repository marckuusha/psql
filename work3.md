1. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными
данным в размере 1 млн строк

```sql
postgres=# CREATE TABLE test(s text);
CREATE TABLE

postgres=# INSERT INTO test(s)
SELECT md5(random()::text) FROM generate_series(1,1000000);
INSERT 0 1000000
```

2. Посмотреть размер файла с таблицей

```sql
postgres=# SELECT pg_size_pretty(pg_table_size('test')) AS table_size;
 table_size 
------------
 65 MB
(1 row)

```

3. 5 раз обновить все строчки и добавить к каждой строчке любой символ

```sql
postgres=# DO $$
BEGIN 
    FOR i IN 1..5 LOOP
        UPDATE test SET s = s || 'a';
    END LOOP;
END $$;
DO
```
4. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил
автовакуум

```sql
postgres=# 
SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum 
FROM pg_stat_user_tables WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 test    |    1000000 |    5000000 |    499 | 2024-10-10 18:37:33.827281+00
(1 row)
```

5. Подождать некоторое время, проверяя, пришел ли автовакуум

```sql
postgres=#                                                                                                              SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum 
FROM pg_stat_user_tables WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum        
---------+------------+------------+--------+-------------------------------
 test    |    1002800 |          0 |      0 | 2024-10-10 18:38:50.181958+00
(1 row)
```

6. 5 раз обновить все строчки и добавить к каждой строчке любой символ

```sql
postgres=# DO $$
BEGIN 
    FOR i IN 1..5 LOOP
        UPDATE test SET s = s || 'a';
    END LOOP;
END $$;
DO
```

7. Посмотреть размер файла с таблицей

```sql
postgres=# SELECT pg_size_pretty(pg_table_size('test')) AS table_size;
 table_size 
------------
 415 MB
(1 row)
```

8. Отключить Автовакуум на конкретной таблице

```sql
postgres=# ALTER SYSTEM SET autovacuum = off;
ALTER SYSTEM
```

9. 10 раз обновить все строчки и добавить к каждой строчке любой символ

```sql
postgres=# DO $$
BEGIN 
    FOR i IN 1..10 LOOP
        UPDATE test SET s = s || 'b';
    END LOOP;
END $$;
DO
```

10. Посмотреть размер файла с таблицей

```sql
postgres=# SELECT pg_size_pretty(pg_table_size('test')) AS table_size;
 table_size 
------------
 841 MB
(1 row)
```

11. Объясните полученный результат

После отключения автовакуума и выполнения 10 дополнительных обновлений размер таблицы вырос. При каждом обновлении строки создаются новые версии строки, а мертвые строки не удаляются, так как автовакуум отключен.


12. Не забудьте включить автовакуум

```sql
postgres=# ALTER SYSTEM SET autovacuum = on;
ALTER SYSTEM
```