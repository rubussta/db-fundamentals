# Решения задач на PostgreSQL из учебного пособия [Новиков Б. А. "Основы технологий баз данных"](https://postgrespro.ru/education/books/dbtech)  

[<< К описанию репозитория](https://github.com/rubussta/db-fundamentals)

## Глава 4. Введение в SQL
<details>
<summary>Упражнение 4.3. Основные конструкции и синтаксис. Фильтрация и упорядочивание #WHERE #ORDER BY </summary>
Найдите все самолеты c максимальной дальностью полета:
1) либо больше 10 000 км, либо меньше 4 000 км;
2) больше 6 000 км, а название не заканчивается на «100».
Обратите внимание на порядок следования предложений WHERE и FROM.

  **Решение 4.3:**

```SQL
SELECT * 
	FROM aircrafts
	WHERE range > 4000 AND range < 10000
	ORDER BY range;

aircraft_code|model          |range|
-------------+---------------+-----+
733          |Boeing 737-300 | 4200|
321          |Airbus A321-200| 5600|
320          |Airbus A320-200| 5700|
319          |Airbus A319-100| 6700|
763          |Boeing 767-300 | 7900|

5 row(s) fetched.

> SELECT * 
	FROM aircrafts
	WHERE range > 6000 AND model !~ '100$' 
	ORDER BY range;

aircraft_code|model         |range|
-------------+--------------+-----+
763          |Boeing 767-300| 7900|
773          |Boeing 777-300|11100|

2 row(s) fetched.
```

</details>
<details>
<summary>Упражнение 4.4. Основные конструкции и синтаксис. Операции с датами, приведение типов #to_char #::</summary>
Определите номера и время отправления всех рейсов, прибывших в аэропорт назначения не вовремя.
  
  **Решение 4.4:**

```SQL
-- Рейс прибыл в аэропорт, если у него есть фактическое время прибытия.
-- Рейс пришел не вовремя, если разница между фактическим и временем
-- по расписанию не равна нулю. Дополнительно вычислим время задержки рейса.

SELECT flight_no , actual_departure , actual_arrival , actual_arrival - scheduled_arrival AS delay  
	FROM flights
	WHERE actual_arrival IS NOT NULL 
AND actual_arrival - scheduled_arrival <> '00:00:00'
	ORDER BY delay DESC
	LIMIT 5;

flight_no|actual_departure             |actual_arrival               |delay   |
---------+-----------------------------+-----------------------------+--------+
PG0073   |2015-11-02 16:53:00.000 +0400|2015-11-02 21:52:00.000 +0400|05:07:00|
PG0040   |2015-12-15 18:29:00.000 +0400|2015-12-15 18:54:00.000 +0400|04:44:00|
PG0533   |2015-12-22 19:37:00.000 +0400|2015-12-22 22:59:00.000 +0400|04:44:00|
PG0132   |2016-07-20 17:06:00.000 +0400|2016-07-20 18:36:00.000 +0400|04:41:00|
PG0531   |2016-07-15 13:56:00.000 +0400|2016-07-15 16:26:00.000 +0400|04:41:00|
5 row(s) fetched.

> -- Другой вариант запроса. Тип time приводится к целочисленному.
-- В шаблон включен день месяца, если рейс задерживается больше суток.
SELECT flight_no , actual_departure , actual_arrival , actual_arrival - scheduled_arrival AS delay  
	FROM flights
	WHERE actual_arrival IS NOT NULL
AND to_char(actual_arrival, 'DDHH24MI')::int - to_char(scheduled_arrival, 'DDHH24MI')::int <> 0
	ORDER BY delay DESC
	LIMIT 5;


flight_no|actual_departure             |actual_arrival               |delay   |
---------+-----------------------------+-----------------------------+--------+
PG0073   |2015-11-02 16:53:00.000 +0400|2015-11-02 21:52:00.000 +0400|05:07:00|
PG0040   |2015-12-15 18:29:00.000 +0400|2015-12-15 18:54:00.000 +0400|04:44:00|
PG0533   |2015-12-22 19:37:00.000 +0400|2015-12-22 22:59:00.000 +0400|04:44:00|
PG0132   |2016-07-20 17:06:00.000 +0400|2016-07-20 18:36:00.000 +0400|04:41:00|
PG0531   |2016-07-15 13:56:00.000 +0400|2016-07-15 16:26:00.000 +0400|04:41:00|

5 row(s) fetched.
```

</details>
<details>
<summary>Упражнение 4.7. Соединения. #JOIN</summary>
Напечатанный посадочный талон должен содержать фамилию и имя пассажира, коды аэропортов вылета и прилета, дату и время вылета и прилета по расписанию, номер места в салоне самолета. Напишите запрос, выводящий всю необходимую информацию для полученных посадочных талонов на рейсы, которые еще не вылетели.

**Решение 4.7:**

