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

SELECT concat(c.last_name, ' ', c.first_name) AS Клиент, SUM(p.amount) as Платеж
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id 
JOIN payment p ON r.rental_date = p.payment_date 
join inventory i on i.inventory_id = r.inventory_id 
where date(p.payment_date) >= '2005-07-30' and date(p.payment_date) < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
GROUP BY c.customer_id;

## Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

Приведите ответ в свободной форме.

В PostgreSQL используется несколько типов индексов, которые могут не иметь аналогов в MySQL. Вот основные типы индексов, которые присутствуют в PostgreSQL, но отсутствуют в MySQL:

GIN (Generalized Inverted Index): Этот индекс предназначен для работы с массивами, JSONB и другими типами данных, которые содержат множество значений. GIN индексы обеспечивают быстрый поиск по элементам.

GiST (Generalized Search Tree): GiST индексы могут использоваться для различных типов данных и обеспечивают поддержку неполных данных и пользовательских типов данных. Они хорошо подходят для пространственных данных и многоуровневых данных.

BRIN (Block Range INdex): BRIN индексы особенно эффективны для очень больших таблиц, в которых данные естественным образом упорядочены. Они хранят информацию о диапазонах блоков, что позволяет экономить место и ускорять запросы.

SP-GiST (Space-partitioned Generalized Search Tree): Этот индекс используется для данных, которые имеют пространственную или иерархическую структуру. Он хорошо подходит для работы с данными, которые требуют разбиения пространства (например, для геометрических типов).

Bloom Index: Такой индекс используется для эффективной фильтрации больших наборов данных, когда необходимо проверить наличие элементов, но не требуется точность поиска.

Эти индексы предоставляют PostgreSQL более продвинутые механизмы для оптимизации поиска и работы с данными различных типов, в отличие от стандартных индексов, таких как B-tree, которые хорошо реализованы в MySQL. Однако стоит отметить, что MySQL имеет свои собственные механизмы индексации и оптимизации, такие как использование полнотекстовых индексов и других подходов.
