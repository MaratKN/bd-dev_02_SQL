# Домашнее задание к занятию 2. «SQL»

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

```
version: '3'

services:

  postgres:

    image: postgres:12

    container_name: postgres12

    restart: always

    ports:

      - 5432:5432

    environment:

      POSTGRES_DB: mydb

      POSTGRES_USER: admin

      POSTGRES_PASSWORD: NeSmotri

    volumes:
      - pg_data:/home/marat/database/
      - pg_backups:/home/marat/backup/

volumes:

  pg_data:

  pg_backups:

```


Затем:

$ docker-compose up -d

$ sudo docker exec -it postgres12 bash

## Задача 2

В БД из задачи 1: 

- создайте пользователя test-admin-user и БД test_db;
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
- создайте пользователя test-simple-user;
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

- id (serial primary key);
- наименование (string);
- цена (integer).

Таблица clients:

- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).

Приведите:

- итоговый список БД после выполнения пунктов выше;
- описание таблиц (describe);
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
- список пользователей с правами над таблицами test_db.

```
root@546abf9ea571:/# createdb test_db -U admin

root@546abf9ea571:/# psql -d test_db -U admin

psql (12.16 (Debian 12.16-1.pgdg120+1))

Type "help" for help.



test_db=# CREATE USER test_admin_user;

CREATE ROLE

test_db=# CREATE TABLE orders

(

   id SERIAL PRIMARY KEY,

   наименование TEXT,

   цена INTEGER

);

CREATE TABLE

test_db=# CREATE TABLE clients

(

    id SERIAL PRIMARY KEY,

    фамилия TEXT,

    "страна проживания" TEXT,

    заказ INTEGER,

    FOREIGN KEY (заказ) REFERENCES orders(id)

);

CREATE TABLE

test_db=# CREATE INDEX country_index ON clients ("страна проживания");

CREATE INDEX

test_db=# GRANT ALL ON TABLE orders TO test_admin_user;

GRANT

test_db=# GRANT ALL ON TABLE clients TO test_admin_user;

GRANT

test_db=# CREATE USER test_simple_user;

CREATE ROLE

test_db=# GRANT SELECT,INSERT,UPDATE,DELETE ON TABLE orders TO test_simple_user;

GRANT

test_db=# GRANT SELECT,INSERT,UPDATE,DELETE ON TABLE clients TO test_simple_user;

GRANT

test_db=# 
```

![alt text](https://github.com/MaratKN/bd-dev_02_SQL/blob/main/1.png)

Итоговый список БД:

![alt text](https://github.com/MaratKN/bd-dev_02_SQL/blob/main/2.png)

Описание таблиц:

![alt text](https://github.com/MaratKN/bd-dev_02_SQL/blob/main/3.png)

Список пользователей с правами над таблицами test_db:

test_db=# SELECT grantee, table_catalog, table_name, privilege_type FROM information_schema.table_privileges WHERE table_name IN ('orders','clients');

![alt text](https://github.com/MaratKN/bd-dev_02_SQL/blob/main/4.png)



## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

Приведите в ответе:

    - запросы,
    - результаты их выполнения.
	
	
	
test_db=# INSERT INTO orders VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);

test_db=# INSERT INTO clients VALUES (1, 'Иванов Иван Иванович', 'USA'), (2, 'Петров Петр Петрович', 'Canada'), (3, 'Иоганн Себастьян Бах', 'Japan'), (4, 'Ронни Джеймс Дио', 'Russia'), (5, 'Ritchie Blackmore', 'Russia');

SELECT COUNT (*) FROM orders;

SELECT COUNT (*) FROM clients;

![alt text](https://github.com/MaratKN/bd-dev_02_SQL/blob/main/5.png)



## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.
 
Подсказка: используйте директиву `UPDATE`.

test_db=# UPDATE clients SET заказ=(select id from orders where наименование='Книга') WHERE фамилия='Иванов Иван Иванович';

test_db=# UPDATE clients SET заказ=(select id from orders where наименование='Монитор') WHERE фамилия='Петров Петр Петрович';

test_db=# UPDATE clients SET заказ=(select id from orders where наименование='Гитара') WHERE фамилия='Иоганн Себастьян Бах';

![alt text](https://github.com/MaratKN/bd-dev_02_SQL/blob/main/6.png)

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

EXPLAIN: 

![alt text](https://github.com/MaratKN/bd-dev_02_SQL/blob/main/7.png)

Метод Seq Scan - последовательное чтение данных. 
0.00 - ожидаемые затраты на получение первой строки.
18.10 — ожидаемые затраты на получение всех строк.
Rows - ожидаемое число строк, которое должен вывести этот узел плана (узел выполняется до конца).
Width - ожидаемый средний размер строк, выводимых этим узлом плана (в байтах).
Каждая запись сравнивается с условием "заказ" IS NOT NULL. Если условие выполняется, то запись вводится в результат.


## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 


root@afde29351d74:/# su - postgres

postgres@afde29351d74:~$ pg_dump test_db > /backup/dump999

postgres@afde29351d74:~$ pg_dumpall > /backup/dumpall999

marat@debian2:~/pgdump$ sudo docker stop afde29351d74

marat@debian2:~/pgdump$ sudo nano docker-compose.yaml 

marat@debian2:~/pgdump$ sudo docker compose up --detach

marat@debian2:~/pgdump$ sudo docker exec -it marat-postgresql2-1 bash

root@d39a79e66147:/# su - postgres

postgres@d39a79e66147:~$ psql -f /backup/dumpall999

postgres@d39a79e66147:~$ pg_dump < /backup/dump999

![alt text](https://github.com/MaratKN/bd-dev_02_SQL/blob/main/8.png)

![alt text](https://github.com/MaratKN/bd-dev_02_SQL/blob/main/9.png)


