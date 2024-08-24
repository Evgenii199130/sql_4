# sql_4

Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.



Задание 2
Выполните explain analyze следующего запроса:

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
перечислите узкие места;
оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

Ответ:
![1](https://github.com/Evgenii199130/sql_4/blob/main/scrin/sql_4.1.png)

### Узкие места

1. Использование функции DATE(): Применение функции DATE() к столбцу p.payment_date может привести к тому, что индекс не будет использоваться, что замедлит выполнение запроса.
2. Кросс-джойны: Использование старого синтаксиса для объединения таблиц (FROM ... WHERE ...) может быть менее оптимально, чем явные JOIN операторы.
3. Отсутствие индексов: Если таблицы не индексированы по столбцам, используемым в условиях объединения и фильтрации, это может значительно замедлить выполнение запроса.


### 1. Перепишем запрос с использованием явных JOIN операторов

SELECT DISTINCT 
    CONCAT(c.last_name, ' ', c.first_name) AS customer_name, 
    SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title) AS total_amount
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE p.payment_date >= '2005-07-30 00:00:00'
  AND p.payment_date < '2005-07-31 00:00:00';

#### 2. Добавление индексов


CREATE INDEX idx_payment_date ON payment (payment_date);
CREATE INDEX idx_rental_date ON rental (rental_date);
CREATE INDEX idx_customer_id ON customer (customer_id);
CREATE INDEX idx_inventory_id ON inventory (inventory_id);
CREATE INDEX idx_film_id ON film (film_id);

### Выполнение оптимизированного EXPLAIN ANALYZE

EXPLAIN ANALYZE
SELECT DISTINCT 
    CONCAT(c.last_name, ' ', c.first_name) AS customer_name, 
    SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title) AS total_amount
FROM payment p
JOIN rental r ON p.
payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE p.payment_date >= '2005-07-30 00:00:00'
  AND p.payment_date < '2005-07-31 00:00:00';

### Проверка результатов

![1](https://github.com/Evgenii199130/sql_4/blob/main/scrin/sql_4.2.png)


Задание 3*
Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

Приведите ответ в свободной форме.

Ответ:

В PostgreSQL существует несколько типов индексов, некоторые из которых отсутствуют в MySQL. Вот краткий список индексов, которые используются в PostgreSQL, но отсутствуют в MySQL:

1. GIN (Generalized Inverted Index): Этот тип индекса используется для эффективного поиска в полях массивов, JSONB и т.д. MySQL не имеет аналогичного индекса.

2. GiST (Generalized Search Tree): Поддерживает широкий спектр пользовательских индексов и используется для геометрических данных, полнотекстового поиска и других. В MySQL нет аналогов.

3. BRIN (Block Range INdex): Индексы на основе диапазонов блоков, которые эффективны для очень больших таблиц, где данные имеют естественный порядок. В MySQL также отсутствует аналогичный тип индекса.

Эти типы индексов предоставляют дополнительные возможности и оптимизации, которые могут быть полезны в специфических сценариях использования PostgreSQL.
