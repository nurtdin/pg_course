# Домашняя работа. Урок №7

## Новый кластер PostgresSQL 14

1. создайте новый кластер PostgresSQL 14

Новый кластер Postgres 14 создан с помощью команды:

```console
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-14
```
Результат установки:
![Результат установки Postgres 14](/images/img23.jpg "Результат установки Postgres 14")

2. зайдите в созданный кластер под пользователем postgres
```console
sudo su postgres
cd /var/lib/postgresql
psql
```

3. Подключение с помощью psql:

![Подключение с помощью psql](/images/img24.jpg "Подключение с помощью psql")

# Новая база данных testdb. Демонстрация назначения привилегий в Postgres и их особенности
1. Создайте новую БД testdb

```sql
create database testdb;
```

![Создание новой БД](/images/img25.jpg "Создание новой БД")

2. зайдите в созданную базу данных под пользователем postgres

```sql
\c testdb
```

![Подключение к БД](/images/img26.jpg "Подключение к  БД")

3. создайте новую схему testnm

```sql
create schema testnm;
```

![Создвание схемы](/images/img27.jpg "Создание схемы")


4. создайте новую таблицу t1 с одной колонкой c1 типа integer

```sql
create table testnm.t1(c1 integer);
```

![Создвание таблицы t1](/images/img28.jpg "Создание таблицы t1")

5. вставьте строку со значением c1=1

```sql
testdb=# insert into testnm.t1(c1) values(1);
```

![Вставка строки в таблицу t1](/images/img29.jpg "Вставка строки в таблицу t1")

6. создайте новую роль readonly

```sql
create role readonly;
```

![Создание роли](/images/img30.jpg "Создание роли")

7. дайте новой роли право на подключение к базе данных testdb

```sql
grant connect on database testdb to readonly;
```

![Привилегия на подключение к БД](/images/img31.jpg "Привилегия на подключение к БД")

9. дайте новой роли право на использование схемы testnm

```sql
grant usage on schema testnm to readonly;
```

![Привилегия на использование схемы](/images/img32.jpg "Привилегия на использование схемы")

10. дайте новой роли право на select для всех таблиц схемы testnm

```sql
grant select on all tables in schema testnm to readonly;
```

![Привилегия на выборку из всех таблиц схемы](/images/img33.jpg "Привилегия на выборку из всех таблиц схемы")

11. создайте пользователя testread с паролем test123

```sql
create user testread with password 'test123';
```

![Создание пользователя](/images/img34.jpg "Создание пользователя")

12. дайте роль readonly пользователю testread

```sql
grant readonly to testread;
```

![Выдача роли](/images/img35.jpg "Выдача роли")

13. зайдите под пользователем testread в базу данных testdb

```cpnsole
psql -h localhost -U testread -d testdb
```

![Подключение к БД под тестовым пользователем](/images/img36.jpg "Подключение к БД под тестовым пользователем")

14. сделайте select * from t1;

![Результат запроса](/images/img37.jpg "Результат запроса")

15. получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)

Не получилось.

16. напишите что именно произошло в тексте домашнего задания

Таблица создана в схеме testnm, а при выполнении

```sql
select * from t1;
```

таблица "искалась" в схеме public.

17. у вас есть идеи почему? ведь права то дали?

Права выданы на таблицу в схеме testnm, к которой мы не обращаемся. Обращение происходит к таблице public.t1, т.к. имя схемы явно не указано. В схеме public таблицы t1 нет, поэтому получаем ошибку:

ERROR:  relation "t1" does not exist

Если бы таблицу создали без указания схемы, то она была бы создана в схеме public, т.к. роль public по умолчанию выдается каждому пользователю.

18. посмотрите на список таблиц

```sql
\dt
```

![Список таблиц](/images/img38.jpg "Список таблиц")

Список таблиц в схеме public пустой.

19. подсказка в шпаргалке под пунктом 20

а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
Нужно или указывать имя схемы перед объектом 
```sql
select * from testnm.t1;
```
или менять текущую схему на testnm
```sql
SET search_path TO testnm, public;
select * from t1;
```

