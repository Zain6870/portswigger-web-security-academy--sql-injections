# SQL Injection UNION Attack: Finding a Column Containing Text

## Overview

This lab demonstrates the second stage of a UNION-based SQL Injection attack. After determining the number of columns returned by the original query, the next objective is to identify which columns can store and display text values. This is necessary because sensitive information such as usernames, passwords, and database version details are typically stored as text.

## Objective

Identify which columns returned by the SQL query accept and display text values.

## Vulnerability

The application was vulnerable to SQL Injection through the category parameter.

**Vulnerable Parameter:**

```http
/category?category=Gifts
```

The application incorporated user-controlled input directly into an SQL query without proper sanitization.

## Methodology

### Step 1: Determine the Number of Columns

From the previous lab, it was determined that the query returned two columns.

### Step 2: Test the First Column

Payload:

```sql
' UNION SELECT 'hello',NULL-- -
```

### Observation

The value **hello** appeared in the application's response.

### Conclusion

The first column accepted and displayed text values.

### Step 3: Test the Second Column

Payload:

```sql
' UNION SELECT NULL,'hello'-- -
```

### Observation

The value **hello** appeared in the application's response.

### Conclusion

The second column also accepted and displayed text values.

## Result

Successfully identified that both columns accepted and displayed text values.

## Why It Worked

A UNION SELECT statement combines the results of two SQL queries. For the attack to succeed, both queries must return the same number of columns and compatible data types. By injecting a known text value into each column individually, it was possible to determine which columns accepted string values and displayed them in the application's response.

## Impact

An attacker may be able to:

- Identify columns suitable for displaying sensitive information
- Retrieve usernames and passwords
- Extract database version information
- Display data from other database tables

## Mitigation

- Use parameterized queries (prepared statements)
- Avoid dynamic SQL query construction
- Validate and sanitize user input
- Restrict database error disclosure
- Perform regular security testing

## Key Concepts Learned

- UNION-Based SQL Injection
- Text Column Identification
- Data Type Compatibility
- UNION SELECT
- Displayable Columns

## Tools Used

- Burp Suite Repeater
- Browser Developer Tools

## Conclusion

This lab demonstrated how attackers identify columns capable of displaying text values during a UNION-based SQL Injection attack. Identifying these columns is an essential step before retrieving sensitive information from other database tables.