```SQL
SELECT 
t.passenger_name ,
f.departure_airport ,
f.arrival_airport ,
f.scheduled_departure ,
f.scheduled_arrival ,
bp.seat_no 
FROM ticket_flights AS tf
	LEFT JOIN tickets AS t ON tf.ticket_no = t.ticket_no -- ФИО пассажира
INNER JOIN boarding_passes AS bp ON tf.ticket_no = bp.ticket_no AND tf.flight_id = bp.flight_id -- Номер места в салоне самолета после регистрации
LEFT JOIN flights AS f ON tf.flight_id = f.flight_id -- Коды аэропортов, дата и время вылета и прилета по расписанию
WHERE EXISTS (SELECT f.actual_departure FROM flights WHERE f.actual_departure IS NULL) -- Исключаем вылетевшие рейсы
LIMIT 5;

passenger_name   |departure_airport|arrival_airport|scheduled_departure          |scheduled_arrival            |seat_no|
-----------------+-----------------+---------------+-----------------------------+-----------------------------+-------+
ALEKSANDR VASILEV|PEE              |VKO            |2016-10-13 18:55:00.000 +0400|2016-10-13 20:20:00.000 +0400|35E    |
VERA IVANOVA     |AER              |SVO            |2016-10-13 18:15:00.000 +0400|2016-10-13 20:00:00.000 +0400|35E    |
EMILIYA BORISOVA |PEE              |VKO            |2016-10-13 18:55:00.000 +0400|2016-10-13 20:20:00.000 +0400|35F    |
LYUDMILA ROMANOVA|AER              |SVO            |2016-10-13 18:15:00.000 +0400|2016-10-13 20:00:00.000 +0400|35F    |
PETR TIKHONOV    |PEE              |VKO            |2016-10-13 18:55:00.000 +0400|2016-10-13 20:20:00.000 +0400|35G    |

5 row(s) fetched.
```

</details>
<details>
<summary>Упражнение 4.9. Соединения. Вывод отсутствующих значений, CTE #JOIN #EXCEPT #WITH</summary>
Выведите номера мест, оставшихся свободными в рейсах из Анапы (AAQ) в Шереметьево (SVO), вместе с номером рейса и его датой.

**Решение 4.9:**

```SQL
WITH sales AS 
(
	SELECT -- Рейсы, на которые проданы билеты
	DISTINCT tf.flight_id ,
	f.departure_airport ,
	f.arrival_airport , 
	f.scheduled_departure ,
	f.scheduled_arrival ,
	f.flight_no ,
	f.aircraft_code 
	FROM ticket_flights AS tf -- Билет
		JOIN boarding_passes AS bp ON tf.ticket_no = bp.ticket_no AND tf.flight_id = bp.flight_id -- Номер места
		JOIN flights AS f ON tf.flight_id = f.flight_id -- Код рейса, код самолета, код аэропорта, дата рейса
		WHERE f.departure_airport = 'AAQ' AND f.arrival_airport = 'SVO' -- Рейс из Анапы (AAQ) в Шереметьево (SVO)
)
SELECT -- Номера всех мест на рейсах в соответствии в компановкой салона самолета
sales.departure_airport ,
sales.arrival_airport , 
sales.scheduled_departure ,
sales.scheduled_arrival ,
sales.flight_no ,
s.seat_no AS not_soled_seat
FROM sales
	JOIN seats AS s ON sales.aircraft_code = s.aircraft_code 
		AND s.aircraft_code  IN (SELECT DISTINCT aircraft_code -- Код модели самолета
		FROM flights WHERE departure_airport = 'AAQ' AND arrival_airport = 'SVO')
EXCEPT 
SELECT -- Номера мест на рейсах, на которые проданы билеты
f.departure_airport ,
f.arrival_airport , 
f.scheduled_departure ,
f.scheduled_arrival ,
f.flight_no ,
bp.seat_no
	FROM ticket_flights AS tf -- Билет
		JOIN boarding_passes AS bp ON tf.ticket_no = bp.ticket_no AND tf.flight_id = bp.flight_id -- Номер места
		JOIN flights AS f ON tf.flight_id = f.flight_id -- Код рейса, код самолета, код аэропорта, дата рейса
		WHERE f.departure_airport = 'AAQ' AND f.arrival_airport = 'SVO' -- Рейс из Анапы (AAQ) в Шереметьево (SVO)
	ORDER BY scheduled_departure , not_soled_seat
LIMIT 5;

departure_airport|arrival_airport|scheduled_departure          |scheduled_arrival            |flight_no|not_soled_seat|
-----------------+---------------+-----------------------------+-----------------------------+---------+--------------+
AAQ              |SVO            |2015-10-14 13:05:00.000 +0400|2015-10-14 14:45:00.000 +0400|PG0252   |10C           |
AAQ              |SVO            |2015-10-14 13:05:00.000 +0400|2015-10-14 14:45:00.000 +0400|PG0252   |10D           |
AAQ              |SVO            |2015-10-14 13:05:00.000 +0400|2015-10-14 14:45:00.000 +0400|PG0252   |11A           |
AAQ              |SVO            |2015-10-14 13:05:00.000 +0400|2015-10-14 14:45:00.000 +0400|PG0252   |11B           |
AAQ              |SVO            |2015-10-14 13:05:00.000 +0400|2015-10-14 14:45:00.000 +0400|PG0252   |11D           |

5 row(s) fetched.
```

