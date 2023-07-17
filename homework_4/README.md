# Логический уровень PostgreSQL

1. Поднимаем новый класер в docker
```shell
docker run --name postgres --rm -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres:15.3
```

2. Создаем новую БД testdb
```sql
create database testdb;
```
3. Создаем новую схему testnm
```sql
create schema testnm;
```
4. Создаем новую таблицу t1 с одной колонкой c1 типа integer
```sql
CREATE TABLE t1(c1 integer);
```
5. Вставляем строку со значением c1=1
```sql
insert into t1 values(1);
```
6. Создаем новую роль readonly
```sql
create role readonly;
```
7. Даем новой роли право на подключение к базе данных testdb
```sql
grant connect on database testdb to readonly;
```
8. Даем новой роли право на использование схемы testnm
```sql
grant usage on schema testnm to readonly;
```
9. Даем новой роли право на чтение для всех таблиц схемы testnm
```sql
grant select on all tables in schema testnm to readonly;
```
10. Создаем пользователя testread с паролем test123
```sql
create user testread with password 'test123';
```
11. Даем роль readonly пользователю testread
```sql
grant readonly TO testread;
```
12. Заходим под пользователем testread в базу данных testdb и делаем select * from t1
```sql
select * from t1;
```
Получаем permission denied for table t1. При создании таблицы не была указана схема, 
соответственно таблица создалась в схеме search_path, по умолчанию это public. 
Схема public по умолчанию входит в search_path любого пользователя. 
Соответственно таблица есть, пользователь testread может ее увидеть, но прав к ней нет, 
т.к. не были предоставлены права к схеме public.

13. Удаляем таблицу t1
```sql
drop table t1;
```
14. Создаем ее заново но уже с явным указанием имени схемы testnm
```sql
create table testnm.t1(c1 integer);
```
15. Вставляем строку со значением c1=1
```sql
insert into testnm.t1 values(1);
```
16. Заходим под пользователем testread в базу данных testdb и делаем select * from t1
```sql
select * from testnm.t1;
```
Получаем permission denied for table t1. На момент выдачи прав на все таблицы в схеме таблицы t1 не существовало, 
соответственно на нее права не были выделены для роли readonly.

17. Повторно Даем новой роли право на чтение для всех таблиц схемы testnm и 
изменим привилегии по умолчанию для схемы testnm роли readonly
```sql
grant select on all tables in schema testnm to readonly;
alter default privileges in schema testnm grant select on tables to readonly;
```
Суть в том, что конструкция `grant something in schema schema to role` - 
предоставляет права на уже созданные обхекты в схеме, 
а `alter default privileges in schema schema grant something to readonly` - предоставляет права к тем объектам, 
которые будут созданы.

18. Под пользователем с ролью readonly попробуем создать таблицу t2 и записать в нее что-нибудь
```sql
create table t2(c1 integer); 
insert into t2 values (2);
```
Получаем ошибку permission denied for schema public. Начиная с 15 версии pgSQL права на создание таблиц в 
схеме public были отозваны.

19. Под пользователем postgres предоставим права на создание объектов в public схеме пользователям с ролью readonly и повторим шаг 18
```sql
grant create on schema public to readonly;
```
Таблица успешно создалась, как и запись в ней. У пользователя testread есть все права на таблицу t2, 
т.к. он является ее владельцем.

20. Отберем у роли readonly права на создание объектов в схеме public и использование схемы
```sql
revoke all on schema public from readonly;
```
Однако все права на таблицу t2 остались у пользователя testread. 
Думаю, что пока пользователь testread является владельцем t2, возможность ограничить testread от t2 отсутствует.

Считаю отличной доработкой 15 pgSQL - отсутствие возможности по умолчанию создавать объекты в схеме public. 
Т.о. pgSQL из коробки заботиться о нашем проде :)