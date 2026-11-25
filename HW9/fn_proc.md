1. Процедуры 3 шт + запрос просмотра всех процедур

1.1 Процедура добавления нового актера
```sql
CREATE OR REPLACE PROCEDURE insert_into_actor(
	new_fio VARCHAR(200)
)
LANGUAGE plpgsql
AS $$
BEGIN
INSERT INTO actor (fio)
VALUES (new_fio);
END;
$$;

CALL insert_into_actor('Милли Бобби Браун');
SELECT * FROM actor;
```
Результат выполнения:
![alt text](<img/1.1.png>)

1.2 Процедура обновления числа рядов и мест в зале
```sql
CREATE OR REPLACE PROCEDURE update_hall(
	new_hall_id INT,
	new_num_row INT,
	new_num_seats INT
)
LANGUAGE plpgsql
AS $$
BEGIN
UPDATE hall
SET num_row = new_num_row,
num_seats = new_num_seats
WHERE hall_id = new_hall_id;
END;
$$;

CALL update_hall(1, 11, 210);
SELECT * FROM hall WHERE hall_id = 1;
```
Результат выполнения:
до
![alt text](<img/1.2begin.png>)
после 
![alt text](<img/1.2end.png>)

1.3 Процедура добавления нового фильма
```sql
CREATE OR REPLACE PROCEDURE insert_into_film(
	new_title VARCHAR(200),
	new_genre VARCHAR(50),
	new_time_of_film INT,
	new_age_rating VARCHAR(20),
	new_box_office DOUBLE PRECISION,
	new_earnings DOUBLE PRECISION
)
LANGUAGE plpgsql
AS $$
BEGIN
INSERT INTO film (title, genre, time_of_film, age_rating, box_office, earnings)
VALUES (new_title, new_genre, new_time_of_film, new_age_rating, new_box_office, new_earnings);
END;
$$;

CALL insert_into_film('Оппенгеймер', 'биография', 180, '18+', 100000000, 975000000);
SELECT * FROM film ;
```
Результат выполнения:
![alt text](<img/1.3.png>)

## Запрос просмотра всех процедур
```sql
SELECT routine_name, routine_type
FROM information_schema.routines
WHERE routine_type = 'PROCEDURE' AND routine_schema = 'public';
```
Результат выполнения:
![alt text](<img/1.png>)


2. Функции

2.1 Проверка на возраст фильма
```sql
CREATE OR REPLACE FUNCTION is_adult_film(age_rating VARCHAR(20))
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
BEGIN
RETURN age_rating IN ('18+', '18');
END;
$$;
```
Результат выполнения:
![alt text](<img/2.1.png>)

2.2 Разность earnings и box_office
```sql
CREATE OR REPLACE FUNCTION difference_box_and_earnings(box_office DOUBLE PRECISION, earnings DOUBLE PRECISION)
RETURNS DOUBLE PRECISION
LANGUAGE plpgsql
AS $$
BEGIN
RETURN earnings - box_office;
END;
$$;
```
Результат выполнения:
![alt text](<img/2.2.png>)

2.3 Проверка действителен ли билет
```sql
CREATE OR REPLACE FUNCTION is_active_ticket(sale_date DATE)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
BEGIN
	RETURN sale_date >= CURRENT_DATE;
END;
$$;
```
Результат выполнения:
![alt text](<img/2.3.png>)

2.4 Проверка действительности билета с выводом текста через переменную
```sql
CREATE OR REPLACE FUNCTION is_active_ticket_text(sale_date DATE)
RETURNS VARCHAR(50)
LANGUAGE plpgsql
AS $$
DECLARE
	active_ticket VARCHAR(50);
BEGIN
	IF sale_date >= CURRENT_DATE THEN
        active_ticket := 'Действителен';
    ELSE
        active_ticket := 'Не действителен';
    END IF;
	RETURN active_ticket;
	END;
$$;
```
Результат выполнения:
![alt text](<img/2.4.png>)

2.5 Длительность фильма
```sql
CREATE OR REPLACE FUNCTION is_long_film(time_of_film INT)
RETURNS VARCHAR(50)
LANGUAGE plpgsql
AS $$
DECLARE
	duration VARCHAR(50);
BEGIN
	IF time_of_film >= 150 THEN
        duration := 'Долгий';
	ELSIF time_of_film >= 110 AND time_of_film < 150 THEN
		duration := 'Средний';
    ELSE
        duration := 'Короткий';
    END IF;
	RETURN duration;
	END;
$$;
```
Результат выполнения:
![alt text](<img/2.5.png>)

