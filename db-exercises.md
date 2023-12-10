# Практические задания из учебника [Основы технологий баз данных: учебное пособие / Б. А. Новиков,Е. А. Горшкова, Н. Г. Графеева; под ред. Е. В. Рогова. — 2-е изд. — М.: ДМК Пресс, 2020. — 582 с.](https://postgrespro.ru/education/books/dbtech)
Задания выполнены на основе [демонстрационной базы данных "Авиаперевозки".](https://postgrespro.ru/education/demodb)

## Глава 4. Введение в SQL
<details>
<summary>Упражнение 4.3 (Основные конструкции и синтаксис)</summary>
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
<summary>Упражнение 4.4 (Основные конструкции и синтаксис)</summary>
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
<summary>Упражнение 4.7 (JOIN)</summary>
Напечатанный посадочный талон должен содержать фамилию и имя пассажира, 
коды аэропортов вылета и прилета, дату и время вылета и прилета по расписанию, 
номер места в салоне самолета. 
Напишите запрос, выводящий всю необходимую информацию для полученных
посадочных талонов на рейсы, которые еще не вылетели.

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
<summary>Упражнение 4.9 (Выводд отсутствующих значений)</summary>
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
<summary>Упражнение 4.11 (Агрегирование и группировка)</summary>
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
<summary>Упражнение 4.15 (Модификация данных)</summary>
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
<summary>Упражнение 4.20 (Описание данных: отношения)</summary>
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
