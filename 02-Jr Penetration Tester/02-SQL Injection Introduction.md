**TryHackMe Room:** [SQL Injection Introduction](https://tryhackme.com/room/sqlinjectionintroduction)

This write-up covers my understanding of the room and the key concepts I took away from it.

# SQL Injection Introduction

SQL Injection (SQLi) is a very famous web vulnerability. I have heard of this since i started using Linux back in university. And i was very familiar (just familiar) with the tool called `sqlmap` from Kali, but I didn't use the tool at all, nor I tried to learn about the vulnerability back then. From what I heard back then, I used to think SQLi is a command line injection to databases, so that the attackers would able to gain unauthorized access to sensitive data.

But now, years later, I am learning the concept and getting hands on it on TryHackMe.

## SQL Essentials for Injections

### SQL Comments

There are two ways you can put comments on `SQL`. `--` and `/* */`.  The former is used for quick and easy one-liner comment whereas you can use the latter for both one-liner and for multiple lines.

For example, here is a normal code:

```sql-text
SELECT * FROM users WHERE username='INPUT' AND password='secret';
```

And here is the same code with comment in the middle:

```sql-text
SELECT * FROM users WHERE username='admin'-- AND password='secret';
```

The commands after `--` is commented out so they are ignored.

### UNION

```sql-text
SELECT name, email FROM users UNION SELECT id, name FROM admins; 
```

And it will return name and email from `users` table and id and name from `admins` table and show them as total result.

### LIKE And Wildcards

```sql-text
SELECT * FROM users WHERE username LIKE "adm%";
```

The returns would be users with the username `adm` at the beginning like admin, administrator, etc.

### LIMIT

```sql-text
SELECT name FROM users LIMIT 1;
SELECT name FROM users LIMIT 2, 1;
```

The first one will show only the first row of the table. The second one will skip the first two rows and show the third one.

### String Functions

1. `group-concat()`

```sql-text
SELECT group_concat(username, ':', password SEPARATOR '<br>') FROM users;
```

It will return `admin:pass123<br>martin:secret<br>jim:work456`.

2. `CONCAT()`

```sql-text
SELECT CONCAT(username, ':', password) FROM users;
```

It will return `admin:pass123`.

And there is also `informatino_schema` database which is a database that contains information about other databases. Every database has it. And two tables from the `information_schema` database are useful.

- `information_schema.tables` lists every tables.
- `information_schema.columns` lists every columns.

## What is SQL Injection?

So, how does SQL injection work? And what is it exactly? Is there a magic code that lets you access unauthorized data? Well, I used to think so. I used to think, you are in a database, and you enter a magic code or codes, and then it will automatically open the tables for you. But it doesn't work like that.

So, in a normal article website, when you select an article like this `https://website.thm/article?id=1,`, behind the scene, the SQL translates this to:

```sql-text
SELECT * FROM articles WHERE id = 1 AND public = 1;
```

But here is where the vulnerability happens when building the query by directly concatenating user input directly into the SQL string, like this:

```php
$query = "SELECT * FROM articles WHERE id = " . $_GET['id'] . " AND public = 1;";
```

So, whatever id you select, becomes part of the SQL query. And you can manipulate it in this way:

```sql-text
SELECT * FROM articles WHERE id = 1 OR 1=1-- AND public = 1;
```
This means get everything from the table named `articles` where one or both these clauses are true `id=1`, `1=1`. So, we all know that `1=1`, so, the result would include everything from the table `articles` even when it's not public because the public part is commented out.

There are three types of SQL injections:

- In-Band SQL Injection
- Blind SQL Injection
- Out-of-Band SQL Injection

Here are the basic commands to detect SQL injection

- `'`
- `"`
- `;--`
- `OR 1=1`

But not every website or database will be set to return errors, you have to check them yourself.

## In-Band SQL Injection

## Blind SQL Injection: Authentication Bypass

## Blind SQL Injection: Boolean and Time Based

## Out-of-Band SQL Injection

## Remediation and Prevention

## Practical

The practical lab of this room is pretty straightforward, and there is a step by step guide on the room itself. So, I won't be including this.

## Conclusion
