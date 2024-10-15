1. Создать таблицу accounts(id integer, amount numeric);

```sql
postgres=# CREATE TABLE accounts(id integer, amount numeric);
CREATE TABLE
```

2. Добавить несколько зuаписей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).

```sql
INSERT INTO accounts(id, amount) values(1, 100), (2, 200), (3,300);
INSERT 0 3

-- tr1
postgres=# begin;
BEGIN
-- В первой транзе сперва обновили строку с id = 1
postgres=*# update accounts set amount = amount + 100 where id = 1;
UPDATE 1
-- В первой транзе обновили строку с id = 2 (после обновления во второй) и заблоичилсь
postgres=*# update accounts set amount = amount + 100 where id = 2;
UPDATE 1
postgres=*# 

--tr2
postgres=# begin;
BEGIN
-- Во второй транзе обновили строку с id = 2
postgres=*# update accounts set amount = amount + 100 where id = 2;
UPDATE 1
postgres=*# update accounts set amount = amount + 100 where id = 1;
-- Во второй транзе обновили строку с id = 1 и поймил дедлок
postgres=*# update accounts set amount = amount + 100 where id = 1;
ERROR:  deadlock detected
DETAIL:  Process 7491 waits for ShareLock on transaction 786; blocked by process 7469.
Process 7469 waits for ShareLock on transaction 787; blocked by process 7491.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,1) in relation "accounts"

```

3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.

```bash
# Лог postgresql в докере
2024-10-15 20:42:50 2024-10-15 17:42:50.923 UTC [7491] ERROR:  deadlock detected
2024-10-15 20:42:50 2024-10-15 17:42:50.923 UTC [7491] DETAIL:  Process 7491 waits for ShareLock on transaction 786; blocked by process 7469.
2024-10-15 20:42:50     Process 7469 waits for ShareLock on transaction 787; blocked by process 7491.
2024-10-15 20:42:50     Process 7491: update accounts set amount = amount + 100 where id = 1;
2024-10-15 20:42:50     Process 7469: update accounts set amount = amount + 100 where id = 2;
2024-10-15 20:42:50 2024-10-15 17:42:50.923 UTC [7491] HINT:  See server log for query details.
2024-10-15 20:42:50 2024-10-15 17:42:50.923 UTC [7491] CONTEXT:  while updating tuple (0,1) in relation "accounts"
2024-10-15 20:42:50 2024-10-15 17:42:50.923 UTC [7491] STATEMENT:  update accounts set amount = amount + 100 where id = 1;
```