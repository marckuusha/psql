
1. Создать таблицу с продажами.

```sql
-- создадим таблицы
CREATE TABLE sales (
    id serial primary key,
    ddate date not null,
    amount int4 
);

-- сгенерируем тестовые данные
DO
$$
DECLARE
    i INT := 1;
    random_date DATE;
    random_amount NUMERIC(10, 2);
BEGIN
    WHILE i <= 100 LOOP
        -- Генерация случайной даты в пределах года (2023 год)
        random_date := DATE '2023-01-01' + (random() * 364)::INT;
        
        -- Генерация случайной суммы от 10 до 1000
        random_amount := round((10 + random() * 990)::NUMERIC, 2);
        
        -- Вставка данных
        INSERT INTO sales (sale_date, amount)
        VALUES (random_date, random_amount);
        
        i := i + 1;
    END LOOP;
END
$$;

```

2. Реализовать функцию выбор трети года (1-4 месяц - первая треть, 5-8 - вторая и тд)

```sql
CREATE OR REPLACE FUNCTION yearthird(sale_date DATE)
RETURNS INT
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN CASE
        WHEN sale_date IS NULL THEN NULL
        WHEN EXTRACT(MONTH FROM sale_date) BETWEEN 1 AND 4 THEN 1
        WHEN EXTRACT(MONTH FROM sale_date) BETWEEN 5 AND 8 THEN 2
        WHEN EXTRACT(MONTH FROM sale_date) BETWEEN 9 AND 12 THEN 3
        ELSE NULL
    END;
END;
$$;
```

3. Вызвать эту функцию в SELECT из таблицы с продажами, уведиться, что всё отработало

```sql
-- (Вроде верно)
SELECT *, yearthird(ddate)  FROM sales

1	2023-09-03	828	3
2	2023-03-18	597	1
3	2023-02-19	470	1
4	2023-08-20	586	2
5	2023-12-25	575	3
```