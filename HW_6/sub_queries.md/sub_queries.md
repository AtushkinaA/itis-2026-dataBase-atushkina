1. SELECT

1.1 Вывод первого фильма
```sql
SELECT (SELECT title FROM film ORDER BY film_id LIMIT 1) AS first_film;
```
Результат выполнения запроса
![alt text](<img/1.png>)

1.2 Среднее значение по времени всех фильмов
```sql
SELECT (SELECT ROUND(AVG(time_of_film), 0) FROM film) AS avg_times_films;
```
Результат выполнения запроса
![alt text](<img/2.png>)

2. FROM

2.1  Вывести ID зала и количество мест, но только для тех залов, где мест больше 100
```sql
SELECT hall_id, seats
FROM (
    SELECT hall_id, SUM(num_seats) AS seats
    FROM hall
    GROUP BY hall_id
) AS sub
WHERE seats > 100;
```
Результат выполнения запроса
![alt text](<img/3.png>)

2.2  Вывести ID человека и количество купленных им билетов, с хотя бы 1 билетом
```sql
SELECT human_id, total_tickets
FROM (
    SELECT human_id, COUNT(ticket_id) AS total_tickets
    FROM ticket
    GROUP BY human_id
) AS sub
WHERE total_tickets >= 1;
```
Результат выполнения запроса
![alt text](<img/4.png>)

3. WHERE

3.1 Вывод фильма с максимальным значением по времени
```sql
SELECT *  FROM film 
WHERE time_of_film = (SELECT MAX(time_of_film) FROM film);
```
Результат выполнения запроса
![alt text](<img/5.png>)

3.2 Вывод самых длинных имен актеров
```sql
SELECT fio
FROM actor
WHERE LENGTH(fio) = (SELECT MAX(LENGTH(fio)) FROM actor);
```
Результат выполнения запроса
![alt text](<img/6.png>)


4. HAVING

4.1 Вывести жанры, чьи фильмы в среднем идут дольше, чем общая средняя длительность по всем фильмам
```sql
SELECT genre, ROUND(AVG(time_of_film), 0) AS avg_time
FROM film
GROUP BY genre
HAVING AVG(time_of_film) > (
    SELECT AVG(time_of_film) FROM film
);
```
Результат выполнения запроса
![alt text](<img/7.png>)


4.2  Фильмы, которые идут дольше, чем самый короткий фильм в базе
```sql
SELECT title, SUM(time_of_film) AS total_time
FROM film
GROUP BY title
HAVING SUM(time_of_film) > (
    SELECT MIN(time_of_film) FROM film
);
```
Результат выполнения запроса
![alt text](<img/8.png>)


5. ALL

5.1 Вывод все ли фильмы идут меньше 200 минут?
```sql
SELECT 200 > ALL(SELECT time_of_film FROM film);
```
Результат выполнения запроса
![alt text](<img/9.png>)

5.2 Вывести фильмы, которые идут дольше, чем любой фильм жанра комедия
```sql
SELECT title, time_of_film
FROM film
WHERE time_of_film > ALL (
    SELECT time_of_film 
    FROM film 
    WHERE genre = 'comedy'
);
```
Результат выполнения запроса
![alt text](<img/10.png>)


6. IN

6.1 Вывести фильмы, на которые были проданы билеты
```sql
SELECT title, genre
FROM film
WHERE film_id IN (
    SELECT s.film_id 
    FROM session_ s
    JOIN ticket t ON s.session_id = t.session_id
);
```
Результат выполнения запроса
![alt text](<img/11.png>)

6.2 Вывести всех актеров, которые играли в фильмах, где кассовые сборы больше или равны 150000000.
```sql
SELECT * FROM actor WHERE actor_id IN (
    SELECT DISTINCT fa.actor_id 
    FROM film_actor fa 
    JOIN film f ON fa.film_id = f.film_id 
    WHERE f.earnings >= 150000000
);
```
Результат выполнения запроса
![alt text](<img/12.png>)


7. ANY

7.1 Найти фильмы, которые идут дольше, чем хотя бы 1 фильм жанра хоррор.

```sql
SELECT title, time_of_film
FROM film
WHERE time_of_film > ANY (
    SELECT time_of_film 
    FROM film 
    WHERE genre = 'horror'
);
```
Результат выполнения запроса
![alt text](<img/13.png>)

7.2 Вывести залы, в которых проходили сеансы фильмов с рейтингом 18+
```sql
SELECT * FROM hall 
WHERE hall_id = ANY (
    SELECT s.hall_id 
    FROM session_ s
    JOIN film f ON s.film_id = f.film_id
    WHERE f.age_rating = '18+'
);
```
Результат выполнения запроса
![alt text](<img/14.png>)


8. EXIST

8.1 Вывести фильмы, на которые не купили ни одного билета
```sql
SELECT f.title 
FROM film f
WHERE NOT EXISTS (
    SELECT 1 
    FROM session_ s
    JOIN ticket t ON s.session_id = t.session_id
    WHERE s.film_id = f.film_id
);
```
Результат выполнения запроса
![alt text](<img/15.png>)

8.2 Вывести имена людей, которые хоть раз покупали билет
```sql
SELECT fio
FROM human h
WHERE EXISTS (
    SELECT 1
    FROM ticket t
    WHERE t.human_id = h.human_id
);
```
Результат выполнения запроса
![alt text](<img/16.png>)


9. Сравнение по нескольким столбцам

9.1 Сверка сеансов (по фильму и залу)
```sql
SELECT * FROM session_
WHERE (film_id, hall_id) IN (
    SELECT film_id, hall_id FROM session_info_bad_2nf
);
```
Результат выполнения запроса
![alt text](<img/17.png>)

9.2  Поиск точных совпадений по ряду и месту
```sql
SELECT * FROM ticket
WHERE (row_, seat) IN (
    SELECT row_, seat FROM ticket_info
);
```
Результат выполнения запроса
![alt text](<img/18.png>)


10. Коррелированные подзапросы

10.1 Фильмы, на которые есть билеты
```sql
SELECT title 
FROM film f
WHERE EXISTS (
    SELECT 1 
    FROM session_ s
    JOIN ticket t ON s.session_id = t.session_id
    WHERE s.film_id = f.film_id
);
```
Результат выполнения запроса
![alt text](<img/19.png>)

10.2 Количество билетов на каждый фильм
```sql
SELECT 
    f.title,
    (SELECT COUNT(*) 
     FROM session_ s
     JOIN ticket t ON s.session_id = t.session_id
     WHERE s.film_id = f.film_id) AS tickets_sold
FROM film f;
```
Результат выполнения запроса
![alt text](<img/20.png>)

10.3 Залы, в которых есть сеансы
```sql
SELECT hall_id, num_hall 
FROM hall h
WHERE EXISTS (
    SELECT 1 
    FROM session_ s 
    WHERE s.hall_id = h.hall_id
);
```
Результат выполнения запроса
![alt text](<img/21.png>)

10.4  Люди, которые покупали билеты
```sql
SELECT fio, email
FROM human h
WHERE EXISTS (
    SELECT 1 
    FROM ticket t 
    WHERE t.human_id = h.human_id
);
```
Результат выполнения запроса
![alt text](<img/22.png>)

10.5 Самый длинный фильм по жанрам 
```sql
SELECT 
    f.title,
    f.genre,
    (SELECT MAX(f2.time_of_film) 
     FROM film f2 
     WHERE f2.genre = f.genre) AS max_in_genre
FROM film f;
```
Результат выполнения запроса
![alt text](<img/23.png>)
