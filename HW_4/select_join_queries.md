1. Выборка всех данных из таблицы

1.1 Выборка всех данных из таблицы film
‘’’ sql
	SELECT * FROM film;
‘’’
Результат выполнения запроса: 1.1
1.2 Выборка всех данных из таблицы human
‘’’ sql
	SELECT * FROM human;
‘’’
Результат выполнения запроса: 1.2

2. Выборка отдельных столбцов

2.1 Вывести из таблицы film только данные из столбцов title и time_of_film
‘’’ sql
	SELECT title, time_of_film FROM film;
‘’’
Результат выполнения запроса: 2.1
2.2 Вывести из таблицы human только данные из столбцов fio и email
‘’’ sql
	SELECT fio, email FROM human;
‘’’
Результат выполнения запроса: 2.2

3. Присвоение новых имен столбцам при формировании выборки

3.1 Присвоение нового имени(Почта) столбцу email таблицы human и выборка столбцов email и fio
‘’’ sql
	SELECT email AS Почта, fio FROM human;
‘’’
Результат выполнения запроса: 3.1
3.2 Присвоение нового имени(Ряд) столбцу row_ и нового имени(Место) столбцу seat таблицы ticket и выборка столбцов ticket_id, sale_date
‘’’ sql
	SELECT ticket_id, row_ AS Ряд, seat AS Место, sale_date FROM ticket;
‘’’
Результат выполнения запроса: 3.2

4. Выборка данных с созданием вычисляемого столбца

4.1 Вычисляем сколько всего мест в каждом зале (таблица hall и столбцы num_row, num_seats)
‘’’ sql
	SELECT hall_id, num_hall, num_row, num_seats, num_row * num_seats AS total_seats FROM hall;
‘’’
Результат выполнения запроса: 4.1
4.2 Удобно выводим контакты пользователя в отдельный столбец
‘’’ sql
	SELECT fio, email, fio || ‘-‘ || email AS contact FROM human;
‘’’
Результат выполнения запроса: 4.2

5. Выборка данных, вычисляемые столбцы, математические функции

5.1 Длительность фильма в часах и минутах таблицы film
‘’’ sql
	SELECT film_id, title, genre, time_of_film / 60 AS hours, time_of_film % 60 AS minutes, age_rating FROM film;
‘’’
Результат выполнения запроса: 5.1
5.2 Увеличенная длительность фильма из-за рекламы таблицы film
‘’’ sql
	SELECT title, time_of_film, time_of_film + 15 AS time_with_advert FROM film;
‘’’
Результат выполнения запроса: 5.2


6. Выборка данных, вычисляемые столбцы, логические функции

6.1 Для каждого фильма из таблицы film установим категорию фильма по длительности, если фильм больше 120 минут, то он длинный, иначе короткий
‘’’ sql
	SELECT title, time_of_film, 
	CASE
		WHEN time_of_film < 120 THEN ‘Короткий’
		ELSE ‘Длинный’
	END AS film_category
FROM film; 
‘’’
Результат выполнения запроса: 6.1
6.2 Для каждого зала из таблицы hall установим категорию зала по количеству мест, если мест меньше 100, то он маленький, если мест больше 200, то он большой, иначе стандартный
‘’’ sql
	SELECT num_hall, num_row, num_seats,
	CASE 
		WHEN num_seats < 100 THEN ‘Маленький’
		WHEN num_seats > 200 THEN ‘Большой’
		ELSE ‘Стандартный’
	END AS hall_category
FROM hall;
‘’’
Результат выполнения запроса: 6.2

7. Выборка данных по условию

7.1 Вывести название фильма и жанр, фильмы которые идут меньше 140 минут
‘’’ sql
	SELECT title, genre
	FROM film
	WHERE time_of_film < 140;
‘’’
Результат выполнения запроса: 7.1
7.2 Вывести номер зала, в котором больше 8 рядов
‘’’ sql
	SELECT num_hall, num_row
	FROM hall
	WHERE num_row > 8;
‘’’
Результат выполнения запроса: 7.2

8. Выборка данных, логические операции
 
8.1 Вывести название, жанр, возрастной рейтинг тех фильмов, которые имеют жанр Хоррор или Триллер
‘’’ sql
	SELECT title, genre, age_rating
	FROM film
	WHERE genre = ‘horror’ OR genre = ‘thriller’;
‘’’
Результат выполнения запроса: 8.1
8.2  Вывести название, жанр, возрастной рейтинг тех фильмов, которые не имеют жанр Хоррор и не 18+
‘’’ sql
	SELECT title, genre, age_rating
	FROM film
	WHERE NOT genre = ‘horror’ AND NOT age_rating = ’18+’;
‘’’
Результат выполнения запроса: 8.2

9. Выборка данных, операторы BETWEEN, IN

9.1 Вывести номер зала и количество рядов, которых от 5 до 14
‘’’ sql
	SELECT num_hall, num_row
	FROM hall
	WHERE num_row BETWEEN 5 AND 14;
‘’’
Результат выполнения запроса: 9.1
9.2 Вывести фио и email с соответствующими значениями из списка
‘’’ sql
	SELECT fio, email
	FROM human
	WHERE fio IN (‘Atushkina Arina Valeryevna’, ‘Jason Statham’);
‘’’
Результат выполнения запроса: 9.2

10. Выборка данных с сортировкой