</details>
<details>
<summary>Упражнение 4.11. Агрегирование и группировка. #GROUP BY #avg()</summary>
Напишите запрос, возвращающий среднюю стоимость авиабилета в каждом из классов перевозки. Модифицируйте его таким образом, чтобы было видно, какому классу какое значение соответствует.

**Решение 4.11:**

```SQL
SELECT fare_conditions, avg(amount) AS avg_ticket_cost
  FROM ticket_flights
  GROUP BY fare_conditions;

fare_conditions|avg_ticket_cost   |
---------------+------------------+
Business       |51557.399820393274|
Comfort        |32724.546136534134|
Economy        |16031.309072998395|

3 row(s) fetched.
```

</details>
<details>
<summary>Упражнение 4.15. Модификация данных. #CREATE TEMP TABLE #UPDATE #SET #RETURNING #POSIX</summary>
В результате еще одной модернизации в самолетах «Аэробус A319» (код 319) ряды кресел с шестого по восьмой были переведены в разряд бизнес-класса. Измените таблицу одним запросом и получите измененные данные с помощью предложения RETURNING.

**Решение 4.15:**

```SQL
CREATE TEMP TABLE seats_tmp AS  
  SELECT * FROM seats
  WHERE aircraft_code = '319';

116 row(s) modified.


UPDATE seats_tmp
	SET fare_conditions = 'Business'
  	WHERE seat_no ~ '^(6|7|8)'
	RETURNING seat_no , fare_conditions;

seat_no|fare_conditions|
-------+---------------+
6A     |Business       |
6B     |Business       |
6C     |Business       |
6D     |Business       |
6E     |Business       |
6F     |Business       |
7A     |Business       |
7B     |Business       |
7C     |Business       |
7D     |Business       |
7E     |Business       |
7F     |Business       |
8A     |Business       |
8B     |Business       |
8C     |Business       |
8D     |Business       |
8F     |Business       |
8E     |Business       |

18 row(s) fetched.
```

</details>
<details>
<summary>Упражнение 4.20. Вложенные подзапросы. #FROM(SELECT FROM) #JOIN</summary>
Найдите модели самолетов «дальнего следования», максимальная продолжительность рейсов которых составила более 6 часов.

**Решение 4.20:**

```SQL
SELECT ac.aircraft_code , a.model
FROM (SELECT DISTINCT aircraft_code FROM flights
	WHERE scheduled_arrival > scheduled_departure + '6 hours' -- Рейс летит более 6 часов по расписанию 
	AND actual_arrival IS NOT NULL) AS ac -- Рейс уже приземлился 
JOIN aircrafts AS a ON ac.aircraft_code = a.aircraft_code;

aircraft_code|model          |
-------------+---------------+
763          |Boeing 767-300 |
319          |Airbus A319-100|

2 row(s) fetched.
```

</details>

## Глава 5. Управление доступом в базах данных
<details>
<summary>Упражнение 5.1. Групповые политики. #\set PROMPT1 %n@%/%R%# #CREATE ROLE #GRANT</summary>
Создайте роль для доступа на чтение к демонстрационной базе данных без права создания сеансов работы с сервером БД.

  **Решение 5.1:**

```SQL
demo=# \set PROMPT1 %n@%/%R%#
postgres@demo=#CREATE ROLE db_reader;
CREATE ROLE

postgres@demo=#GRANT SELECT ON ALL TABLES IN SCHEMA bookings TO db_reader;
GRANT

--Проверяем есть ли доступ на примере одной из таблиц БД.
postgres@demo=#\dp aircrafts

                                    Access privileges
  Schema  |   Name    | Type  |     Access privileges     | Column privileges | Policies
----------+-----------+-------+---------------------------+-------------------+----------
 bookings | aircrafts | table | postgres=arwdDxt/postgres+|                   |
          |           |       | db_reader=r/postgres      |                   |
(1 row)

-- У таблицы есть права на чтение (r)для роли db_reader, предоставленные суперпользователем (db_reader=r/postgres).
-- Проверяем есть ли у роли права на подключение к БД.

postgres@demo=#\du db_reader
            List of roles
 Role name |  Attributes  | Member of
-----------+--------------+-----------
 db_reader | Cannot login | {}

--Права на подключение отсутствуют, поскольку при создании роли не был указан оператор LOGIN и по-умолчанию принимается NOLOGIN, не дающий право на подключение и данная роль с этим оператором не может быть начальным авторизованным именем при подключении клиента.
```

</details>
<details>
<summary>Упражнение 5.2. Групповые политики. #CREATE USER #ALTER USER #GRANT</summary>
Создайте пользователя сервера БД и предоставьте ему привилегию использования роли, созданной в предыдущем упражнении.  
Проверьте, что этот пользователь может выполнять любые запросы на выборку из таблиц демонстрационной базы данных, но не может их обновлять.

  **Решение 5.2:**