![Результат с указанием схемы](/images/img39.jpg "Результат с указанием схемы")

20. вернитесь в базу данных testdb под пользователем postgres

```console
sudo su postgres
psql
```

21. удалите таблицу t1

```sql
drop table testnm.t1;
```

22. создайте ее заново но уже с явным указанием имени схемы testnm

```sql
create table testnm.t1(c1 integer);
```

23. вставьте строку со значением c1=1

```sql
insert into testnm.t1(c1) values(1);
```

24. зайдите под пользователем testread в базу данных testdb

```console
psql -h localhost -U testread -d testdb
```

25. сделайте select * from testnm.t1;

```sql
select * from testnm.t1;
```

26. получилось?

Не получилось, возникла ошибка

![Ошибка при выполнении запроса](/images/img40.jpg "Ошибка при выполнении запроса")

27. есть идеи почему? если нет - смотрите шпаргалку

Т.к. не определены привилегии по умолчанию. См. https://www.postgresql.org/docs/current/sql-alterdefaultprivileges.html
Привилегии на выборку из всех таблиц в схеме testnm были выданы после создания таблицы t1. В дальнейшем таблица была пересоздана, привилегий на нее выдано не было. Поведение Postgres by design.

28. как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку

Необходимо выдать привилегии по умолчанию:

```sql
alter default privileges in schema testnm grant select on tables to readonly;
```

![Привилегии по умолчанию](/images/img41.jpg "Привилегии по умолчанию")

29. сделайте select * from testnm.t1;

```sql
select * from testnm.t1;
```

30. получилось?

Не получилось, возникла ошибка
ERROR:  permission denied for table t

![Ошибка при выполнении запроса](/images/img42.jpg "Ошибка при выполнении запроса")

31. есть идеи почему? если нет - смотрите шпаргалку

Т.к. привилегии по умолчанию будут работать для вновь создаваемых таблиц. Для того, чтобы работало для существующих, нужно или явно выдать права на таблицу, или пересоздать таблицу.

```
psql
\c testdb
drop table testnm.t1;
create table testnm.t1(c1 integer);
insert into testnm.t1(c1) value(1);
\q

psql -h localhost -U testread -d testdb
select * from testnm.t1;
```

32. сделайте select * from testnm.t1;

![Выполнение запроса без ошибки](/images/img43.jpg "Выполнение запроса без ошибки")

получилось?
Получилось!
ура!

33. теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);

```sql
create table t2(c1 integer);
insert into t2 values (2);
```

![Создание таблиц в public](/images/img44.jpg "Создание таблиц в public")

34. а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?

У каждой новой учетной записи по умолчанию есть роль public. у testread роль public есть, поэтому удалось создать таблицу и вставить данные в схеме public.

35. есть идеи как убрать эти права? если нет - смотрите шпаргалку
Необходимо отозвать роль public. Посомтрел шпаргалку. 

36. если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды

```sql
revoke CREATE on SCHEMA public FROM public; 
```
Отзыв привилегии CREATE в схеме public у роли public. После этого члены роли public не смогут создавать объекты в схеме public;

```sql
revoke all on DATABASE testdb FROM public; 
```
Отзыв всех привилегий у роли public в БД testdb;


37. теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
расскажите что получилось и почему

Создание новой таблицы невозможно, т.к. у роли public отозвана привилегия CREATE в схеме public.
Вставка данных прошла без ошибки, т.к. привилегия на вставку данных в  t2 осталась. 

![Поведение после отзыва привилегий](/images/img45.jpg "Поведение после отзыва привилегий")

Отзовем явно права на вставку у public, подключившись под postgres. Делаем для новой таблицы, т.к. для существующей t2 изменений не будет.
```sql
create table t3(c1 integer);
revoke insert on t3 from public;
```
и получаем ошибку
```sql
insert into t3 values(2);
```

![Отзыв привилегий на вставку](/images/img46.jpg "Отзыв привилегий на вставку")

