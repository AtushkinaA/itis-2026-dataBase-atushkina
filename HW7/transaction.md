1. Базовые операции с транзакциями 

1.1 Добавляем клиента и увеличиваем доход фильма / ошибка 
```sql
BEGIN;
INSERT INTO human (human_id, fio, email, birthday)
VALUES (15, 'Николаев Николай Николаевич', 'nikolaev@icloud.com', '1993-06-12');
UPDATE film
SET earnings = earnings + 500
WHERE film_id = 1;
COMMIT;
```
Результат выполнения запроса:
![alt text](<img/1.1.png>)
![alt text](<img/1.1 Film.png>)
![alt text](<img/1.1 human.png>)

```sql
BEGIN;
INSERT INTO human (fio, email, birthday)
VALUES ('Николаев Николай Николаевич', 'nikolaev@icloud.com', '1993-06-12');
UPDATE film
SET earnings = earnings + 500
WHERE film_id = 1;
ROLLBACK;
```
Результат выполнения запроса: изменений нет, данные остались те же, скриншоты идентичны

```sql
BEGIN;
INSERT INTO human (fio, email, birthday)
VALUES ('Иванов Иван Николаевич', 'ivanaev@icloud.com', '1993-06-12');
UPDATE film
SET earnings = earnings + 500
WHERE film_id = 3;
SELECT 1/0;
COMMIT;
```
Результат выполнения запроса: изменения не сохранены, новые данные не добавлены
    ERROR:  division by zero 
    SQL state: 22012


1.2 Добавляем новый зал и увеличиваем число мест в зале №1
```sql
BEGIN;
INSERT INTO hall (num_hall, num_row, num_seats)
VALUES (11, 10, 20);
UPDATE hall
SET num_seats = num_seats + 1
WHERE hall_id = 1;
COMMIT;
```
Результат выполнения запроса:
![alt text](<img/1.2 commit.png>)

```sql
BEGIN;
INSERT INTO hall (num_hall, num_row, num_seats)
VALUES (11, 10, 20);
UPDATE hall
SET num_seats = num_seats + 1
WHERE hall_id = 1;
ROLLBACK;
```
Результат выполнения запроса:
![alt text](<img/1.2 rollback.png>)

```sql
BEGIN;
INSERT INTO hall (num_hall, num_row, num_seats)
VALUES (11, 10, 20);
UPDATE hall
SET num_seats = num_seats + 1
WHERE hall_id = 1;
SELECT 1 / 0;
COMMIT;
```
Результат выполнения запроса:
изменения не сохранены, новые данные не добавлены
    ERROR:  division by zero 
    SQL state: 22012

Выводы: 
1) COMMIT - новые данные добавлены и изменены
2) ROLLBACK - новые данные не добавлены и не изменены
3) Ошибка - всё отменено и COMMIT игнорируется


2. READ UNCOMMITTED

2.1 Обновляем box_office у фильма с id = 2
```sql
--- T1
BEGIN;
UPDATE film
SET box_office = 410000000
WHERE film_id = 2;
--- данные не закоммичены, то есть не сохранятся, при SELECT ... выведится старое значение:
```
![alt text](<img/2.1.png>)
```sql
--- T2
BEGIN ISOLATION LEVEL READ UNCOMMITTED;
SELECT film_id, title, box_office
FROM film
WHERE film_id = 2;
ROLLBACK;
--- И всё равно покажет старое значение, в выводе: ROLLBACK Query returned successfully in 32 msec.
```
Результат выполнения запроса:
![alt text](<img/2.1.png>)

2.2 Обновляем у клиента email 
```sql
--- T1
BEGIN;
UPDATE human
SET email = 'new_email@icloud.com'
WHERE human_id = 1;
--- данные не закоммичены, то есть не сохранятся, при SELECT ... выведится старое значение:
```
![alt text](<img/2.2.png>)
```sql
--- T2
BEGIN ISOLATION LEVEL READ UNCOMMITTED;
SELECT human_id, fio, email
FROM human
WHERE human_id = 1;
ROLLBACK;
--- И всё равно покажет старое значение, в выводе: ROLLBACK Query returned successfully in 36 msec.
```
Результат выполнения запроса:
![alt text](<img/2.2.png>)


3. READ COMMITTED

3.1 Обновляем у клиента email 
```sql
--- T1
BEGIN ISOLATION LEVEL READ COMMITTED;
UPDATE human
SET email = 'read_test@icloud.com'
WHERE human_id = 1;
--- Результат: UPDATE 1. Транзакция открыта, изменения не закоммичены
```

