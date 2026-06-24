# SQL Injection Attack: Listing the Database Contents on Non-Oracle Databases

## Overview

This lab demonstrates how UNION-based SQL Injection can be used to enumerate database contents on non-Oracle databases. By querying system tables, it was possible to discover table names, column names, and ultimately retrieve sensitive information stored within the database.

## Objective

Enumerate the database structure and retrieve administrator credentials from the users table.

## Vulnerability

The application was vulnerable to SQL Injection through the category parameter.

**Vulnerable Parameter:**

```http
/category?category=Gifts
```

User input was incorporated directly into an SQL query without proper sanitization.

## Methodology

### Step 1: Determine the Number of Columns

The number of columns returned by the query was identified using ORDER BY testing.

Payloads:

```sql
' ORDER BY 1#
' ORDER BY 2#
' ORDER BY 3#
```

### Observation

The first two payloads worked successfully while the third generated an error.

### Conclusion

The query returned two columns.

### Step 2: Identify Columns That Accept Text

Payload:

```sql
' UNION SELECT 'test',NULL#
```

### Observation

The text value appeared in the application's response.

### Conclusion

The first column accepted and displayed text data.

### Step 3: Enumerate Database Tables

Non-Oracle databases store metadata within the `information_schema` database.

Payload:

```sql
' UNION SELECT table_name,NULL FROM information_schema.tables#
```

### Observation

Multiple table names were displayed in the response.

### Conclusion

Database tables could be enumerated successfully.

### Step 4: Identify User Table

Among the returned tables, a table containing user information was identified.

Example:

```text
users
```

### Step 5: Enumerate Column Names

Payload:

```sql
' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users'#
```

### Observation

Column names associated with the users table were returned.

Example:

```text
username
password
```

### Step 6: Retrieve User Credentials

Payload:

```sql
' UNION SELECT username,password FROM users#
```

### Observation

Usernames and passwords were displayed in the application's response.

Administrator credentials were successfully identified.

### Step 7: Log In as Administrator

The extracted administrator credentials were used to authenticate to the application.

### Result

Successfully accessed the administrator account.

## Why It Worked

The application directly incorporated user-controlled input into an SQL query. By leveraging UNION-based SQL Injection, it was possible to query the database metadata tables, discover the database structure, identify relevant tables and columns, and extract sensitive user credentials.

## Impact

An attacker may be able to:

* Enumerate database tables and columns
* Discover sensitive application data
* Retrieve usernames and passwords
* Compromise privileged accounts
* Gain unauthorized access to application functionality

## Mitigation

* Use parameterized queries (prepared statements)
* Avoid dynamic SQL query construction
* Restrict database metadata access
* Apply least-privilege database permissions
* Conduct regular security assessments

## Key Concepts Learned

* UNION-Based SQL Injection
* Database Enumeration
* information_schema
* Table Discovery
* Column Discovery
* Credential Extraction

## Tools Used

* Burp Suite Repeater
* Browser Developer Tools

## Conclusion

This lab demonstrated how attackers can move beyond simple SQL Injection and begin enumerating the database structure. By leveraging metadata tables within information_schema, it was possible to discover table names, identify sensitive columns, extract credentials, and compromise the administrator account.
