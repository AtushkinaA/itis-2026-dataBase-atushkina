1. AVG

1.1 Среднее значение кассовых сборов всех фильмов
```sql
SELECT AVG(box_office) AS Average_Box_Office FROM film;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic.png>)

1.2 Среднее значение прибыли всех фильмов
```sql
SELECT AVG(earnings) AS Average_Earnings FROM film;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 1.png>)

2. COUNT

2.1 Количество человек, которые зарегистрированы в кинотеатре
```sql
SELECT COUNT(*) FROM human;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 2.png>)

2.2 Вывести сколько уникальных значений возрастного ограничения есть в таблице film
```sql
SELECT COUNT(DISTINCT age_rating) FROM film;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 3.png>)

3. MIN

3.1 Вывести минимальный кассовый сбор из всех фильмов
```sql
SELECT MIN(box_office) AS Min_Box_Office_Of_Film FROM film;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 4.png>)

3.2 Вывести минимальную прибыль из всех фильмов
```sql
SELECT MIN(earnings) AS Min_Earnings_Of_Film FROM film;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 6.png>)


4. MAX

4.1 Вывести максимальный кассовый сбор из всех фильмов
```sql
SELECT MAX(box_office) AS Max_Box_Office_Of_Film FROM film;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 5.png>)


4.2 Вывести максимальную прибыль из всех фильмов
```sql
SELECT MAX(earnings) AS Max_Earnings_Of_Film FROM film;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 7.png>)


5. SUM

5.1 Вывести общую сумму кассовых сборов на все фильмы
```sql
SELECT SUM(box_office) FROM film;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 8.png>)

5.2 Вывести общую сумму мест во всех залах
```sql
SELECT SUM(num_seats) AS all_count_seats FROM hall;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 9.png>)


6. STRING_AGG

6.1 Вывести все названия фильмов
```sql
SELECT STRING_AGG(title, ', ') FROM film;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 10.png>)

6.2 Вывести фио всех покупателей
```sql
SELECT STRING_AGG(fio, ', ') FROM human;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 11.png>)


7. GROUP BY

7.1 Названия фильмов, сгруппированные по жанру

```sql
SELECT genre, title
FROM film
GROUP BY genre, title;
```
Результат выполнения запроса
![alt text](<img/7.1.png>)

7.2 ФИО покупателей, сгруппированные по дате рождения
```sql
SELECT birthday, fio
FROM human
GROUP BY birthday, fio;
```
Результат выполнения запроса
![alt text](<img/7.2.png>)


8. HAVING

8.1 Вывести фильмы только с возрастным рейтингом 18+
```sql
SELECT age_rating, title
FROM film
GROUP BY age_rating, title
HAVING age_rating = '18+';
```
Результат выполнения запроса
![alt text](<img/8.1.png>)

8.2 Вывести залы с числом мест больше 100
```sql
SELECT num_hall, num_seats
FROM hall
GROUP BY num_hall, num_seats
HAVING num_seats > 100;
```
Результат выполнения запроса
![alt text](<img/8.2.png>)


9. GROUPING SETS

9.1 Сборы по фильмам + жанрам + общий итог
```sql
SELECT genre,
       title,
       SUM(box_office) AS total_box
FROM film
GROUP BY GROUPING SETS ((genre, title), (genre), ());
```
Результат выполнения запроса
![alt text](<img/9.1.png>)

9.2  Сборы по возрастному рейтингу + жанрам
```sql
SELECT age_rating,
       genre,
       SUM(box_office) AS total_box
FROM film
GROUP BY GROUPING SETS ((age_rating), (genre));
```
Результат выполнения запроса
![alt text](<img/9.2.png>)


10. ROLLUP

10.1 Вывести места по количеству рядов + общий итог
```sql
SELECT num_row,
       num_hall,
       SUM(num_seats) AS total_seats
FROM hall
GROUP BY ROLLUP (num_row, num_hall);
```
Результат выполнения запроса
![alt text](<img/10.1.png>)

10.2 Сколько продано мест по каждому ряду + итог
```sql
SELECT num_row,
       SUM(num_seats) AS total_seats
FROM hall
GROUP BY ROLLUP (num_row);
```
Результат выполнения запроса
![alt text](<img/10.2.png>)


11. CUBE

11.1 Вывести места по числу рядов, число залов и все подытоги
```sql
SELECT num_row,
       num_hall,
       SUM(num_seats) AS total_seats
FROM hall
GROUP BY CUBE (num_row, num_hall);
```
Результат выполнения запроса
![alt text](<img/11.1.png>)

11.2 Вывести количество сеансов по залу, фильму и все подытоги
```sql
SELECT hall_id,
       film_id,
       COUNT(*) AS sessions
FROM session_
GROUP BY CUBE (hall_id, film_id);
```
Результат выполнения запроса
![alt text](<img/11.2.png>)


12.SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY

12.1 Выбрать жанры, у которых средние сборы больше 100 млн, отсортировать по убыванию
```sql
SELECT genre, AVG(box_office) AS avg_box
FROM film
WHERE age_rating = '18+'
GROUP BY genre
HAVING AVG(box_office) > 100000000
ORDER BY avg_box DESC;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 12.png>)

12.2 Вывести сколько фильмов, которые идут больше 100 минут, в каждом жанре
```sql
SELECT genre, COUNT(*) AS cnt
FROM film
WHERE time_of_film > 100
GROUP BY genre
HAVING COUNT(*) >= 1
ORDER BY cnt DESC;
```
Результат выполнения запроса
![alt text](<img/Pasted Graphic 13.png>)