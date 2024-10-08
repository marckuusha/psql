1. открыть консоль и зайти по ssh на ВМ
2. открыть вторую консоль и также зайти по ssh на ту же ВМ (можно в докере 2 сеанса)

3. запустить везде psql из под пользователя postgres

```bash
console psql % psql -h localhost -p 5432 -U postgres                                                                        
Password for user postgres: 
psql (14.11 (Homebrew), server 16.4 (Debian 16.4-1.pgdg120+2))
WARNING: psql major version 14, server major version 16.
         Some psql features might not work.
Type "help" for help.

postgres=# 
```
4. сделать в первой сессии новую таблицу и наполнить ее данными

```sql
postgres=# CREATE TABLIE t(value INTEGER);
CREATE TABLE

postgres=# insert into t(value) values (1), (2), (3);
INSERT 0 3
```

5. посмотреть текущий уровень изоляции:

```sql
postgres=# SHOW TRANSACTION ISOLATION LEVEL;
 transaction_isolation 
-----------------------
 read committed
(1 row)
```

6. начать новую транзакцию в обеих сессиях с дефолтным (не меняя) уровнень изоляции

```sql
-- 1 session
postgres=# begin;
BEGIN

-- 2 session
postgres=# begin;
BEGIN
```

7. в первой сессии добавить новую запись

```sql
-- 1 session
postgres=*# insert into t(value) values(4);
INSERT 0 1
```

8. сделать запрос на выбор всех записей во второй сессии

```sql
-- 2 session
postgres=*# select * from t;
 value 
-------
     1
     2
     3
(3 rows)
```

9. видите ли вы новую запись и если да то почему? После задания можете сверить
правильный ответ с эталонным (будет доступен после 3 лекции)

Новую запись не видно так как уровень изоляции read commited, а первая сессия еще не зафиксировала изменения.

10. завершить транзакцию в первом окне

```sql
-- 1 session
postgres=*# commit;
COMMIT
```

11. сделать запрос на выбор всех записей второй сессии


```sql
-- 2 session
postgres=*# select * from t;
 value 
-------
     1
     2
     3
     4
(4 rows)
```

12. видите ли вы новую запись и если да то почему?

Теперь данные видны, 1 транзакция зафиксировала изменения. 

13. завершите транзакцию во второй сессии

```sql
-- 2 session
postgres=*# commit;
COMMIT
```

14. начать новые транзакции, но уже на уровне repeatable read в ОБЕИХ сессиях

```sql
-- 1 session
postgres=# BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN

-- 2 session
postgres=# BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN
```

15. в первой сессии добавить новую запись

```sql
-- 1 session
postgres=*# insert into t(value) values(5);
INSERT 0 1
```

16. сделать запрос на выбор всех записей во второй сессии

```sql
-- 2 session
postgres=*# select * from t;
 value 
-------
     1
     2
     3
     4
(4 rows)
```

17. видите ли вы новую запись и если да то почему?

Новую запись не видно, так как уровень изоляции repeatable read. Снимок данных строится на момент создании транзакции.

18. завершить транзакцию в первом окне


```sql
-- 1 session
postgres=*# commit;
COMMIT
```

19. сделать запрос во выбор всех записей второй сессии

```sql
-- 2 session
postgres=*# select * from t;
 value 
-------
     1
     2
     3
     4
(4 rows)
```

20. видите ли вы новую запись и если да то почему?

Новую запись не видно, так как уровень изоляции repeatable read. Снимок данных строится на момент создании транзакции.