```sql
--- T2
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT human_id, fio, email
FROM human
WHERE human_id = 1;
--- И всё равно покажет старое значение -  «грязные» данные не видны
```
Результат выполнения запроса:
![alt text](<img/2.2.png>)
```sql
--- T1
SELECT human_id, fio, email
FROM human
WHERE human_id = 1;
--- Результат: внутри своей транзакции изменения видны:
```
![alt text](<img/3.1.png>)

Результат:
![alt text](<img/3.1 итог.png>)

3.2 Обновляем у зала количество мест
```sql
-- T1
BEGIN ISOLATION LEVEL READ COMMITTED;
UPDATE hall
SET num_seats = num_seats + 5
WHERE hall_id = 1;
--- Результат: UPDATE 1. Транзакция открыта, изменения не закоммичены.
```

```sql
-- T2
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT hall_id, num_hall, num_row, num_seats
FROM hall
WHERE hall_id = 1;
--- «Грязные» данные не видны — T2 читает старую версию строки
```
![alt text](<img/3.2 T2.png>)

Проверяем, что внутри T1 изменения видны
```sql
-- T1 
SELECT hall_id, num_hall, num_row, num_seats
FROM hall
WHERE hall_id = 1;
---  Внутри своей транзакции T1 видит свои изменения, значения обновлено:
```
![alt text](<img/3.2 T1.png>)

Результат запросов:
![alt text](<img/3.2 T2.png>)

## Вывод:
# в PostgreSQL READ UNCOMMITTED фактически совпадает с READ COMMITTED. Поэтому пример с READ UNCOMMITTED и пример с READ COMMITTED дадут одинаковый результат — старые данные до коммита, новые после

# В READ COMMITTED «грязные» данные увидеть нельзя. T2 видит только закоммиченные данные. Незакоммиченные изменения T1 для T2 невидимы — T2 читает старую версию строки

3.3 Неповторяющееся чтения. 
1) Дважды читаем сборы фильма
```sql
-- T1
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT film_id, title, box_office
FROM film
WHERE film_id = 2;
```
Вывод
![alt text](<img/3.3 T1.png>)

```sql
-- T2
BEGIN;
UPDATE film
SET box_office = 500000000
WHERE film_id = 2;
COMMIT;
--- Результат: COMMIT, изменения закоммичены — теперь они видны всем
```
T1 — делаем второй SELECT
```sql
-- T1
SELECT film_id, title, box_office
FROM film
WHERE film_id = 2;
```
Вывод с новым значением:
![alt text](<img/3.3 после коммита.png>)

## Тот же самый запрос в той же самой транзакции вернул другое значение. Это и есть non-repeatable read — «неповторяющееся чтение».

2) Дважды читаем время сеанса
```sql
-- T1
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT session_id, film_id, hall_id, date_time
FROM session_
WHERE session_id = 1;
```
Вывод
![alt text](<img/3.3 2) до.png>)

```sql
-- T2
BEGIN;
UPDATE session_
SET date_time = '2023-12-15 20:00:00'
WHERE session_id = 1;
COMMIT;
--- Результат: COMMIT, изменения закоммичены — теперь они видны всем
```
T1 — делаем второй SELECT
```sql
-- T1
SELECT session_id, film_id, hall_id, date_time
FROM session_
WHERE session_id = 1;
```
Вывод с новым значением:
![alt text](<img/3.3 2) после.png>)


4. REPEATABLE READ

4.1 T1 не видит изменений T2

1) Обновляем box_office у фильма с id = 2
```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT film_id, title, box_office
FROM film
WHERE film_id = 2;
```
![alt text](<img/4.1 T1.png>)
```sql
-- T2
BEGIN;
UPDATE film
SET box_office = 600000000
WHERE film_id = 2;
COMMIT;
--- Результат: COMMIT, изменения закоммичены — для Т1 не видны
```
T1 — делаем второй SELECT
```sql
-- T1
SELECT film_id, title, box_office
FROM film
WHERE film_id = 2;
```
Вывод с новым значением:
![alt text](<img/4.1 T1.png>)
Делаем ROLLBACK для Т1 и получаем измененные данные
![alt text](<img/4.1 1).png>)

2) Обновляем email у клиента с id = 1
```sql
--  T1
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT human_id, fio, email
FROM human
WHERE human_id = 1;
```
![alt text](<img/4.1 2)T1.png>)
```sql
-- T2
BEGIN;
UPDATE human
SET email = 'repeatable_test@icloud.com'
WHERE human_id = 1;
COMMIT;
--- Результат: COMMIT, изменения закоммичены — для Т1 не видны
```
T1 — делаем второй SELECT
```sql
-- T1
SELECT human_id, fio, email
FROM human
WHERE human_id = 1;
```
Вывод с новым значением:
![alt text](<img/4.1 2)T1.png>)
Делаем ROLLBACK для Т1 и получаем измененные данные
![alt text](<img/4.1 2).png>)

