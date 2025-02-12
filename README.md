## Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

## ОТВЕТ:

SELECT 
    ROUND(SUM(index_length) / SUM(data_length + index_length) * 100, 2) AS '% of Index'
FROM 
    INFORMATION_SCHEMA.TABLES
WHERE 
    TABLE_SCHEMA NOT IN ('mysql', 'information_schema', 'performance_schema')
    AND data_length > 0;

## Задание 2

Выполните explain analyze следующего запроса:

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
перечислите узкие места;
оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

## ОТВЕТ:

При использование запроса из примера было обработано 200 записей за 15 секунд.  inventory, rental и film лишние.

Исправленный запрос.

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p, customer c
where date(p.payment_date) = '2005-07-30' and p.customer_id = c.customer_id 

Обработано 391 запись за 0.2 мс.
