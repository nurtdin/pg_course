# Домашняя работа. Урок №1

## Виртуальная машина в Яндекс.Облако
1. создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере
![Новый проект в Яндекс.Облако](/images/img1.jpg "Проект в Яндекс.Облако")
3. далее создать инстанс виртуальной машины с дефолтными параметрами
![Экземпляр виртуальной машины в Яндекс.Облако](/images/img2.jpg "Экземпляр ВМ в Яндекс.Облако")
5. добавить свой ssh ключ в metadata ВМ
![Публичный ключ для виртуальной машины в Яндекс.Облако](/images/img3.jpg "Публичный ключ")

## Демонстрация работы уровней изоляции в СУБД Postgres
### Установка PostgreSQL, запуск приложения psql
1. зайти удаленным ssh (первая сессия), не забывайте про ssh-add
2. поставить PostgreSQL
3. зайти вторым ssh (вторая сессия)
4. запустить везде psql из под пользователя postgres
### Поведение psql с отключенной опцие auto commit
5. выключить auto commit
6. сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
7. посмотреть текущий уровень изоляции: show transaction isolation level
8. начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
9. в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
10. сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
11. завершить первую транзакцию - commit;
12. сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
13. завершите транзакцию во второй сессии
### Поведение Postgres с уровнем изоляции repeatable read
14. начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
15. в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
16. сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
17. завершить первую транзакцию - commit;
18. сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
19. завершить вторую транзакцию
20. сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