```SQL
postgres@demo=#CREATE USER db_user;
CREATE ROLE

postgres@demo=#ALTER USER db_user WITH PASSWORD 'userpass';
ALTER ROLE

postgres@demo=#\du db_user
           List of roles
 Role name | Attributes | Member of
-----------+------------+-----------
 db_user   |            | {}

postgres@demo=# GRANT CONNECT ON DATABASE demo TO db_user;
GRANT

postgres@demo=#GRANT USAGE ON SCHEMA bookings TO db_user;
GRANT

postgres=# GRANT db_reader TO db_user;
GRANT ROLE

db_user@demo=>\du db_user
            List of roles
 Role name | Attributes |  Member of
-----------+------------+-------------
 db_user   |            | {db_reader}

postgres@demo=#\c demo db_user
Password for user db_user:
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "demo" as user "db_user".

db_user@demo=>SELECT * FROM bookings.aircrafts;

 aircraft_code |        model        | range
---------------+---------------------+-------
 773           | Boeing 777-300      | 11100
 763           | Boeing 767-300      |  7900
 SU9           | Sukhoi SuperJet-100 |  3000
 320           | Airbus A320-200     |  5700
 321           | Airbus A321-200     |  5600
 319           | Airbus A319-100     |  6700
 733           | Boeing 737-300      |  4200
 CN1           | Cessna 208 Caravan  |  1200
 CR2           | Bombardier CRJ-200  |  2700
(9 rows)

db_user@demo=>UPDATE bookings.aircrafts SET range = 11000 WHERE range = 11100;
ERROR:  permission denied for table aircrafts
db_user@demo=>
```

</details>
<details>
<summary>Упражнение 5.3. Групповые политики. Отзыв привелегий. #REVOKE</summary>
Заберите у пользователя привилегию, выданную в предыдущем упражнении. 
Убедитесь, что этот пользователь не сможет выбирать данные из таблиц демобазы.

  **Решение 5.3:**

```SQL
postgres@demo=#REVOKE db_reader FROM db_user;
REVOKE ROLE
postgres@demo=#\du db_user
           List of roles
 Role name | Attributes | Member of
-----------+------------+-----------
 db_user   |            | {}

postgres@demo=#\c demo db_user
Password for user db_user:
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "demo" as user "db_user".

db_user@demo=>SELECT * FROM bookings.aircrafts;
ERROR:  permission denied for table aircrafts
db_user@demo=>
```

</details>

## Глава 6. Транзакции и согласованность базы данных
<details>
<summary>Упражнение 6.1. Пример транзакции. Начало и откат. #BEGIN #ROLLBACK</summary>
Начните транзакцию (командой BEGIN) и создайте новое бронирование в таблице bookings сегодняшней датой. Добавьте два электронных билета в таблицу tickets, связанных с созданным бронированием. Представьте, что пользователь не подтвердил бронирование и все введенные данные необходимо отменить. Выполните отмену транзакции и проверьте, что никакой добавленной вами информации действительно не осталось.

  **Решение 6.1:**

```SQL
demo=# BEGIN;
BEGIN
demo=*# INSERT INTO bookings (book_ref, book_date, total_amount)
VALUES ('999999', CURRENT_TIMESTAMP, 12000);
INSERT 0 1
demo=*# SELECT * FROM bookings
WHERE book_ref = '999999';
 book_ref |           book_date           | total_amount
----------+-------------------------------+--------------
 999999   | 2023-11-20 20:02:40.202792+04 |     12000.00
(1 row)

demo=*# INSERT INTO tickets (ticket_no, book_ref, passenger_id, passenger_name, contact_data)
VALUES (000999999998, '999999', '1111 111111', 'IVAN IVANOV', '{"phone": "+79999999999"}'::jsonb),
(000999999999, '999999', '2222 222222', 'PETR IVANOV', '{"phone": "+79999999999"}'::jsonb);
INSERT 0 2
demo=*# SELECT * FROM tickets
WHERE book_ref = '999999';
   ticket_no   | book_ref | passenger_id | passenger_name |       contact_data
---------------+----------+--------------+----------------+---------------------------
 999999998     | 999999   | 1111 111111  | IVAN IVANOV    | {"phone": "+79999999999"}
 999999999     | 999999   | 2222 222222  | PETR IVANOV    | {"phone": "+79999999999"}
(2 rows)

demo=*# ROLLBACK;
ROLLBACK
demo=# SELECT * FROM bookings
WHERE book_ref = '999999';
 book_ref | book_date | total_amount
----------+-----------+--------------
(0 rows)

demo=# SELECT * FROM tickets
WHERE book_ref = '999999';
 ticket_no | book_ref | passenger_id | passenger_name | contact_data
-----------+----------+--------------+----------------+--------------
(0 rows)

demo=#
```