4.2 Фантомное чтение через INSERT в T2

1) Добавляем новый хоррор фильм
```sql
-- Т1
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT film_id, title, genre, box_office
FROM film
WHERE genre = 'horror'
ORDER BY film_id; 
-- Результат 7 строк
```
![alt text](<img/4.2 1)1.png>)
```sql
-- T2
BEGIN;
INSERT INTO film (title, genre, time_of_film, age_rating, box_office, earnings)
VALUES ('The Conjuring 2', 'horror', 134, '18+', 320000000, 321000000);
COMMIT;
--- Результат: COMMIT, добавления закоммичены — для Т1 не видны
```
T1 — делаем второй SELECT
```sql
-- T1
SELECT film_id, title, genre, box_office
FROM film
WHERE genre = 'horror'
ORDER BY film_id;
```
Вывод с новой строкой:
![alt text](<img/4.2 1)1.png>)
Делаем ROLLBACK для Т1 и получаем новые данные + 1 фильм
![alt text](<img/4.2 1)2.png>)

2) Добавляем нового клиента
```sql
-- Т1
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT human_id, fio, email
FROM human
WHERE email LIKE '%@gmail.com'
ORDER BY human_id;
-- Результат 5 строк
```
![alt text](<img/4.2 2)1.png>)
```sql
-- T2
BEGIN;
INSERT INTO human (fio, email, birthday)
VALUES ('Фантомный Клиент', 'phantom@gmail.com', '1995-05-05');
COMMIT;
--- Результат: COMMIT, добавления закоммичены — для Т1 не видны
```
T1 — делаем второй SELECT
```sql
-- T1
SELECT human_id, fio, email
FROM human
WHERE email LIKE '%@gmail.com'
ORDER BY human_id;
```
Вывод с новой строкой:
![alt text](<img/4.2 2)1.png>)
Делаем ROLLBACK для Т1 и получаем новые данные + 1 клиент
![alt text](<img/4.2 2)2.png>)

## Вывод:
# В PostgreSQL на уровне REPEATABLE READ фантомное чтение НЕ происходит. T1 видит снимок данных на момент начала транзакции. Новая строка T2 (даже закоммиченная) для T1 невидима, пока T1 не завершится.


5. SERIALIZABLE

5.1 Смоделируй конфликт: две транзакции вставляют одинаковые данные.

1) Две транзакции одновременно пытаются добавить зал с одинаковым номером
```sql
-- Т1
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT num_hall FROM hall WHERE num_hall = 99;
INSERT INTO hall (num_hall, num_row, num_seats) VALUES (99, 10, 20);
```
```sql
-- T2
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT num_hall FROM hall WHERE num_hall = 99;
INSERT INTO hall (num_hall, num_row, num_seats) VALUES (99, 12, 25);
```
T1 — делаем COMMIT
T2 — делаем COMMIT и получаем ошибку: ERROR:  could not serialize access due to read/write dependencies among transactions
Reason code: Canceled on identification as a pivot, during commit attempt. 

Проверим, что в итоге получили
```sql
SELECT * FROM hall WHERE num_hall = 99;
```
Результат: 
![alt text](<img/5.1 1).png>)

2) Две транзакции одновременно пытаются изменить earnings у фильма
```sql
-- Т1
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT film_id, title, earnings
FROM film
WHERE film_id = 1;
```
```sql
-- T2
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT film_id, title, earnings
FROM film
WHERE film_id = 1;
```
Оба выводят: 
![alt text](<img/5.2 1.png>)
T1 — делаем UPDATE и COMMIT
```sql
UPDATE film
SET earnings = earnings + 1000
WHERE film_id = 1;
COMMIT;
-- earnings увеличился
```
T2 — делаем UPDATE и получаем ошибку: ERROR:  could not serialize access due to concurrent update  
T2 — откатываем и повторяем транзакцию: ROLLBACK;
```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT film_id, title, earnings
FROM film
WHERE film_id = 1;
UPDATE film
SET earnings = earnings + 2000
WHERE film_id = 1;
COMMIT;
--  Теперь всё прошло успешно — T2 увидел свежее значение и добавил свои 2000.
```
Проверим, что в итоге получили
```sql
SELECT film_id, title, earnings
FROM film
WHERE film_id = 1;
```
Результат: 
![alt text](<img/5.2 2.png>)

