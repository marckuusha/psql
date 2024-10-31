
1. Сгенерировать таблицу с 1 млн JSONB документов

```sql
-- Создаем таблице с 1млн записей
# CREATE TABLE t AS
SELECT i AS id, (SELECT jsonb_object_agg(j, j) FROM generate_series(1, 400) j) js
FROM generate_series(1, 1000000) i;
CREATE TABLE

-- проверяем размер таблицы и тоаст таблицы
# SELECT oid::regclass AS heap_rel,
       pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
       reltoastrelid::regclass AS toast_rel,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
FROM pg_class WHERE relname = 't';
 heap_rel | heap_rel_size |         toast_rel         | toast_rel_size 
----------+---------------+---------------------------+----------------
 t        | 50 MB         | pg_toast.pg_toast_2514675 | 2604 MB
(1 row)
```

2. Создать индекс

```sql
-- вешаем индекс на поле типа jsonb
# CREATE INDEX idx_t ON t USING gin(js);
CREATE INDEX
```

3. Обновить 1 из полей в json

```sql
# UPDATE t SET js = js::jsonb || '{"a":1}';
UPDATE 1000000
```

4. Убедиться в блоатинге TOAST

```sql
-- смотрим, что после апдейта, размер тоаст увеличился в x2
# SELECT oid::regclass AS heap_rel,
       pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
       reltoastrelid::regclass AS toast_rel,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
FROM pg_class WHERE relname = 't';
 heap_rel | heap_rel_size |         toast_rel         | toast_rel_size 
----------+---------------+---------------------------+----------------
 t        | 100 MB        | pg_toast.pg_toast_2514675 | 5208 MB
(1 row)
```

5. Придумать методы избавится от него и проверить на практике

```sql
-- Самый простой способ vacuum full
# vacuum full;
VACUUM

-- тоаст уменьшился
# SELECT oid::regclass AS heap_rel,
       pg_size_pretty(pg_relation_size(oid)) AS heap_rel_size,
       reltoastrelid::regclass AS toast_rel,
       pg_size_pretty(pg_relation_size(reltoastrelid)) AS toast_rel_size
FROM pg_class WHERE relname = 't';
 heap_rel | heap_rel_size |         toast_rel         | toast_rel_size 
----------+---------------+---------------------------+----------------
 t        | 50 MB         | pg_toast.pg_toast_2514675 | 2604 MB
(1 row)

```

6. Не забываем про блоатинг индексов*

```sql
-- размер индекса до reindex
# SELECT c.relname AS index_name, pg_size_pretty(pg_relation_size(c.oid)) AS index_size
FROM pg_class c
WHERE c.relname = 'idx_t';
 index_name | index_size 
------------+------------
 idx_t      | 1874 MB
(1 row)

# REINDEX INDEX CONCURRENTLY public.idx_t;
REINDEX

# SELECT c.relname AS index_name, pg_size_pretty(pg_relation_size(c.oid)) AS index_size
FROM pg_class c
WHERE c.relname = 'idx_t';
 index_name | index_size 
------------+------------
 idx_t      | 820 MB
(1 row)


```
