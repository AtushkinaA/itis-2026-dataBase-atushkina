1. CTE

1.1 Вывести жанры, которые в среднем собирают больше 250000000
```sql
WITH GenreStats AS (
    SELECT 
        genre,
        COUNT(film_id) AS total_films,
        AVG(box_office) AS avg_box_office
    FROM film
    GROUP BY genre
)
SELECT 
    genre,
    total_films,
    ROUND(avg_box_office::numeric, 2) AS avg_earnings
FROM GenreStats
WHERE avg_box_office > 250000000
ORDER BY avg_box_office DESC;
```
Результат выполнения запроса:
![alt text](<img/cte1.png>)

1.2 Выведите клиентов, которые купили хотя бы один билет
```sql
WITH ClientTicket AS (
    SELECT 
        h.fio,
        h.email,
        s.film_id,
        s.hall_id
    FROM ticket t
    JOIN session_ s ON t.session_id = s.session_id
    JOIN human h ON t.human_id = h.human_id
)
SELECT 
    ct.fio,
    ct.email,
    f.title,
    h.num_hall
FROM ClientTicket ct
JOIN film f ON ct.film_id = f.film_id
JOIN hall h ON ct.hall_id = h.hall_id
ORDER BY ct.fio;
```
Результат выполнения запроса:
![alt text](<img/cte2.png>)

1.3 Расписание сеансов с названиями фильмов и номерами залов
```sql
WITH DetailedSessions AS (
    SELECT 
        s.session_id,
        f.title AS film_title,
        f.genre,
        h.num_hall,
        s.date_time
    FROM session_ s
    JOIN film f ON s.film_id = f.film_id
    JOIN hall h ON s.hall_id = h.hall_id
)
SELECT 
    date_time,
    film_title,
    genre,
    num_hall
FROM DetailedSessions
ORDER BY date_time;
```
Результат выполнения запроса:
![alt text](<img/cte3.png>)


2. UNION, INTERSECT, EXCEPT

2.1 Список контактов
```sql
SELECT fio, 'Клиент' AS contact_type
FROM human WHERE human_id <= 10

UNION ALL

SELECT fio, 'Сотрудник' AS contact_type
FROM human WHERE human_id > 10

ORDER BY contact_type, fio;
```
Результат выполнения запроса:
![alt text](<img/union.png>)

2.2 Любители хорроров и триллеров
```sql
SELECT h.fio
FROM human h
JOIN ticket t ON h.human_id = t.human_id
JOIN session_ s ON t.session_id = s.session_id
JOIN film f ON s.film_id = f.film_id
WHERE f.genre = 'horror'

INTERSECT

SELECT h.fio
FROM human h
JOIN ticket t ON h.human_id = t.human_id
JOIN session_ s ON t.session_id = s.session_id
JOIN film f ON s.film_id = f.film_id
WHERE f.genre = 'thriller';
```
Результат выполнения запроса:
![alt text](<img/intersect.png>)

2.3 Клиенты, которые не пришли на комедию
```sql
SELECT h.fio
FROM human h
JOIN ticket t ON h.human_id = t.human_id

EXCEPT

SELECT h.fio
FROM human h
JOIN ticket t ON h.human_id = t.human_id
JOIN session_ s ON t.session_id = s.session_id
JOIN film f ON s.film_id = f.film_id
WHERE f.genre = 'comedy';
```
Результат выполнения запроса:
![alt text](<img/except.png>)


3. PARTITION BY

3.1 Считаем количество билетов, купленных каждым человеком
```sql
SELECT 
    h.fio,
    t.ticket_id,
    s.session_id,
    COUNT(t.ticket_id) OVER (PARTITION BY h.human_id) AS total_tickets_by_user
FROM human h
JOIN ticket t ON h.human_id = t.human_id
JOIN session_ s ON t.session_id = s.session_id
ORDER BY h.fio, t.ticket_id;
```
Результат выполнения запроса:
![alt text](<img/okno1.png>)

3.2 Максимальный box_office внутри каждого жанра
```sql
SELECT 
    title,
    genre,
    box_office,
    MAX(box_office) OVER (PARTITION BY genre) AS max_box_office_in_genre
FROM film
ORDER BY genre, box_office DESC;
```
Результат выполнения запроса:
![alt text](<img/okno2.png>)


4. PARTITION BY + ORDER BY

4.1 Сколько билетов клиент купил нарастающим итогом от самого раннего билета
```sql
SELECT 
    h.fio,
    t.ticket_id,
    t.sale_date,
    COUNT(t.ticket_id) OVER (
        PARTITION BY h.human_id 
        ORDER BY t.sale_date
    ) AS running_total_tickets
FROM human h
JOIN ticket t ON h.human_id = t.human_id
ORDER BY h.fio, t.sale_date;
```
Результат выполнения запроса:
![alt text](<img/okno3.png>)