</details>
<details>
<summary>Упражнение 6.2. Пример транзакции. Точка сохранения. #BEGIN #SAVEPOINT #ROLLBACK TO SAVEPOINT</summary>
Теперь представьте сценарий, в котором нужно отменить не все данные, а только последний из добавленных электронных билетов. Для этого повторите все действия из предыдущего упражнения, но перед добавлением каждого билета создавайте точку сохранения (с одним и тем же именем). После ввода второго билета выполните откат к точке сохранения. Проверьте, что бронирование и первый билет остались.

  **Решение 6.2:**

```SQL
demo=# BEGIN;
BEGIN

demo=*# INSERT INTO bookings (book_ref, book_date, total_amount)
VALUES ('999999', CURRENT_TIMESTAMP, 12000);
INSERT 0 1

demo=*# SAVEPOINT my_svp;
SAVEPOINT
demo=*# INSERT INTO tickets (ticket_no, book_ref, passenger_id, passenger_name, contact_data)
VALUES (000999999998, '999999', '1111 111111', 'IVAN IVANOV', '{"phone": "+79999999999"}'::jsonb);
INSERT 0 1

demo=*# SAVEPOINT my_svp;
SAVEPOINT

demo=*# INSERT INTO tickets (ticket_no, book_ref, passenger_id, passenger_name, contact_data)
VALUES (000999999999, '999999', '2222 222222', 'PETR IVANOV', '{"phone": "+79999999999"}'::jsonb);
INSERT 0 1

demo=*# ROLLBACK TO SAVEPOINT my_svp;
ROLLBACK

demo=*# SELECT * FROM bookings -- Проверка, что бронирование осталось
WHERE book_ref = '999999';

 book_ref |           book_date           | total_amount
----------+-------------------------------+--------------
 999999   | 2023-11-21 14:15:36.686815+04 |     12000.00
(1 row)

demo=*# SELECT * FROM tickets -- Проверка, что первый билет 000999999998 остался
WHERE book_ref = '999999';

   ticket_no   | book_ref | passenger_id | passenger_name |       contact_data
---------------+----------+--------------+----------------+---------------------------
 999999998     | 999999   | 1111 111111  | IVAN IVANOV    | {"phone": "+79999999999"}
(1 row)
```

</details>
<details>
<summary>Упражнение 6.3. Пример транзакции. Фиксация. Откат к точке сохранения. #COMMIT #ROLLBACK TO SAVEPOINT</summary>
В рамках той же транзакции добавьте еще один электронный билет и зафиксируйте транзакцию. 
Обратите внимание на то, что после этой операции отменить внесенные транзакцией изменения будет уже невозможно.

  **Решение 6.3:**

```SQL
demo=*# INSERT INTO tickets (ticket_no, book_ref, passenger_id, passenger_name, contact_data)
VALUES (000999999999, '999999', '2222 222222', 'PETR IVANOV', '{"phone": "+79999999999"}'::jsonb);
INSERT 0 1

demo=*# COMMIT;
COMMIT

demo=# SELECT * FROM bookings -- Проверка, что бронирование осталось
WHERE book_ref = '999999';

 book_ref |           book_date           | total_amount
----------+-------------------------------+--------------
 999999   | 2023-11-21 14:15:36.686815+04 |     12000.00
(1 row)

demo=# SELECT * FROM tickets -- Проверка, что билеты вставились
WHERE book_ref = '999999';

   ticket_no   | book_ref | passenger_id | passenger_name |       contact_data
---------------+----------+--------------+----------------+---------------------------
 999999998     | 999999   | 1111 111111  | IVAN IVANOV    | {"phone": "+79999999999"}
 999999999     | 999999   | 2222 222222  | PETR IVANOV    | {"phone": "+79999999999"}
(2 rows)

demo=# ROLLBACK TO SAVEPOINT my_svp; -- Пробуем откатиться к первой точке сохранения после завершения транзакции
ERROR:  ROLLBACK TO SAVEPOINT can only be used in transaction blocks
demo=# -- Точки сохранения существуют только в рамках открытой транзакции и удаляются после ее завершения.
```

</details>
<details>
<summary>Упражнение 6.4. Уровень изоляции Read Committed. #SHOW transaction_isolation #BEGIN #UPDATE #COMMIT</summary>
Перед началом выполнения задания проверьте, что в таблице bookings нет бронирований на сумму total_amount 1 000 рублей. 
1. В первом сеансе начните транзакцию (командой BEGIN). Выполните обновление таблицы bookings: увеличьте total_amount в два раза в тех строках, где сумма равна 1 000 рублей. 
2. Во втором сеансе (откройте новое окно psql) вставьте в таблицу bookings новое бронирование на 1 000 рублей и зафиксируйте транзакцию. 
3. В первом сеансе повторите обновление таблицы bookings и зафиксируйте транзакцию. Осталась ли сумма добавленного бронирования равной 1 000 рублей? Почему это не так?

  **Решение 6.4:**

