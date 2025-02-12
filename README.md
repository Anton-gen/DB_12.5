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

При использование запроса из примера было обработано 200 записей за 15 секунд.

Исправленный запрос.

SELECT 
    CONCAT(c.last_name, ' ', c.first_name) AS Клиент, 
    SUM(p.amount) AS Платеж
FROM 
    customer c
JOIN 
    rental r ON c.customer_id = r.customer_id 
JOIN 
    payment p ON r.rental_id = p.rental_id 
JOIN 
    inventory i ON i.inventory_id = r.inventory_id 
WHERE 
    p.payment_date >= '2005-07-30' 
    AND p.payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
GROUP BY 
    c.customer_id, c.last_name, c.first_name;

Обработано за 0.2 мс.
