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

      - pg_data:/var/lib/postgresql/data

      - pg_backups:/backups

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

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