```SQL
demo=# SELECT * FROM bookings
WHERE total_amount = 1000;

 book_ref | book_date | total_amount
----------+-----------+--------------
(0 rows)

demo=# SHOW transaction_isolation;

 transaction_isolation
-----------------------
 read committed
(1 row)

demo=# BEGIN; -- Начало первой транзакции
BEGIN

demo=*# UPDATE bookings
SET total_amount = total_amount * 2
WHERE total_amount = 1000;
UPDATE 0

demo=# BEGIN; -- Начало второй транзакции
BEGIN

demo=*# INSERT INTO bookings (book_ref, book_date, total_amount)
VALUES ('999999', CURRENT_TIMESTAMP, 1000);
INSERT 0 1

demo=*# COMMIT; -- Фиксация второй транзакции
COMMIT

demo=*# UPDATE bookings -- Обновление таблицы в первой транзакции
SET total_amount = total_amount * 2
WHERE total_amount = 1000;
UPDATE 1

demo=*# COMMIT; -- Фиксация первой транзакции
COMMIT

demo=# SELECT * FROM bookings -- Проверка суммы бронирования
WHERE book_ref = '999999';

 book_ref |           book_date           | total_amount
----------+-------------------------------+--------------
 999999   | 2023-11-21 16:32:02.423827+04 |      2000.00
(1 row)

demo=# -- По умолчанию PostgreSQL использует уровень изоляции Read Committed. 
demo=# -- Этот уровень изоляции гарантируется отсутствие потерянных обновлений.
demo=# -- После фиксации второй транзакции со вставкой бронирования на 1000 рублей
demo=# -- эта вставка не была потеряна и была учтена в незакрытой первой транзакции.
```

</details>
<details>
<summary>Упражнение 6.5. Уровень изоляции Read Committed. #EGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ #COMMIT</summary>
Повторите предыдущее упражнение, но начните транзакцию в первом сеансе с уровнем изоляции транзакций Repeatable Read. Объясните различие полученных результатов.

  **Решение 6.5:**

```SQL
demo=#  DELETE FROM bookings WHERE book_ref = '999999';
DELETE 1

demo=# BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ; -- Начало первой транзакции
BEGIN

demo=*# UPDATE bookings
SET total_amount = total_amount * 2
WHERE total_amount = 1000;
UPDATE 0
demo=*#

demo=# BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ; -- Начало второй транзакции
BEGIN

demo=*# INSERT INTO bookings (book_ref, book_date, total_amount)
VALUES ('999999', CURRENT_TIMESTAMP, 1000);
INSERT 0 1

demo=*# COMMIT; -- Фиксация второй транзакции
COMMIT

demo=*# SELECT * FROM bookings -- Проверка суммы бронирования в первой транзакции
WHERE book_ref = '999999';

 book_ref | book_date | total_amount
----------+-----------+--------------
(0 rows)

demo=*# UPDATE bookings -- Обновление таблицы в первой транзакции
SET total_amount = total_amount * 2
WHERE total_amount = 1000;
UPDATE 0

demo=*# COMMIT; -- Фиксация первой транзакции
COMMIT

demo=# SELECT * FROM bookings -- Проверка суммы бронирования
WHERE book_ref = '999999';

 book_ref |           book_date           | total_amount
----------+-------------------------------+--------------
 999999   | 2023-11-21 18:16:02.969452+04 |      1000.00
(1 row)

demo=# -- Транзакции уровня Repeatable Read создают снимок данных однократно в начале транзакции перед запросами.
demo=# -- В данном случае первая транзакция не смогла обновить строки после их изменения в результате второй транзакции.
demo=# -- Но после завершения транзакций и разблокировки строк первая транзакция увидела изменения во второй.
```

</details>
<details>
<summary>Упражнение 6.6. Уровень изоляции Read Committed. #BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ #INSERT INTO #COMMIT</summary>
Выполните указанные действия в двух сеансах: 
1. В первом сеансе начните новую транзакцию с уровнем изоляции Repeatable Read. Вычислите количество бронирований с суммой 20 000 рублей. 
2. Во втором сеансе начните новую транзакцию с уровнем изоляции Repeatable Read. Вычислите количество бронирований с суммой 30 000 рублей. 
3. В первом сеансе добавьте новое бронирование на 30 000 рублей и снова вычислите количество бронирований с суммой 20 000 рублей. 
4. Во втором сеансе добавьте новое бронирование на 20 000 рублей и снова вычислите количество бронирований с суммой 30 000 рублей. 
5. Зафиксируйте транзакции в обоих сеансах. 
Соответствует ли результат ожиданиями? Можно ли сериализовать эти транзакции (иными словами, можно ли представить такой порядок последовательного выполнения этих транзакций, при котором результат совпадет с тем, что получился при параллельном выполнении)?

  **Решение 6.6:**

