# Домашнее задание к занятию «Репликация и масштабирование. Часть 1» Павлов Дмитрий  

## Задание 1  
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия. 
## Решение  
В случае master-slave репликация происходит в одну строну, в случае master-master репликация происходит друг на друга.

## Задание 2  
Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.  
Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.  
## Решение  
Выполнил задание через docker compose  
Создал docker-compose.yaml в нем два контейнера мастер и слейв, так же поднята сеть replica для них  
```
services:
  master:
    image: mysql:8.4
    container_name: master
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
    networks:
      service_replica:
    ports:
      - "3306:3306"
    volumes:
      - ./master.cnf:/etc/my.cnf
      - ./01_master_create_user_replica.sql:/docker-entrypoint-initdb.d/01_master_create_user_replica.sql
      - ./02_master_create_db.sql:/docker-entrypoint-initdb.d/02_master_create_db.sql
  replica:
    image: mysql:8.4
    container_name: replica
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
    networks:
      service_replica:      
    ports:
      - "3307:3306"
    volumes:
      - ./replica.cnf:/etc/my.cnf
      - ./01_replica_create_replica.sql:/docker-entrypoint-initdb.d/01_replica_create_replica.sql

networks:
  service_replica:
    name: service_replica
```

При запуске копируются конфигурационные файлы в контейнеры master.cnf и replica.cnf  
В мастере с помощью sql скрипта 01_master_create_user_replica.sql создается пользователь с полными правами  
```
CREATE USER 'replica'@'%';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
FLUSH PRIVILEGES;
```
В мастере с помощью sql скрипта 02_master_create_db,sql создается база данныз с одной таблицей и с одной строкой данных  
```
CREATE DATABASE test_db;
USE test_db;
CREATE TABLE test_table (id INT PRIMARY KEY, name VARCHAR(50));
INSERT INTO test_table VALUES (1, 'Master Record');
```
В слейве с помощью sql скрипта ./01_replica_create_replica.sql настраивается и запускается реплика  
```
CHANGE REPLICATION SOURCE TO
SOURCE_HOST='master',
SOURCE_USER='replica';
START REPLICA;
```
На скриншоте видим что реплика настроена  
Слева мастер контейнер с созданой базой, справа контейнер с настроенной репликой  
![скриншот к заданию 2](/pic/pic01.png)
На скриншоте видим что БД в двух контейнерах среплицировалась    
![скриншот к заданию 2](/pic/pic02.png)