## Вывод:
# SERIALIZABLE — максимальная изоляция: результат параллельных транзакций такой же, как при последовательном выполнении. Исключает все аномалии, но при конфликте прерывает транзакцию ошибкой 40001, которую приложение должно повторять.


6. SAVEPOINT

6.1 добавляем двух клиентов. После первого — ставим SAVEPOINT. После второго — откатываемся к точке. Проверяем, что первый клиент остался, а второй — нет
```sql
BEGIN;
INSERT INTO human (fio, email, birthday)
VALUES ('Первый Клиент', 'first@example.com', '1990-01-01');
SAVEPOINT after_first;
INSERT INTO human (fio, email, birthday)
VALUES ('Второй Клиент', 'second@example.com', '1991-02-02');
ROLLBACK TO SAVEPOINT after_first;
COMMIT;
```
Результат выполнения:
![alt text](<img/6.1.png>)
Второго клиента нет — он был отменён ROLLBACK TO SAVEPOINT. Первый — есть, потому что COMMIT зафиксировал всё, что было до точки.

6.2 добавляем трёх клиентов. Ставим SAVEPOINT после первого и после второго. Затем откатываемся сначала ко второй точке (отменяем третьего), потом к первой (отменяем второго). Проверяем, что остался только первый
```sql
BEGIN;
INSERT INTO human (fio, email, birthday)
VALUES ('Клиент A', 'a@example.com', '1990-01-01');
SAVEPOINT point_1;
INSERT INTO human (fio, email, birthday)
VALUES ('Клиент B', 'b@example.com', '1991-02-02');
SAVEPOINT point_2;
INSERT INTO human (fio, email, birthday)
VALUES ('Клиент C', 'c@example.com', '1992-03-03');
ROLLBACK TO SAVEPOINT point_2;
ROLLBACK TO SAVEPOINT point_1;
COMMIT;
```
Результат выполнения:
![alt text](<img/6.2.png>)
Клиенты B и C отменены. Остался только A — он был добавлен до первой точки сохранения.

6.3 администратор кинотеатра корректирует сборы фильма, затем добавляет новый фильм, потом ставит точку сохранения и делает ещё одно обновление. После этого откатывается к точке — и проверяет, что осталось
```sql
BEGIN;
UPDATE film
SET box_office = 710000000
WHERE film_id = 1;
INSERT INTO film (title, genre, time_of_film, age_rating, box_office, earnings)
VALUES ('The Conjuring 3', 'horror', 112, '18+', 200000000, 202000000);
SAVEPOINT after_insert;
UPDATE film
SET box_office = 420000000
WHERE film_id = 2;
ROLLBACK TO SAVEPOINT after_insert;
COMMIT;
```
Результат выполнения:
Сборы фильма с film_id = 2 — не должны быть обновлены:
![alt text](<img/6.3.png>)
Новый фильм — должен быть добавлен:
![alt text](<img/6.3 2.png>)
Сборы IT — должны быть обновлены:
![alt text](<img/6.3 3.png>)

6.4 администратор меняет количество мест в зале, добавляет два новых зала, ставит две точки сохранения и откатывается к каждой из них.
```sql
BEGIN;
UPDATE hall
SET num_seats = num_seats + 5
WHERE hall_id = 1;
SAVEPOINT point_1;
INSERT INTO hall (num_hall, num_row, num_seats)
VALUES (11, 10, 20);
SAVEPOINT point_2;
INSERT INTO hall (num_hall, num_row, num_seats)
VALUES (12, 8, 15);
ROLLBACK TO SAVEPOINT point_2;
ROLLBACK TO SAVEPOINT point_1;
COMMIT;
```
Результат выполнения:
11 и 12 зала нет
Зал с id=1 обновлен по количеству мест на 5
![alt text](<img/6.4.png>)

# Вывод:
## SAVEPOINT позволяет откатывать часть транзакции, не отменяя её целиком. Всё, что было до точки сохранения, остаётся. Всё, что после — отменяется при ROLLBACK TO SAVEPOINT. Это удобно, когда нужно «попробовать» изменение и отменить его, не теряя уже сделанную работу.


# Как работают ROLLBACK и SAVEPOINT на моих данных:
## ROLLBACK отменяет всю транзакцию: и INSERT в human, и UPDATE в film/hall — данные возвращаются к исходному состоянию.
## ROLLBACK TO SAVEPOINT отменяет только изменения после точки: клиенты/залы, добавленные до SAVEPOINT, остаются, а добавленные после удаляются.
## COMMIT после ROLLBACK TO SAVEPOINT фиксирует то, что осталось до точки.