```SQL
demo=#  BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ; -- Начало первой транзакции
BEGIN
demo=*# SELECT count(*) FROM bookings
WHERE total_amount = 20000;
 count
-------
  2235
(1 row)

demo=*#
demo=# BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ; -- Начало второй транзакции
BEGIN
demo=*# SELECT count(*) FROM bookings
WHERE total_amount = 30000;
 count
-------
  2894
(1 row)

demo=*# INSERT INTO bookings (book_ref, book_date, total_amount) -- В первой транзакции
VALUES ('999998', CURRENT_TIMESTAMP, 30000);
INSERT 0 1
demo=*# SELECT count(*) FROM bookings
WHERE total_amount = 20000;
 count
-------
  2235
(1 row)

demo=*# SELECT total_amount, count(total_amount) FROM bookings
WHERE total_amount = 20000 OR total_amount = 30000
GROUP BY total_amount;
 total_amount | count
--------------+-------
     20000.00 |  2235
     30000.00 |  2895
(2 rows)

demo=*# INSERT INTO bookings (book_ref, book_date, total_amount) -- Во второй транзакции
VALUES ('999999', CURRENT_TIMESTAMP, 20000);
INSERT 0 1
demo=*# SELECT count(*) FROM bookings
WHERE total_amount = 30000;
 count
-------
  2894
(1 row)

demo=*# SELECT total_amount, count(total_amount) FROM bookings
WHERE total_amount = 20000 OR total_amount = 30000
GROUP BY total_amount;
 total_amount | count
--------------+-------
     20000.00 |  2236
     30000.00 |  2894

demo=*# COMMIT; -- Первой транзакции
COMMIT

demo=# SELECT total_amount, count(total_amount) FROM bookings
WHERE total_amount = 20000 OR total_amount = 30000
GROUP BY total_amount;
 total_amount | count
--------------+-------
     20000.00 |  2235
     30000.00 |  2895
(2 rows)

demo=*#demo=*# COMMIT; -- Второй транзакции
COMMIT

demo=# SELECT total_amount, count(total_amount) FROM bookings
WHERE total_amount = 20000 OR total_amount = 30000
GROUP BY total_amount;
 total_amount | count
--------------+-------
     20000.00 |  2236
     30000.00 |  2895
(2 rows)

demo=# -- Т.к. снимок данных делается на начало транзакций, то не смотря на вставку demo=# -- данных и сообщение об успешной вставке,
demo=# -- незакрытые транзакции видят данные внутри себя без изменений,
demo=# -- При любом порядке последовательного выполнения транзакций 
demo=# -- одна из них будет читать измененные несогласованные строки другой транзакции.
demo=# -- Сериализовать транзакции не удастся.
```

</details>
<details>
<summary>Упражнение 6.7. Уровень изоляции Serializable. #BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE #INSERT INTO #COMMIT</summary>
Повторите предыдущее упражнение, но транзакции в обоих сеансах начните с уровнем изоляции Serializable. Если вы правильно ответили на его последний вопрос, вы поймете, почему теперь эти действия приводят к ошибке. Если же результат этого упражнения стал для вас неожиданностью, четко сформулируйте различие уровней Repeatable Read и Serializable.

  **Решение 6.7:**

```SQL
demo=# BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;-- Начало первой транзакции
BEGIN
demo=*# SELECT count(*) FROM bookings
WHERE total_amount = 20000;
 count
-------
  2235
(1 row)

demo=*#
demo=# BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;-- Начало второй транзакции
BEGIN
demo=*# SELECT count(*) FROM bookings
WHERE total_amount = 30000;
 count
-------
  2894
(1 row)

demo=*# INSERT INTO bookings (book_ref, book_date, total_amount) -- В первой транзакции
VALUES ('999998', CURRENT_TIMESTAMP, 30000);
INSERT 0 1
demo=*# SELECT count(*) FROM bookings
WHERE total_amount = 20000;
 count
-------
  2235
(1 row)

demo=*# INSERT INTO bookings (book_ref, book_date, total_amount) -- Во второй транзакции
VALUES ('999999', CURRENT_TIMESTAMP, 20000);
INSERT 0 1
demo=*# SELECT count(*) FROM bookings
WHERE total_amount = 30000;
 count
-------
  2894
(1 row)
demo=*# COMMIT; -- Первой транзакции
COMMIT
COMMIT; -- Второй транзакции
ERROR:  could not serialize access due to read/write dependencies among transactions
DETAIL:  Reason code: Canceled on identification as a pivot, during commit attempt.
HINT:  The transaction might succeed if retried.

demo=# SELECT total_amount, count(total_amount) FROM bookings
WHERE total_amount = 20000 OR total_amount = 30000
GROUP BY total_amount;
 total_amount | count
--------------+-------
     20000.00 |  2235
     30000.00 |  2895
(2 rows)

demo=# -- Сериализация не прошла. Вторая транзакция пытается прочитать строки, измененные в первой.
demo=# -- Зафиксированы изменения только первой транзакции, поскольку она зафиксировала их первой.
```

</details>

## Глава 8. Расширения реляционной модели
<details>
<summary>Упражнение 8.1 Преобразование в json и возврат json. #to_json() #json_build_object()</summary>
Напишите запрос, выдающий список самолетов из демонстрационной базы в формате JSON.

  **Решение 8.1:**

