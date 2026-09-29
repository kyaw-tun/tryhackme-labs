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

## In-Band SQL Injection

## Blind SQL Injection: Authentication Bypass

## Blind SQL Injection: Boolean and Time Based

## Out-of-Band SQL Injection

## Remediation and Prevention

## Practical

## Conclusion
