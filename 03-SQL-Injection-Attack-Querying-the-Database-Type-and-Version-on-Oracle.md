# SQL Injection Attack: Querying the Database Type and Version on Oracle

## Overview

This lab demonstrates how UNION-based SQL Injection can be used to identify the database type and retrieve version information from an Oracle database. Database fingerprinting is an important step in the exploitation process because different database management systems support different functions, tables, and syntax.

## Objective

Determine the database type and retrieve the Oracle database version information.

## Vulnerability

The application was vulnerable to SQL Injection through the category parameter.

**Vulnerable Parameter:**

```http
/category?category=Pets
```

The parameter value was incorporated directly into an SQL query without proper sanitization.

## Methodology

### Step 1: Determine the Number of Columns

The number of columns returned by the original query was identified using ORDER BY testing.

Payloads:

```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

### Observation

The first two payloads worked successfully, while increasing the column number eventually generated an error.

### Conclusion

The query returned two columns.

### Step 2: Test UNION Injection

A UNION SELECT statement was used to verify that additional data could be returned.

Payload:

```sql
' UNION SELECT NULL,NULL FROM dual--
```

### Observation

The request was processed successfully.

### Conclusion

UNION-based SQL Injection was possible.

### Step 3: Retrieve Database Version Information

Oracle stores version information in the `v$version` table.

Payload:

```sql
' UNION SELECT banner,NULL FROM v$version--
```

### Observation

The response displayed Oracle database version details.

### Result

Successfully identified:

* Database Type: Oracle
* Database Version Information

## Why It Worked

The application failed to sanitize user-controlled input before incorporating it into the SQL query. Because the number of columns was correctly identified and UNION queries were supported, data from Oracle's internal system tables could be retrieved and displayed in the application's response.

## Impact

An attacker may be able to:

* Fingerprint the database platform
* Identify vulnerable database versions
* Plan database-specific attacks
* Gather information for further exploitation

## Mitigation

* Use parameterized queries (prepared statements)
* Avoid dynamic SQL query construction
* Restrict access to system tables
* Apply least-privilege database permissions
* Perform regular security testing

## Key Concepts Learned

* UNION-Based SQL Injection
* Database Fingerprinting
* Oracle Database Enumeration
* ORDER BY Column Discovery
* Oracle System Tables
* Version Enumeration

## Tools Used

* Burp Suite Repeater
* Browser Developer Tools

## Conclusion

This lab demonstrated how UNION-based SQL Injection can be used to identify the database platform and retrieve version information. Database fingerprinting is often one of the first steps performed by an attacker because it helps determine which database-specific techniques and payloads can be used in later stages of exploitation.
