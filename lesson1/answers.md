# Домашняя работа. Урок №1

## Виртуальная машина в Яндекс.Облако
1. создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере

Выбрано Яндекс.Облако, в котором создан новый проект nurtdincloud:
![Новый проект в Яндекс.Облако](/images/img1.jpg "Проект в Яндекс.Облако")

2. далее создать инстанс виртуальной машины с дефолтными параметрами

Создана новая виртуальная машина ppstgresdb с минимальными характеристиками:
![Экземпляр виртуальной машины в Яндекс.Облако](/images/img2.jpg "Экземпляр ВМ в Яндекс.Облако")

3. добавить свой ssh ключ в metadata ВМ

В Windows 10 с помощью ssh-keygen создана пара публичный/приватный ключи, публичный ключ укзан при создании виртутальной машины:
![Публичный ключ для виртуальной машины в Яндекс.Облако](/images/img3.jpg "Публичный ключ")

## Демонстрация работы уровней изоляции в СУБД Postgres
### Установка PostgreSQL, запуск приложения psql
1. зайти удаленным ssh (первая сессия), не забывайте про ssh-add

При подключении не потребовалось использовать УЗ Ubuntu, подключение с Windows-машины прошло под своей УЗ.
![Первая сессия через SSH](/images/img4.jpg "Первая сессия через SSH")

2. поставить PostgreSQL
Установка выполнена с помощью команды:
```console
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -0 - hhtps://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update  && sudo apt-get -y install postgresql-15
```
![Результат установки Postgres 15](/images/img5.jpg "Результат установки Postgres 15")

3. зайти вторым ssh (вторая сессия)

![Вторая сессия через SSH](/images/img6.jpg "Вторая сессия через SSH")

4. запустить везде psql из под пользователя postgres
Запуск осуществлен с помощью
```console
cd /etc/postgresql
sudo -u postgres psql
```
Первая сессия psql:

![Первая сессия](/images/img7.jpg "Первая сессия")

Вторая сессия psql:

![Вторая сессия](/images/img8.jpg "Вторая сессия")

При подлкючении из домашнего каталога пользователя nurtdin получал предупреждение:

![Подключение из домашнего каталога](/images/img9.jpg "Подключение из домашнего каталога")

**Вопрос:** Почему при инсталляции Postgres в Ubuntu не создался профиль пользователя Postgres? И каким образом можно найти файл .psqlrc?

### Поведение psql с отключенной опцие auto commit
5. выключить auto commit

Отключаем AUTOCOMMIT в обеих сессиях, предварительно проверив текущее состояние:

![Значение по умолчанию AUTOCOMMIT](/images/img10.jpg "Значение по умолчанию AUTOCOMMIT")

![Отключение AUTOCOMMIT](/images/img11.jpg "Отключение AUTOCOMMIT")

6. сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

Выполнены dml:
```sql
create table persons(id serial, first_name text, second_name text); 
insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
insert into persons(first_name, second_name) values('petr', 'petrov'); 
commit;
```
![Вставка данных и фиксация](/images/img12.jpg "Вставка данных и фиксация")

7. посмотреть текущий уровень изоляции: show transaction isolation level

```sql
show transaction isolation level;
```
![Текущий уровень изоляции](/images/img13.jpg "Текущий уровень изоляции")

8. начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции

```sql
begin;
```
9. в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');

```sql
insert into persons(first_name, second_name) values('sergey', 'sergeev');
```

![Вставка данных в первой сессии](/images/img14.jpg "Вставка данных в первой сессии")

10. сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?

```sql
select * from persons;
```
![Выборка данных во второй сессии](/images/img15.jpg "Выборка данных во второй сессии")

Новая запись, вставленная в первой сессии, не видна, т.к. при уровне изоляции "read committed" предотвращается чтение "грязных" данных, т.е. данных, которые были изменены, но не зафиксированы другими транзакциями. В данном случае в первой сессии транзакция изменила данны (осуществила вставку), но осталась незавершенной (явно не выполнили commit).

11. завершить первую транзакцию - commit;
```sql
commmit;
```
![Фиксация транзакции в первой сессии](/images/img16.jpg "Фиксация транзакции в первой сессии")

12. сделать select * from persons во второй сессии

```sql
select * from persons;
```

![Выборка данных во второй сессии](/images/img17.jpg "Выборка данных во второй сессии")

видите ли вы новую запись и если да то почему?

Новая запись, вставленная в первой сессии, видна, т.к. при выполнении commit транзакция в первой сессии была завершена, измененные данные стали доступна в других сессиях.
13. завершите транзакцию во второй сессии
```sql
commit;
```
### Поведение Postgres с уровнем изоляции repeatable read
14. начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;

В обеих сессиях установлен уровень изоляции repeatable read:
```sql
set transaction isolation level repeatable read;
```
![Уровень изоляции](/images/img18.jpg "Уровень изоляции")

15. в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');

```sql
insert into persons(first_name, second_name) values('sveta', 'svetova');
```
![Вставка данных в первой сессии](/images/img19.jpg "Вставка данных в первой сессии")

16. сделать select * from persons во второй сессии

```sql
select * from persons;
```
![Выборка данных во второй сессии](/images/img20.jpg "Выборка данных во второй сессии")

видите ли вы новую запись и если да то почему?

Новая запись, вставленная в первой сессии, не видна, т.к. при уровне изоляции "repeatable read" транзакция не видит изменения данных, которые ранее не были ею прочитаны. И тем более не видит данные, которые были вставлены, но не зафиксированы.
17. завершить первую транзакцию - commit;

```sql
commit;
```

![Фиксация транзакции в первой сессии](/images/img21.jpg "Фиксация транзакции в первой сессии")
18. сделать select * from persons во второй сессии

```sql
select * from persons;
```

видите ли вы новую запись и если да то почему?
Новая запись, вставленная в первой сессии видна, т.к. транзакция в первой сессии была зафиксирована.

18. завершить вторую транзакцию

```sql
commit;
```
21. сделать select * from persons во второй сессии

```sql
select * from persons;
```

![Выборка данных во второй сессии](/images/img22.jpg "Выборка данных во второй сессии")
видите ли вы новую запись и если да то почему?
Новая запись видна, т.к. никаких незафиксированных транзакций, изменяющих эту строку, нет.
