1. What is the balance of accountid=42?
```sql
SELECT SUM(amount) as balance 
FROM transaction 
WHERE accountid = 42;
```
2. What was the transaction amount of transactionid=42?
```sql
SELECT amount
FROM transaction 
WHERE transactionid= 42;
```
3. Which transactionids do not sum up to zero (are invalid)?
```sql
WITH tran_groups AS (
    SELECT transactionid, SUM(amount) as total
    FROM transaction 
    GROUP BY transactionid
)
SELECT transactionid FROM tran_groups
WHERE total <> 0;
```
4. List of customers without accounts?
```sql
WITH accounts AS (
    SELECT customerid FROM account
)
SELECT customerid, username, fname, lname
FROM customer
WHERE customerid NOT IN (accounts)
```
5. What is the balance (total across all accounts) for customerid=42?
```sql
SELECT SUM(a.amount) as balance
FROM transaction a
JOIN acount b ON a.accountid = b.accountid
WHERE b.customerid = 42;
```
6. What is the total balance of all customers living in zip code 10001?
```sql
SELECT SUM(t.amount) as total_balance
FROM transaction t
JOIN account a ON t.accountid = a.accountid
JOIN customer c ON a.customerid = c.customerid
WHERE c.zip = 10001;
```
7. Which zip code has the highest balance?
```sql
SELECT c.zip, SUM(t.amount) as total_balance
FROM transaction t
JOIN account a ON t.accountid = a.accountid
JOIN customer c ON a.customerid = c.customerid
GROUP BY c.zip
ORDER BY total_balance DESC
LIMIT 1;
```
8. List the top 1% of customers (ordered by balance)?
```sql
WITH customers_balance AS (
  SELECT c.customerid, c.fname, c.lname, SUM(t.amount) as total_balance
  FROM transaction t
  JOIN account a ON t.accountid = a.accountid
  JOIN customer c ON a.customerid = c.customerid
  GROUP BY c.customerid
)
SELECT customerid, fname, lname, total_balance
FROM customers_balance
ORDER BY total_balance DESC
LIMIT (SELECT CEIL(COUNT(*) * 0.01) FROM customers_balance);

```
12. For each account, what was the closing balance on December 31, 2022?
```sql
SELECT accountid, SUM(amount) as closing_balance
FROM transaction
WHERE trantimestamp BETWEEN '2022-12-01' AND '2022-12-31'
GROUP BY accountid;

```
13. What percentage of bank's money is held by people in the tri-state area today?(NY, NJ, CT)?
```sql
WITH customer_balances AS (
  SELECT customerid, SUM(amount) as total_balance
  FROM account
  JOIN transaction ON account.accountid = transaction.accountid
  JOIN customer ON account.customerid = customer.customerid
  WHERE state IN ('NY', 'NJ', 'CT')
  GROUP BY customerid
)
SELECT SUM(total_balance) / (SELECT SUM(total_balance) FROM customer_balances) * 100 as percentage_of_banks_money
FROM customer_balances;
```
### The next questions are being solved with the help of ChatGpt
11. Write a query to add 0.01% to each savings account(note that the money has to be accounted for)? 
```sql
WITH updated_accounts AS (
  SELECT accountid, (1 + 0.0001) * SUM(amount) as new_balance
  FROM account
  JOIN transaction ON account.accountid = transaction.accountid
  WHERE description = 'savings'
  GROUP BY accountid
)
INSERT INTO transaction (transactionid, trantimestamp, accountid, amount)
SELECT (SELECT MAX(transactionid) FROM transaction) + ROW_NUMBER() OVER (ORDER BY updated_accounts.accountid),
       CURRENT_TIMESTAMP, accountid, new_balance - SUM(amount)
FROM updated_accounts
JOIN transaction ON updated_accounts.accountid = transaction.accountid
GROUP BY accountid, new_balance;
```
9. Using balances for previous two months, predict what the balances will be next month. (tip: find slope of a line; x-axis is days, y-axis is balance. 2 previous months means you have 2 points, finding slope is easy. Use slope to predict where next month's balance will be.)<br>
```sql
WITH previous_months AS (
  SELECT accountid, trantimestamp, SUM(amount) as balance,
         ROW_NUMBER() OVER (PARTITION BY accountid ORDER BY trantimestamp) as row_num
  FROM transaction
  WHERE trantimestamp BETWEEN '2022-12-01' AND '2023-01-31'
  GROUP BY accountid, trantimestamp
)
SELECT accountid, (balance - LAG(balance) OVER (PARTITION BY accountid ORDER BY trantimestamp)) /
                 (trantimestamp - LAG(trantimestamp) OVER (PARTITION BY accountid ORDER BY trantimestamp)) * 30 + balance as predicted_balance
FROM previous_months
WHERE row_num = 2;

```
10. List top 10 fastest growing accounts (using previous 2 months). (tip: same as above, fastest growing means steepest slope).
```sql
WITH previous_months AS (
  SELECT accountid, trantimestamp, SUM(amount) as balance,
         ROW_NUMBER() OVER (PARTITION BY accountid ORDER BY trantimestamp) as row_num
  FROM transaction
  WHERE trantimestamp BETWEEN '2022-12-01' AND '2023-01-31'
  GROUP BY accountid, trantimestamp
), slopes AS (
  SELECT accountid, (balance - LAG(balance) OVER (PARTITION BY accountid ORDER BY trantimestamp)) /
                   (trantimestamp - LAG(trantimestamp) OVER (PARTITION BY accountid ORDER BY trantimestamp)) * 30 as slope
  FROM previous_months
  WHERE row_num = 2
)
SELECT accountid, slope
FROM slopes
ORDER BY slope DESC
LIMIT 10;

```