2.6 Вместимость зала
```sql
CREATE OR REPLACE FUNCTION is_big_hall(num_row INT, num_seats INT)
RETURNS VARCHAR(50)
LANGUAGE plpgsql
AS $$
DECLARE
	size VARCHAR(50);
	count_seats INT;
BEGIN
	count_seats := num_row * num_seats;
	IF count_seats >= 500 THEN
        size := 'Большой';
	ELSIF count_seats >= 250 AND count_seats < 500 THEN
		size := 'Средний';
    ELSE
        size := 'Маленький';
    END IF;
	RETURN size;
	END;
$$;
```
Результат выполнения:
![alt text](<img/2.6.png>)

## Запрос просмотра всех функций
```sql
SELECT routine_name, routine_type
FROM information_schema.routines
WHERE routine_type = 'FUNCTION' AND routine_schema = 'public';
```
Результат выполнения:
![alt text](<img/2.png>)


3. Блок DO

3.1 Добавляем нового пользователя
```sql
DO $$ 
BEGIN 
IF NOT EXISTS (SELECT 1 FROM human WHERE email = 'humanity@gmail.com') THEN
INSERT INTO human (fio, email, birthday) VALUES ('Желтов Иван Ибрагимович', 'humanity@gmail.com', '1999-11-12');
END IF;
END;
$$ LANGUAGE plpgsql;
```
Результат выполнения:
![alt text](<img/3.1.png>)


4. IF

4.1 Длительность фильма
```sql
CREATE OR REPLACE FUNCTION is_long_film(time_of_film INT)
RETURNS VARCHAR(50)
LANGUAGE plpgsql
AS $$
DECLARE
	duration VARCHAR(50);
BEGIN
	IF time_of_film >= 150 THEN
        duration := 'Долгий';
	ELSIF time_of_film >= 110 AND time_of_film < 150 THEN
		duration := 'Средний';
    ELSE
        duration := 'Короткий';
    END IF;
	RETURN duration;
	END;
$$;
```
Результат выполнения:
![alt text](<img/2.5.png>)


5. CASE

5.1 Длительность фильма через CASE
```sql
CREATE OR REPLACE FUNCTION is_long_film(time_of_film INT)
RETURNS VARCHAR(50)
LANGUAGE plpgsql
AS $$
DECLARE
    duration VARCHAR(50);
BEGIN
    CASE
        WHEN time_of_film >= 150 THEN duration := 'Долгий';
        WHEN time_of_film >= 110 THEN duration := 'Средний';
        ELSE duration := 'Короткий';
    END CASE;

    RETURN duration;
END;
$$;
```
Результат выполнения:
![alt text](<img/2.5.png>)


6. WHILE

6.1 Создаем тестовых актеров
```sql
CREATE OR REPLACE PROCEDURE create_test_actor(count_actors INT)
LANGUAGE plpgsql
AS $$
DECLARE
	i INT := 1;
    new_actor_id INT := 9;
BEGIN
    WHILE i <= count_actors LOOP
		INSERT INTO actor (actor_id, fio)
		VALUES (new_actor_id + i, 'Актёр ' || i);
		i := i + 1;
	END LOOP;
	RAISE NOTICE 'Создано % тестовых актеров', count_actors;
END;
$$;

CALL create_test_actor(3);

SELECT * FROM actor WHERE fio LIKE 'Актёр%';
```
Результат выполнения:
![alt text](<img/while.png>)


7. EXCEPTION

7.1 Деление на ноль
```sql
DO $$ 
BEGIN 
SELECT 1 / 0;
EXCEPTION 
WHEN division_by_zero THEN
RAISE NOTICE 'На ноль делить нельзя';
END;
$$;
```
Результат выполнения:
![alt text](<img/exception.png>)


8. RAISE

8.1 Продажа билетов только совершеннолетним
```sql
DO $$
DECLARE
    age INT := 16;
BEGIN
    IF age < 18 THEN
        RAISE EXCEPTION 'Возраст % — продажа билета запрещена (нужно 18+)', age;
    END IF;
    RAISE NOTICE 'Билет продан';
END;
$$;
```
Результат выполнения:
![alt text](<img/raise.png>)