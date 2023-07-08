# Создание и настройка БД
1. Подключаемся к виртуальной машина в YC
```shell
ssh user@ip_address
```
2. Запускаем psql
```shell
sudo su postgres
psql
```
3. Создаем вторую ssh сессию к виртуальной машине

4. В обоих сессиях отключаем auto commit
```sql
\set AUTOCOMMIT off
```

5. Создаем в первой сессии новую таблицу в новой схеме и заполняем ее данными
```sql
create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
insert into persons(first_name, second_name) values('petr', 'petrov'); 
commit;
```


# Проверка уровня изоляции транзакций по умолчанию и работа с ним
1. Проверяем текущий уровень изоляции транзакций
```sql
show transaction isolation level;
```
Текущий уровень изоляции транзакций - `Read commited`

2. В обоих сессиях работы с postgres открываем новую транзакцию
```sql
begin transaction;
```

3. В первой сессии добавляем запись
```sql
insert into persons(first_name, second_name) values('sergey', 'sergeev');
```

4. Во второй сессии запрашиваем информацию из persons
```sql
select * from persons;
```
Добавленной в первой сессии записи не обнаружено.
Так произошло, потому что мы не сделали коммит в первой сессии, т.о. мы создали аномалию `dirty-read`. 
Аномалию `dirty-read` postgres не поддерживает ни на одном уровне изоляций транзакций

5. В первой сессии фиксируем транзакцию
```sql
commit;
```

6. Во второй сессии запрашиваем информацию из persons
```sql
select * from persons;
```
Добавленная в первой сессии запись обнаружена. 
Так произошло, потому что мы создали аномалию `non-repeatable read`, а при уровне изоляции транзакций `Read commited`
аномалия `non-repeatable read` возможна

7. Завершаем транзакцию во второй сессии
```sql
commit/rollback;
```


# Уровень изоляции транзакций `Repeatable read` и работа с ним
1. В обоих сессиях работы с postgres открываем новую транзакцию с уровнем изоляции `Repeatable read`
```sql
begin transaction;
set transaction isolation level repeatable read;
```

2. В первой сессии добавляем запись
```sql
insert into persons(first_name, second_name) values('sveta', 'svetova');
```

3. Во второй сессии запрашиваем информацию из persons
```sql
select * from persons;
```
Добавленной в первой сессии записи не обнаружено.
Так произошло, потому что мы не сделали коммит в первой сессии, т.о. мы создали аномалию `dirty-read`. 
Аномалию `dirty-read` postgres не поддерживает ни на одном уровне изоляции транзакций.

4. В первой сессии фиксируем транзакцию
```sql
commit;
```

5. Во второй сессии запрашиваем информацию из persons
```sql
select * from persons;
```
Добавленной в первой сессии записи не обнаружено.
Так произошло, потому что мы создали аномалию `non-repeatable read`, а при уровне изоляции транзакций `Repeatable read`
аномалия `non-repeatable read` невозможна

6. Завершаем транзакцию во второй сессии
```sql
commit/rollback;
```

7. Во второй сессии запрашиваем информацию из persons
```sql
select * from persons;
```
Наконец, наблюдаем запись, добавленную первой сессией. В текущей ситуации мы не создаем никаких аномалий, 
соотвественно видим те данные, которые в текущий момент времени находятся в талице persons.