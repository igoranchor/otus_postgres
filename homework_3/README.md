# Создание и настройка БД
1. Подключаемся к виртуальной машина в YC
```shell
ssh user@ip_address
```

2. Проверяем, что кластер запущен, заходим в psql
```shell
sudo su postgres
pg_lsclusters
psql
```


1. Создаем произовльную таблицу с произвольным содержимым и выходим из psql
```sql
create table test(c1 text);
insert into test values('1');
```

2. Останавливаем кластер
```shell
pg_ctlcluster 15 main stop
exit #to user
```

3. Создаем новый SSD диск в YC с помощью веб-интерфейса и добавляем к виртуальной машине

4. проинициализируем диск и монтируем к файловую систему (не описано shell командами, использовал утилиту lsblk) 
и проверяем, что все успешно подмонтировалось
```shell
exit #to user
sudo fdisk -l
sudo lsblk -o NAME,FSTYPE,LABEL,UUID,MOUNTPOINT
```
Disk /dev/vdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: XXXXXX
First LBA: 34
Last LBA: 20971486
Alternative LBA: 20971519
Partition entries LBA: 2
Allocated partition entries: 128

vda                                                                
├─vda1                                                             
└─vda2 ext4                   82aeea96-6d42-49e6-85d5-9071d3c9b6aa /
vdb                                                                
└─vdb1 ext4     datapartition 2b62740d-71b1-41a5-8e7d-4965f5ca913b /mnt/data

5. Делаем пользователя postgres владельцем /mnt/data
```shell
sudo chown -R postgres:postgres /mnt/data/
```

6. Переносим содержимое /var/lib/postgres/15 в /mnt/data/15
```shell
sudo mv /var/lib/postgresql/15/ /mnt/data/15/
```

7. Стартуем кластер
```shell
sudo -u postgres pg_ctlcluster 15 main start
```
Получаем ошибку /var/lib/postgresql/15/main is not accessible or does not exist. 
Пока все логично, перенесли директорию из /var/lib/postgresql/15/main в другое место, а pgsql об этом ничего не знает.

8. Меняем содержимое файла /etc/postgresql/15/main/postgres.conf, устанавливаем параметр
data_directory = /mnt/data/15/main
```shell
sudo nano /etc/postgresql/15/main/postgres.conf
```

9. Стартуем кластер еще раз
```shell
sudo -u postgres pg_ctlcluster 15 main start
```
После того, как мы указали в postgres.conf где искать файлы, кластер успешно стартанул