```SQL
-- В одной колонке преобразуем простое скалярное значение в json, в другой – возвратим json-объект из списка аргументов.
SELECT to_json (model) AS json_simple, json_build_object ('model' , model , 'range' , range)
	FROM aircrafts;

      json_simple      |                 json_build_object
-----------------------+---------------------------------------------------
 "Boeing 777-300"      | {"model" : "Boeing 777-300", "range" : 11100}
 "Boeing 767-300"      | {"model" : "Boeing 767-300", "range" : 7900}
 "Sukhoi SuperJet-100" | {"model" : "Sukhoi SuperJet-100", "range" : 3000}
 "Airbus A320-200"     | {"model" : "Airbus A320-200", "range" : 5700}
 "Airbus A321-200"     | {"model" : "Airbus A321-200", "range" : 5600}
 "Airbus A319-100"     | {"model" : "Airbus A319-100", "range" : 6700}
 "Boeing 737-300"      | {"model" : "Boeing 737-300", "range" : 4200}
 "Cessna 208 Caravan"  | {"model" : "Cessna 208 Caravan", "range" : 1200}
 "Bombardier CRJ-200"  | {"model" : "Bombardier CRJ-200", "range" : 2700}
(9 rows)
```

</details>
<details>
<summary>Упражнение 8.3 Возврат в формате json. #json_agg() #json_build_object()</summary>
Напишите запрос, выдающий заданное бронирование в формате JSON, включая все входящие в него билеты и перелеты для каждого из билетов.

  **Решение 8.3:**

```SQL
-- С помощью агрегатной функции собираем в один json-документ все билеты и перелеты одного конкретного бронирования

WITH booking AS (
	SELECT ticket_no, book_ref 
	FROM tickets
	WHERE book_ref = 'FCC5B7')
SELECT json_agg(
json_build_object(
'ticket_no' , tf.ticket_no ,
'book_ref' , b.book_ref , 
'flight_id' , tf.flight_id, 
'flight_no' , f.flight_no , 
'dep_airport' , f.departure_airport , 
'arrival_airport' , f.arrival_airport))
	FROM ticket_flights AS tf
	JOIN booking AS b ON tf.ticket_no = b.ticket_no
	JOIN flights AS f ON tf.flight_id = f.flight_id;

json_agg                                                                                                                       |
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
[{"ticket_no" : "0005432000301", "book_ref" : "FCC5B7", "flight_id" : 187801, "flight_no" : "PG0242", "dep_airport" : "CSY", "arrival_airport" : "SVO"}, {"ticket_no" : "0005432000302", "book_ref" : "FCC5B7", "flight_id" : 187801, "flight_no" : "PG0242", "|

1 row(s) fetched.
```

</details>

## Глава 16. Функции и процедуры в базе данных
<details>
<summary>Упражнение 16.1. Напишите на языке PL/pgSQL функцию, возвращающую все данные, относящиеся к одному бронированию, номер которого задан параметром.</summary>  

  **Решение 16.1:**  
По параметру номера бронирования функция возвращает этот номер, дату бронирования, сумму бронирования и из другой связанной таблицы номер билета(ов) в бронировании, id пассажира и его имя. Другая информация из этой таблицы по имейлу и номеру телефона в формате jsonb закомментарена. Также в функции реализована обработка исключений при запросе несуществующего номера бронирования.  

```SQL
CREATE OR REPLACE FUNCTION get_b_info_max(b_ref text) 
RETURNS TABLE (
book_ref CHARACTER(6), 
book_date DATE, 
total_amount NUMERIC(10,2), 
ticket_no CHARACTER(13), 
passenger_id CHARACTER VARYING(20), 
passenger_name TEXT
) AS 
$$
BEGIN
    RETURN QUERY 
			SELECT
				b.book_ref, 
				b.book_date::date, 
				b.total_amount, 
				t.ticket_no, 
				t.passenger_id, 
				t.passenger_name 
				--t.contact_data::json ->> 'email' AS email,  
				--t.contact_data::json ->> 'phone' AS phone
				FROM bookings AS b
				JOIN tickets AS t ON b.book_ref = t.book_ref
				WHERE b.book_ref = $1;
	IF NOT FOUND THEN
        RAISE EXCEPTION 'Не найдено бронирование с номером: %', $1;
    END IF;
	RETURN;
END;
$$ LANGUAGE plpgsql;
```

Подадим на вход функции номер бронирования 000010. В этом бронировании два билета.
```SQL
demo=# SELECT * FROM get_b_info_max('000010');

 book_ref | book_date  | total_amount |   ticket_no   | passenger_id |   passenger_name   
----------+------------+--------------+---------------+--------------+--------------------
 000010   | 2017-01-08 |     50900.00 | 0005432295359 | 5722 837257  | ALEKSANDR SOKOLOV
 000010   | 2017-01-08 |     50900.00 | 0005432295360 | 0564 044306  | LYUDMILA BOGDANOVA

```

Подадим на вход функции несуществующий номер бронирования:
```SQL
demo=# SELECT * FROM get_b_info_max('000001');

ОШИБКА:  Не найдено бронирование с номером: 000001
КОНТЕКСТ:  функция PL/pgSQL get_b_info_max(text), строка 17, оператор RAIS
```

</details>