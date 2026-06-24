# SQL Injection Attack: Querying the Database Type and Version on MySQL and Microsoft SQL Server

## Overview

This lab demonstrates how UNION-based SQL Injection can be used to identify the database type and retrieve version information from MySQL and Microsoft SQL Server databases. Obtaining database version details helps attackers understand the target environment and identify database-specific attack techniques.

## Objective

Determine the database type and retrieve version information from the backend database.

## Vulnerability

The application was vulnerable to SQL Injection through the category parameter.

**Vulnerable Parameter:**

```http
/category?category=Gifts
```

User input was directly incorporated into an SQL query without proper sanitization.

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

The first two payloads were successful while increasing the column number eventually generated an error.

### Conclusion

The query returned two columns.

### Step 2: Verify UNION-Based SQL Injection

A UNION SELECT statement was used to confirm that additional data could be returned.

Payload:

```sql
' UNION SELECT NULL,NULL#
```

### Observation

The request was processed successfully.

### Conclusion

UNION-based SQL Injection was possible.

### Step 3: Retrieve Database Version Information

MySQL and Microsoft SQL Server expose version information through the `@@version` variable.

Payload:

```sql
' UNION SELECT @@version,NULL#
```

### Observation

The application's response displayed database version information.

### Result

Successfully identified:

* Database Platform
* Database Version Information

## Why It Worked

The application directly incorporated user input into an SQL query. After determining the correct number of columns and confirming UNION injection, it was possible to retrieve data from database system variables. The `@@version` variable exposed information about the backend database platform and version.

## Impact

An attacker may be able to:

* Fingerprint the database platform
* Identify outdated or vulnerable database versions
* Develop database-specific exploitation strategies
* Gather intelligence for further attacks

## Mitigation

* Use parameterized queries (prepared statements)
* Avoid dynamic SQL query construction
* Restrict unnecessary database information disclosure
* Apply least-privilege database permissions
* Conduct regular security testing

## Key Concepts Learned

* UNION-Based SQL Injection
* Database Fingerprinting
* MySQL Enumeration
* Microsoft SQL Server Enumeration
* ORDER BY Column Discovery
* Version Enumeration

## Tools Used

* Burp Suite Repeater
* Browser Developer Tools

## Conclusion

This lab demonstrated how UNION-based SQL Injection can be leveraged to identify the backend database platform and retrieve version information. Database fingerprinting is a critical reconnaissance step that helps attackers understand the target environment and select appropriate exploitation techniques.
