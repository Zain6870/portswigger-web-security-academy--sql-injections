# SQL Injection UNION Attack: Determining the Number of Columns Returned by the Query

## Overview

This lab demonstrates the first stage of a UNION-based SQL Injection attack: determining the number of columns returned by the original SQL query. Before a UNION attack can be performed successfully, the injected query must return the same number of columns as the original query. This process is commonly referred to as **column count discovery**.

## Objective

Determine the number of columns returned by the application's SQL query.

## Vulnerability

The application was vulnerable to SQL Injection through the `category` parameter.

**Vulnerable Parameter:**

```http
/category?category=Gifts
```

User-controlled input was directly concatenated into an SQL query without proper sanitization or parameterization.

---

## Methodology

### Step 1: Verify SQL Injection

A single quote (`'`) was appended to the parameter value to determine whether the application was vulnerable to SQL Injection.

**Payload:**

```sql
'
```

### Observation

The application generated an SQL error, confirming that user input was interpreted as part of the SQL statement.

---

### Step 2: Determine the Number of Columns

The `ORDER BY` clause was used to identify the number of columns returned by the original query.

**Payloads Tested:**

```sql
' ORDER BY 1-- -
```

```sql
' ORDER BY 2-- -
```

```sql
' ORDER BY 3-- -
```

### Observation

* `ORDER BY 1` executed successfully.
* `ORDER BY 2` executed successfully.
* `ORDER BY 3` generated an SQL error.

### Conclusion

The original SQL query returned **2 columns**.

---

## Why This Technique Works

The `ORDER BY` clause sorts query results using the specified column number.

For example:

```sql
ORDER BY 1
```

sorts by the first column.

If the query only contains two columns:

```sql
ORDER BY 3
```

references a column that does not exist, causing the database to generate an error.

This allows an attacker to determine the exact number of columns returned by the original query.

---

## Result

Successfully identified that the vulnerable query returned **2 columns**, satisfying the first requirement for a successful UNION-based SQL Injection attack.

---

## Impact

Determining the correct column count enables an attacker to:

* Construct valid UNION SELECT statements
* Retrieve arbitrary data from the database
* Enumerate database objects
* Extract sensitive information

This technique is a critical reconnaissance step during SQL Injection exploitation.

---

## Mitigation

* Use parameterized queries (prepared statements).
* Avoid constructing SQL statements using user input.
* Implement strict server-side input validation.
* Suppress detailed database error messages.
* Conduct regular application security testing.

---

## Key Concepts Learned

* UNION-Based SQL Injection
* Column Count Discovery
* ORDER BY Enumeration
* SQL Error Analysis
* Reconnaissance for Data Extraction

---

## Tools Used

* Burp Suite Repeater
* Browser Developer Tools

---

## Conclusion

This lab demonstrated how attackers determine the number of columns returned by a vulnerable SQL query using the `ORDER BY` clause. Identifying the correct column count is an essential prerequisite for successful UNION-based SQL Injection because both queries must return the same number of columns before data can be extracted.