10.1 Вывести название, жанр и длительность фильма (от коротких к длинным)
‘’’ sql
	SELECT title, genre, time_of_film
	FROM film
	ORDER BY time_of_film;
‘’’
Результат выполнения запроса: 10.1
10.2 Вывести фио по убыванию алфавита, email по возрастанию алфавита, и день рождение таблицы human 
‘’’ sql
	SELECT fio, email, birthday
	FROM human
	ORDER BY email, fio DESC;
‘’’
Результат выполнения запроса: 10.2

11. Выборка данных, оператор LIKE

11.1 Вывести фио посетителей, которые родились в 20 веке
‘’’ sql
	SELECT fio, birthday FROM human
	WHERE birthday::text LIKE '19__-%';
‘’’
Результат выполнения запроса: 11.1
11.2 Вывести все фио и email, который не mail.ru
‘’’ sql
	SELECT fio, email FROM human
	WHERE email NOT LIKE ‘%@mail.ru’;
‘’’
Результат выполнения запроса: 11.2

12. Выбор уникальных элементов столбца

12.1 Вывести какие вообще рейтинги есть в базе, без повторов
‘’’ sql
	SELECT DISTINCT age_rating FROM film;
‘’’
Результат выполнения запроса: 12.1
12.2 Вывести в каких залах вообще проходили сеансы без повторов
‘’’ sql
	SELECT DISTINCT hall_id FROM session_;
‘’’
Результат выполнения запроса: 12.2

13. Выбор ограниченного количества возвращаемых строк

13.1 Топ-2 самых длинных фильма
‘’’ sql
	SELECT title, time_of_film
	FROM film
	ORDER BY time_of_film DESC
	LIMIT 2;
‘’’
Результат выполнения запроса: 13.1
13.2 Топ-2 самых длинных фильма с 1 пропуском
‘’’ sql
	SELECT title, time_of_film
	FROM film
	ORDER BY time_of_film DESC
	LIMIT 2 OFFSET 1;
‘’’
Результат выполнения запроса: 13.2

14. Соединение INNER JOIN

14.1 Вывести сеансы с названиями фильмов
‘’’ sql
	SELECT film.title, session_.date_time
	FROM session_
	INNER JOIN film ON session_.film_id = film.film_id;
‘’’
Результат выполнения запроса: 14.1
14.2 Вывести билеты с именами клиентов
‘’’ sql
	SELECT human.fio, ticket.row_, ticket.seat
	FROM ticket
	INNER JOIN human ON ticket.human_id = human.human_id;
‘’’
Результат выполнения запроса: 14.2

15. Соединение LEFT JOIN

15.1 Вывести все фильмы, даже без сеансов
‘’’ sql
	SELECT film.title, session_.date_time
	FROM film
	LEFT JOIN session_ ON film.film_id = session_.film_id;
‘’’
Результат выполнения запроса: 15.1
15.2 Вывести все сеансы, даже без проданных билетов
‘’’ sql
	SELECT session_.session_id, ticket.ticket_id, ticket.seat
	FROM session_
	LEFT JOIN ticket ON session_.session_id = ticket.session_id
‘’’
Результат выполнения запроса: 15.2

16. Соединение RIGHT JOIN

16.1 Вывести все фильмы, даже без сеансов
‘’’ sql
	SELECT session_.session_id, film.title
	FROM session_
	RIGHT JOIN film ON session_.film_id = film.film_id;
‘’’
Результат выполнения запроса: 16.1
16.2 Вывести всех клиентов, даже если билет не привязан
‘’’ sql
	SELECT ticket.ticket_id, human.fio
	FROM ticket
	RIGHT JOIN human ON ticket.human_id = human.human_id;
‘’’
Результат выполнения запроса: 16.2

17. Соединение OUTER JOIN

17.1 Вывести все фильмы и все сеансы
‘’’ sql
	SELECT film.title, session_.date_time
	FROM film
	FULL OUTER JOIN session_ ON film.film_id = session_.film_id;
‘’’
Результат выполнения запроса: 17.1
17.2 Вывести всех клиентов и все билеты
‘’’ sql
	SELECT human.fio, ticket.ticket_id, ticket.seat
	FROM human
	FULL OUTER JOIN ticket ON human.human_id = ticket.human_id;
‘’’
Результат выполнения запроса: 17.2

18. Соединение CROSS JOIN

18.1 Вывести все фильмы х все залы
‘’’ sql
	SELECT film.title, hall.num_hall
	FROM film
	CROSS JOIN hall;
‘’’
Результат выполнения запроса: 18.1
18.2 Вывести все фильмы х всех клиентов
‘’’ sql
	SELECT human.fio, film.title
	FROM human
	CROSS JOIN film;
‘’’
Результат выполнения запроса: 18.2

19. Запросы на выборку из нескольких таблиц

19.1 Вывести расписание - фильм, зал и дата
‘’’ sql
	SELECT film.title, hall.num_hall, session_.date_time
	FROM session_
	JOIN film ON session_.film_id = film.film_id
	JOIN hall ON session_.hall_id = hall.hall_id;
‘’’
Результат выполнения запроса: 19.1
19.2 Вывести кто купил билет и на какой фильм
‘’’ sql
	SELECT human.fio, film.title, ticket.row_, ticket.seat
	FROM ticket
	JOIN human ON ticket.human_id = human.human_id
	JOIN session_ ON ticket.session_id = session_.session_id
	JOIN film ON session_.film_id = film.film_id;
‘’’
Результат выполнения запроса: 19.2
