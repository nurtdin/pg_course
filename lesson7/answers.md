# �������� ������. ���� �7

## ����� ������� PostgresSQL 14

1. �������� ����� ������� PostgresSQL 14

����� ������� Postgres 14 ������ � ������� �������:

```console
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-14
```
��������� ���������:
![��������� ��������� Postgres 14](/images/img23.jpg "��������� ��������� Postgres 14")

2. ������� � ��������� ������� ��� ������������� postgres
```console
sudo su postgres
cd /var/lib/postgresql
psql
```

3. ����������� � ������� psql:

![����������� � ������� psql](/images/img24.jpg "����������� � ������� psql")

# ����� ���� ������ testdb. ������������ ���������� ���������� � Postgres � �� �����������
1. �������� ����� �� testdb

```sql
create database testdb;
```

![�������� ����� ��](/images/img25.jpg "�������� ����� ��")

2. ������� � ��������� ���� ������ ��� ������������� postgres

```sql
\c testdb
```

![����������� � ��](/images/img26.jpg "����������� �  ��")

3. �������� ����� ����� testnm

```sql
create schema testnm;
```

![��������� �����](/images/img27.jpg "�������� �����")


4. �������� ����� ������� t1 � ����� �������� c1 ���� integer

```sql
create table testnm.t1(c1 integer);
```

![��������� ������� t1](/images/img28.jpg "�������� ������� t1")

5. �������� ������ �� ��������� c1=1

```sql
testdb=# insert into testnm.t1(c1) values(1);
```

![������� ������ � ������� t1](/images/img29.jpg "������� ������ � ������� t1")

6. �������� ����� ���� readonly

```sql
create role readonly;
```

![�������� ����](/images/img30.jpg "�������� ����")

7. ����� ����� ���� ����� �� ����������� � ���� ������ testdb

```sql
grant connect on database testdb to readonly;
```

![���������� �� ����������� � ��](/images/img31.jpg "���������� �� ����������� � ��")

9. ����� ����� ���� ����� �� ������������� ����� testnm

```sql
grant usage on schema testnm to readonly;
```

![���������� �� ������������� �����](/images/img32.jpg "���������� �� ������������� �����")

10. ����� ����� ���� ����� �� select ��� ���� ������ ����� testnm

```sql
grant select on all tables in schema testnm to readonly;
```

![���������� �� ������� �� ���� ������ �����](/images/img33.jpg "���������� �� ������� �� ���� ������ �����")

11. �������� ������������ testread � ������� test123

```sql
create user testread with password 'test123';
```

![�������� ������������](/images/img34.jpg "�������� ������������")

12. ����� ���� readonly ������������ testread

```sql
grant readonly to testread;
```

![������ ����](/images/img35.jpg "������ ����")

13. ������� ��� ������������� testread � ���� ������ testdb

```cpnsole
psql -h localhost -U testread -d testdb
```

![����������� � �� ��� �������� �������������](/images/img36.jpg "����������� � �� ��� �������� �������������")

14. �������� select * from t1;

![��������� �������](/images/img37.jpg "��������� �������")

15. ����������? (����� ���� �� ������ ���� �� �� ��������� � �� �������� ���� ������������ ������ ��� ������� �����)

�� ����������.

16. �������� ��� ������ ��������� � ������ ��������� �������

������� ������� � ����� testnm, � ��� ����������

```sql
select * from t1;
```

������� "��������" � ����� public.

17. � ��� ���� ���� ������? ���� ����� �� ����?

����� ������ �� ������� � ����� testnm, � ������� �� �� ����������. ��������� ���������� � ������� public.t1, �.�. ��� ����� ���� �� �������. � ����� public ������� t1 ���, ������� �������� ������:

ERROR:  relation "t1" does not exist

���� �� ������� ������� ��� �������� �����, �� ��� ���� �� ������� � ����� public, �.�. ���� public �� ��������� �������� ������� ������������.

18. ���������� �� ������ ������

```sql
\dt
```

![������ ������](/images/img38.jpg "������ ������")

������ ������ � ����� public ������.

19. ��������� � ��������� ��� ������� 20

� ������ ��� ���������� � �������� (���� ������ ���� � ��� ��������� �� ����� � ��� ��� ���������)
����� ��� ��������� ��� ����� ����� �������� 
```sql
select * from testnm.t1;
```
��� ������ ������� ����� �� testnm
```sql
SET search_path TO testnm, public;
select * from t1;
```

