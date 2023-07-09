# Разворачиваем docker-compose с сервером и клиентом PostgreSQL 15

### 1. Запускаем docker-compose файл
```shell
docker-compose up -d 
```
Сервис client не сможет стартануть, потому что не описаны переменные окружения с кредами POSTGRES. 
Зато к контейнеру client можно обратиться с помощью:
```shell
docker-compose run --rm client /bin/bash
```
Далее попадем в psql
```shell
psql -h server -U docker_postgres
```

### 2. Создадим базу данных, схему и таблицу, а также вставим несколько записей в таблцу
```sql
CREATE DATABASE otus_db;
\c otus_db;
CREATE SCHEMA otus;
CREATE TABLE otus.first_table (first_row numeric);
INSERT INTO otus.first_table (first_row) values (1);
INSERT INTO otus.first_table (first_row) values (2);
INSERT INTO otus.first_table (first_row) values (3);
\q
```
Выходим из оболочки контейнера client. 
### 3. Подключаемся занового к client и проверяем что данные по прежнему на месте и доступны:
```sql
select * from otus.first_table;
```

# Пробуем подключиться к контейнеру server с ноутбука/компьютера (IP 192.168.1.X) извне хоста с запущенным docker-compose
### 1. Попробуем подключиться с помощью dBeaver. В качестве хоста сервера укажем хост машины с запущенным docker-compose (IP 192.168.1.120).
Подключение извне хоста с запущенным docker-compose удалось потому что мы явно пробросили порт 5432 котейнера в порт 5433 хоста с запущенным docker-compose. Без указания проброса портов подключение к server не возможно извне docker сети pg_net.
### 2. Пробуем вычитать ранее созданные данные.
```sql
select * from otus.first_table;
```

# Удаляем docker-compose
```shell
docker-compose down
```
Поднимаем занового
```shell
docker-compose up -d 
```
С помощью контейнера client подключаемся к контейнеру server аналогично п.X. Подключение удалось, попробуем вычитать ранее созданные данные.
```sql
select * from otus.first_table;
```
Видим, что ранее созданные данные не удалены и по прежнему доступны. Это возможно, потому что пользовательские данные лежат в /var/lib/postgresql/data который мы пробросили на хостовую машину в каталог ./pg_data.
