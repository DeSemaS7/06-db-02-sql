### Задача 1

docker run -e POSTGRES_PASSWORD=postgres -e PGDATA=/var/lib/postgresql/data -p 5432:5432 -v /tmp/postgres/data:/var/lib/postgresql/data -v /tmp/postgres/backups:/var/lib/postgresql/backups -d postgres

### Задача 2


test_db=# \l
                                <br> List of databases

| Name      | Owner    | Encoding | Collate    | Ctype      | Access privileges                                                     |
|-----------|----------|----------|------------|------------|-----------------------------------------------------------------------|
| postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |                                                                       |
| template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres + postgres=CTc/postgres                                   |
| template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres + postgres=CTc/postgres                                   |
| test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres + postgres=CTc/postgres + "test-admin-user"=CTc/postgres |
(4 rows)

<br>
test_db-# \d+ orders<br>
Table "public.orders"<br>

| Column       | Type    | Collation | Nullable | Default                            | Storage  | Stats target | Description |
|--------------|---------|-----------|----------|------------------------------------|----------|--------------|-------------|
| id           | integer |           | not null | nextval('orders_id_seq'::regclass) | plain    |              |             |
| наименование | text    |           |          |                                    | extended |              |             |
| цена         | numeric |           |          |                                    | main     |              |             |
<p>Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
<p>Referenced by:
    TABLE "clients" CONSTRAINT "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)
<p>Access method: heap

<br>
<br>
<br>
test_db-# \d+ clients
<p> Table "public.clients"

| Column            | Type    | Collation | Nullable | Default                            | Storage  | Stats target | Description |
|-------------------|---------|-----------|----------|------------------------------------|----------|--------------|-------------|
| id                | integer |           | not null | nextval('orders_id_seq'::regclass) | plain    |              |             |
| фамилия           | text    |           |          |                                    | extended |              |             |
| страна проживания | заказ   |           |          |                                    | extended |              |             |
| заказ             | заказ   |           |          |                                    | extended |              |             |

<p>Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "clients_страна проживания_idx" btree ("страна проживания")
<p>Foreign-key constraints:
    "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)
<p>Access method: heap

<br>
<br>
<br>
SELECT grantee,table_catalog, table_schema, table_name, privilege_type
FROM   information_schema.table_privileges 
WHERE  grantee not in ('postgres','PUBLIC');

  |      grantee     | table_name | privilege_type |
|:----------------:|:----------:|:--------------:|
| test-admin-user  | orders     | INSERT         |
| test-admin-user  | orders     | SELECT         |
| test-admin-user  | orders     | UPDATE         |
| test-admin-user  | orders     | DELETE         |
| test-admin-user  | orders     | TRUNCATE       |
| test-admin-user  | orders     | REFERENCES     |
| test-admin-user  | orders     | TRIGGER        |
| test-admin-user  | clients    | INSERT         |
| test-admin-user  | clients    | SELECT         |
| test-admin-user  | clients    | UPDATE         |
| test-admin-user  | clients    | DELETE         |
| test-admin-user  | clients    | TRUNCATE       |
| test-admin-user  | clients    | REFERENCES     |
| test-admin-user  | clients    | TRIGGER        |
| test-simple-user | orders     | INSERT         |
| test-simple-user | orders     | SELECT         |
| test-simple-user | orders     | UPDATE         |
| test-simple-user | orders     | DELETE         |
| test-simple-user | clients    | INSERT         |
| test-simple-user | clients    | SELECT         |
| test-simple-user | clients    | UPDATE         |
| test-simple-user | clients    | DELETE         |


### Задача 3

INSERT INTO orders (наименование, цена) VALUES ('Шоколад',10),('Принтер',3000),('Книга',500),('Монитор',7000),('Гитара',4000);
INSERT INTO clients ("фамилия", "страна проживания") VALUES ('Иванов Иван Иванович','USA'),('Петров Петр Петрович','Canada'),('Иоганн Себастьян Бах','Japan'),('Ронни Джеймс Дио','Russia'),('Ritchie Blackmore','Russia');

select count(*) from orders;
 count
---
     5
(1 row)
<br>
<br>
select count(*) from clients;
 count
---
     5
(1 row)

### Задача 4

update clients
set заказ = orders.наименование
from orders
where clients.id = 1
and orders.id = 3;

update clients
set заказ = orders.наименование
from orders
where clients.id = 2
and orders.id = 4;

update clients
set заказ = orders.наименование
from orders
where clients.id = 3
and orders.id = 5;
<br><br><br>
test_db=# select фамилия from clients where "заказ" is not null;
       
| фамилия              |
|----------------------|
| Иванов Иван Иванович |
| Петров Петр Петрович |
| Иоганн Себастьян Бах |
| (3 rows)             |

### Задача 5

test_db=# explain select фамилия from clients where "заказ" is not null;
                        QUERY PLAN
-----------------------------------------------------------
 Seq Scan on clients  (cost=0.00..16.30 rows=627 width=32)
   Filter: ("заказ" IS NOT NULL)
(2 rows)
<br>видим что идёт последовательное сканирование таблицы клиент с применением фильтра "заказ" IS NOT NULL

### Задача 6
pg_dump -U postgres test_db > /var/lib/postgresql/backups/test_db.sql
<p>psql -U postgres test_db < /var/lib/postgresql/backups/test_db.sql