![��������� � ��������� �����](/images/img39.jpg "��������� � ��������� �����")

20. ��������� � ���� ������ testdb ��� ������������� postgres

```console
sudo su postgres
psql
```

21. ������� ������� t1

```sql
drop table testnm.t1;
```

22. �������� �� ������ �� ��� � ����� ��������� ����� ����� testnm

```sql
create table testnm.t1(c1 integer);
```

23. �������� ������ �� ��������� c1=1

```sql
insert into testnm.t1(c1) values(1);
```

24. ������� ��� ������������� testread � ���� ������ testdb

```console
psql -h localhost -U testread -d testdb
```

25. �������� select * from testnm.t1;

```sql
select * from testnm.t1;
```

26. ����������?

�� ����������, �������� ������

![������ ��� ���������� �������](/images/img40.jpg "������ ��� ���������� �������")

27. ���� ���� ������? ���� ��� - �������� ���������

�.�. �� ���������� ���������� �� ���������. ��. https://www.postgresql.org/docs/current/sql-alterdefaultprivileges.html
���������� �� ������� �� ���� ������ � ����� testnm ���� ������ ����� �������� ������� t1. � ���������� ������� ���� �����������, ���������� �� ��� ������ �� ����. ��������� Postgres by design.

28. ��� ������� ��� ����� ����� ������ �� �����������? ���� ��� ���� - �������� ���������

���������� ������ ���������� �� ���������:

```sql
alter default privileges in schema testnm grant select on tables to readonly;
```

![���������� �� ���������](/images/img41.jpg "���������� �� ���������")

29. �������� select * from testnm.t1;

```sql
select * from testnm.t1;
```

30. ����������?

�� ����������, �������� ������
ERROR:  permission denied for table t

![������ ��� ���������� �������](/images/img42.jpg "������ ��� ���������� �������")

31. ���� ���� ������? ���� ��� - �������� ���������

�.�. ���������� �� ��������� ����� �������� ��� ����� ����������� ������. ��� ����, ����� �������� ��� ������������, ����� ��� ���� ������ ����� �� �������, ��� ����������� �������.

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

32. �������� select * from testnm.t1;

![���������� ������� ��� ������](/images/img43.jpg "���������� ������� ��� ������")

����������?
����������!
���!

33. ������ ���������� ��������� ������� create table t2(c1 integer); insert into t2 values (2);

```sql
create table t2(c1 integer);
insert into t2 values (2);
```

![�������� ������ � public](/images/img44.jpg "�������� ������ � public")

34. � ��� ���? ��� �� ����� ���� �� �������� ������ � insert � ��� ��� ����� readonly?

� ������ ����� ������� ������ �� ��������� ���� ���� public. � testread ���� public ����, ������� ������� ������� ������� � �������� ������ � ����� public.

35. ���� ���� ��� ������ ��� �����? ���� ��� - �������� ���������
���������� �������� ���� public. ��������� ���������. 

36. ���� �� ���������� ���� �� ���������� ��� ������� � ������, ���� �������� ��������� - ��������� ��� ������� � ������ �������� ��������� � ��� �������

```sql
revoke CREATE on SCHEMA public FROM public; 
```
����� ���������� CREATE � ����� public � ���� public. ����� ����� ����� ���� public �� ������ ��������� ������� � ����� public;

```sql
revoke all on DATABASE testdb FROM public; 
```
����� ���� ���������� � ���� public � �� testdb;


37. ������ ���������� ��������� ������� create table t3(c1 integer); insert into t2 values (2);
���������� ��� ���������� � ������

�������� ����� ������� ����������, �.�. � ���� public �������� ���������� CREATE � ����� public.
������� ������ ������ ��� ������, �.�. ���������� �� ������� ������ �  t2 ��������. 

![��������� ����� ������ ����������](/images/img45.jpg "��������� ����� ������ ����������")

������� ���� ����� �� ������� � public, ������������� ��� postgres. ������ ��� ����� �������, �.�. ��� ������������ t2 ��������� �� �����.
```sql
create table t3(c1 integer);
revoke insert on t3 from public;
```
� �������� ������
```sql
insert into t3 values(2);
```

![����� ���������� �� �������](/images/img46.jpg "����� ���������� �� �������")

