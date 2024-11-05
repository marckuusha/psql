Проанализировать данные о зарплатах сотрудников с использованием оконных функций.
а) На сколько было увеличение с предыдущей зарплатой
б) если это первая зарплата - вместо NULL вывести 0

```sql
SELECT 
    fk_employee,
    amount,
    from_date,
    to_date,
    fk_grade,
    COALESCE(amount - LAG(amount) OVER (PARTITION BY fk_employee ORDER BY from_date), 0) AS increase
FROM 
    salary
ORDER BY 
    fk_employee, from_date;
```