4.2 Накопительная сумма сборов по жанрам
```sql
SELECT 
    title,
    genre,
    box_office,
    SUM(box_office) OVER (
        PARTITION BY genre 
        ORDER BY box_office DESC
    ) AS running_box_office_sum
FROM film
ORDER BY genre, box_office DESC;
```
Результат выполнения запроса:
![alt text](<img/okno4.png>)

5. ROWS

5.1 3 последних билета человека
```sql
SELECT 
    h.fio,
    t.ticket_id,
    t.sale_date,
    COUNT(*) OVER (
        PARTITION BY h.human_id 
        ORDER BY t.sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS tickets_in_last_3_purchases
FROM human h
JOIN ticket t ON h.human_id = t.human_id
ORDER BY h.fio, t.sale_date;
```
Результат выполнения запроса:
![alt text](<img/okno5.png>)

5.2  Сравнение с предыдущим и следующим билетом
```sql
SELECT 
    t.ticket_id,
    t.human_id,
    SUM(t.ticket_id) OVER (
        PARTITION BY t.human_id 
        ORDER BY t.ticket_id
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) AS sum_of_neighbor_ticket_ids
FROM ticket t
ORDER BY t.human_id, t.ticket_id;
```
Результат выполнения запроса:
![alt text](<img/okno6.png>)

6. RANGE

6.1 Все покупки за последние 30 дней
```sql
SELECT 
    h.fio,
    t.ticket_id,
    t.sale_date,
    COUNT(*) OVER (
        PARTITION BY h.human_id 
        ORDER BY t.sale_date
        RANGE BETWEEN INTERVAL '30 days' PRECEDING AND CURRENT ROW
    ) AS tickets_last_30_days
FROM human h
JOIN ticket t ON h.human_id = t.human_id
ORDER BY h.fio, t.sale_date;
```
Результат выполнения запроса:
![alt text](<img/okno7.png>)

6.2 Сумма сборов фильмов с разницей box_office ≤ 100 млн
```sql
SELECT 
    title,
    genre,
    box_office,
    SUM(box_office) OVER (
        PARTITION BY genre 
        ORDER BY box_office
        RANGE BETWEEN 100000000 PRECEDING AND CURRENT ROW
    ) AS box_office_in_range_100m
FROM film
ORDER BY genre, box_office;
```
Результат выполнения запроса:
![alt text](<img/okno8.png>)

7. Ранжирующие функции

7.1 Нумерация фильмов по жанру
```sql
SELECT 
    genre,
    title,
    box_office,
    ROW_NUMBER() OVER (PARTITION BY genre ORDER BY box_office DESC) AS row_num
FROM film
ORDER BY genre, row_num;
```
Результат выполнения запроса:
![alt text](<img/row.png>)

7.2 Фильмы по жанрам и box_office (rank)
```sql
SELECT 
    genre,
    title,
    box_office,
    RANK() OVER (PARTITION BY genre ORDER BY box_office DESC) AS rank_with_gaps
FROM film
ORDER BY genre, rank_with_gaps;
```
Результат выполнения запроса:
![alt text](<img/rank.png>)

7.3 Фильмы по жанрам и box_office (DENSE_RANK)
```sql
SELECT 
    genre,
    title,
    box_office,
    DENSE_RANK() OVER (PARTITION BY genre ORDER BY box_office DESC) AS dense_rank_no_gaps
FROM film
ORDER BY genre, dense_rank_no_gaps;
```
Результат выполнения запроса:
![alt text](<img/DENSE_RANK.png>)


8. Оконные функции смещения

8.1 Достаем значения из предыдущего фильма
```sql
SELECT 
    title,
    genre,
    box_office,
    LAG(box_office, 1, 0) OVER (PARTITION BY genre ORDER BY box_office DESC) AS previous_box_office
FROM film
ORDER BY genre, box_office DESC;
```
Результат выполнения запроса:
![alt text](<img/lag.png>)

8.2 Достаем значения из следующего фильма
```sql
SELECT 
    title,
    genre,
    box_office,
    LEAD(title) OVER (PARTITION BY genre ORDER BY box_office DESC) AS next_film_title
FROM film
ORDER BY genre, box_office DESC;
```
Результат выполнения запроса:
![alt text](<img/lead.png>)

8.3 Самый кассовый фильм жанра
```sql
SELECT 
    title,
    genre,
    box_office,
    FIRST_VALUE(title) OVER (PARTITION BY genre ORDER BY box_office DESC) AS top_film_in_genre
FROM film
ORDER BY genre, box_office DESC;
```
Результат выполнения запроса:
![alt text](<img/first_value.png>)

8.4 Самый некассовый фильм жанра
```sql
SELECT 
    title,
    genre,
    box_office,
    LAST_VALUE(title) OVER (
        PARTITION BY genre 
        ORDER BY box_office DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS worst_film_in_genre
FROM film
ORDER BY genre, box_office DESC;
```
Результат выполнения запроса:
![alt text](<img/last_value.png>)