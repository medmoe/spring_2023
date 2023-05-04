# 26/01/2023
## agenda
 - DDL, DML
 - Projection, Functions, Aggregate functions
 - CTEs
 - CTAS (create table as select ... from blah)
### DBMS review
relational algebra
SQL: structured query language
DDL: data definition language
 - create table book (
    bookid bigint,
    title string,
   );
 - drop table book;
 - drop table if exists book;
 - 
DML: data manipulation language
 - CRUD operations
    - INSERT: create new records in database
    - SELECT: get records from database
    - UPDATE: update existing records
    - DELETE: delete existing records <br>
 <u>example</u>:
    - insert into book(bookid, title) values (42, 'HHGG');
    - select title from book where bookid=42;
    - update book set title='Hichhiker\'s GUILD TO THE GALAXY where bookid=42;
    - delete from book where bookid=42;<br>
 - DDL-ish statements
   - create table bookauthors as select author from book group by author;
   - insert into bookauthors select author from otherbook group by author;<br>
 
PROJECTION: select a subset of columns
 - select author from book;
 - select substr(author, 1,1) as initial from book;
 - select now(), author, upper(title) from book;
 
FUNCTIONS:
 - SIN/COS/EXP/LOG/POW
 - string functions: substr
 - date functions: adding/subtracting dates.

AGGREGATE FUNCTIONS:
 - operate on a group for example:
   - select count (*) from book;
   - select author, count(*) from book group by author;
   - sum/min/max/count/avg/stddev
 - -- not 'exactly' the same
 - select author, sum(1) from book group by author;
CASE STATEMENTS
 - select a.*, case when author like '%DNA%' then 'Y' else 'N' end dnaflag from book;
 - select <br> sum(case when author like '%DNA%' then 1.0 <br> else 0.0 end) / sum(1.0) frac <br> from book; <br>

Some functions are missing, we have (sum) but we don't have product.

product function is just a exp sum if logs, e.g.:
 - select product(rates) <br> select exp(sum(log(rates))) <br> from

COMMON TABLE EXPRESSIONS: CTEs
 - customer(cid, name, dob, ssn, phone, email, city, state, zip) <br> with custage(<br> select a.*, <br> extract(year from age(dob)) age <br> from cutomer <br>), <br> agegrps as (<br> select a.*, <br> case when age < 18 then 'Y' <br> when age < 42 then 'M'<br> when age < 65 then 'E'<br> else 'R' <br> end agegrp <br> from custage <br>))
# 2/2/2023
 ## agenda
  - Continue review 
  - JOINS
 ### DBMS review continue
  - with custage as (select a.*, extract( year from age(dob)) age) from customer),<br>
grps as (<br> select a.*, <br> case when age < 18 then 'A'<br> when age < 42 then 'B'<br> when age < 55 then 'C' <br> when age < 65 then 'D'<br> else'E' <br> end grp <br> from custage a <br>), cntsbygrp as ( --count customers in each age group <br> select grp, count(*) cnt from grps group by grp) <br> select * from cntsbygrp;
  - inner join
  - left outer join (and right outer join)
  - full outer join 
  - cross join
  - "Modern" databses stopped supporting "natural" joins. e.g.Trino
    - Full outer join will return all records from both tables.
### examples
  - Domain: pizza shop
  - menueitem(mi, name, price, calories)
  - order(oid, cid, tim, ...)
  - orderitem(oid, oiid, mi, qty)
  - toppings(topid, name, price)
  - orderitemtoping(oiid, topid, qty) -- toppings for each order item
  - customer(cid, name, email, dob)
  - custpaymethod(pmntid, cid, name, ccn, ...) -- customer payment method
  - orderpayment(oid, type, amnt, ccn)
  
how much money did john doe spend on pizza last year?
```sql
select sum(amnt) tot 
from customer a 
inner join order before on a.cid=b.cid
inner join order
```

whats the most popular menuitem ?
```sql
with cnts as (
    select mi, sum(qty) cnt
    from orderitem
    group by mi
), 
maxcnt as (
    select max(cnt) as mcnt
    from cnts
)
select mi --return the mi with maximum count
from cnts inner join maxnt on a.cnt=b.mcnt;
```
what's the most popular (liked by most customers) menuitem
```sql
select mi, count(distinct ccn)
from orderitem a 
    inner join orderpayment b 
    on a.oid=b.oid
    group by mi
```
find customers who order pizza for more than 40 wwks out of a year(for last year). customers who eat pizza every week, for at least 40 weeks in a year
```sql
with oweek as (
    select distinct a.cid,
    EXTRACT ('week' from tim) wnum
    from order a 
    inner join orderitem b 
    on a.oid=b.oid
    inner join menuitem c on b.mi=c.mi and c.name='pizza'
)
select cid, from oweek group by cid having count(*) >= 40;
```
# 09/02/2023
## agenda
 - Window functions
 - Indexes

## indexes
 - bitmap indexes are useful if the data is sorted.
 - explain select from tabl1 inner join tabl2 on -> that's how to figure out what type of join is running
 - cross join or a join without a clear key e.g. non-equality join conditions
## window functions
 -
# 16/02/2023
## agenda
 - hierarchy

# 23/02/2023
## agenda
 - go over the homework
## hw2 
 - list the top 1 % of customers
```sql
with balances as (
    select b.customerid, sum(amount) bal
    from transaction a inner join account b 
    on a.accountid=b.accountid
    group by b.customerid
),
rownums as (
    select a.*, row_number() over (order by bal) rn, count() over() cnt
    from balances
),
percentiles as (
    select a.*, 100.0 * rn/cnt as prcntl from rownums a
)
select * from percentiles where prcntl >= 99;
```
 - useful link techcrunch.com/2023/02/23/sample-pre-seed-pitch...
## partitions
 - create partitions as tables for quick selection. <br>
1 1 1 1 0 0 0 0 <br>
1 1 1 1 0 0 0 0 <br>
1 1 1 1 1 1 1 1 <br>
1 1 1 1 1 1 1 1 <br>
1 1 1 1 0 0 0 0 <br>
1 1 1 1 0 0 0 0 <br>
# 03/02/2023
 ## agenda
 - Big Data
 - Hadoop
 - Spark
 - MPP DBs (Netezza)
 - NO-SQL
 ## To do
 - install Hadoop
# 03/09/2023
 ## agenda
 - Big data
 - finding hash code of integers is faster than strings.
 - when a table is small we can replicate it in every single machine for faster processing.
# 04/27/2023
 ## agenda
- HBase
### Implementation
for small data we use hash table in a RAM. However, for large <br>
data we need to use multiple computers where each computer <br>
is assigned a region.<br>
Similarly, the data is stored in the disk as regions where each region<br>
is divided into files and there are column families.<br>
