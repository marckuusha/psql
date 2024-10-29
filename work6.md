

2. Залил самую маленькую версию базы

3. Проверить скорость выполнения сложного запроса (приложен в конце файла скриптов)

```sql
-- После выполнения explain analyze показывает следующий план
Limit  (cost=329314.10..329314.12 rows=10 width=56) (actual time=1282.307..1297.716 rows=10 loops=1)
...
```

4. Навесить индексы на внешние ключ

```sql 
-- Создаю индексы на все внешние ключи используемые в запросе
CREATE INDEX ON book.seat (fkbus);
CREATE INDEX ON book.tickets (fkride);
CREATE INDEX ON book.ride (fkschedule);
CREATE INDEX ON book.schedule (fkroute);
CREATE INDEX ON book.busroute (fkbusstationfrom);
CREATE INDEX ON book.ride (fkbus);
```

5. Проверить, помогли ли индексы на внешние ключи ускориться
```sql
-- План запроса никак не поменялся, возможно нужно применить другую стратегию (сделать через матвью)? Потому как отключение seq scan еще хуже план строит
Limit  (cost=329314.10..329314.12 rows=10 width=56) (actual time=1282.307..1297.716 rows=10 loops=1)
...
```