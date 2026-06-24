# Blind SQL Injection with Time Delays and Information Retrieval

## Overview

This lab demonstrates how a Blind SQL Injection vulnerability can be exploited to retrieve sensitive information using conditional time delays. Unlike previous techniques, the application does not display query results or database errors, and its responses remain identical regardless of whether an injected condition is TRUE or FALSE. Instead, information is inferred by observing intentional delays in the application's response time.

## Objective

Determine the administrator's password by exploiting a Blind SQL Injection vulnerability using conditional time delays and log in as the administrator.

## Vulnerability

The application was vulnerable to SQL Injection through the `TrackingId` cookie.

**Vulnerable Parameter:**

```http
Cookie: TrackingId=<value>
```

The cookie value was directly incorporated into an SQL query without proper sanitization.

## Methodology

### Step 1: Verify Time-Based SQL Injection

A payload using PostgreSQL's `pg_sleep()` function was injected into the `TrackingId` cookie.

Payload:

```sql
x'||pg_sleep(10)--
```

### Observation

The application's response was delayed by approximately **10 seconds**.

### Conclusion

The injected SQL statement executed successfully, confirming the presence of a Blind SQL Injection vulnerability.

---

### Step 2: Verify Conditional Execution

A `CASE` statement was used to execute the delay only when a specified condition evaluated to TRUE.

Payload:

```sql
x'||(SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END)--
```

### Observation

The application delayed its response by approximately **10 seconds**.

The condition was then modified:

```sql
x'||(SELECT CASE WHEN (1=2) THEN pg_sleep(10) ELSE pg_sleep(0) END)--
```

### Observation

The application responded immediately.

### Conclusion

Response timing depended on whether the injected SQL condition evaluated to TRUE or FALSE.

---

### Step 3: Determine the Password Length

The `LENGTH()` function was used to identify the number of characters in the administrator's password.

Example Payload:

```sql
x'||(SELECT CASE WHEN (username='administrator' AND LENGTH(password)>20) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users)--
```

The comparison value was gradually adjusted until the exact password length was determined.

### Observation

The administrator's password was found to be **20 characters** long.

### Conclusion

The password length was successfully identified using conditional time delays.

---

### Step 4: Extract the Password

The `SUBSTRING()` function was used to test each character of the administrator's password individually.

Example Payload:

```sql
x'||(SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users)--
```

Burp Suite Intruder was configured to iterate through the characters:

```text
a-z
0-9
```

Only the payload that caused a **10-second delay** represented the correct character.

This process was repeated for each character position until the complete password was recovered.

---

### Step 5: Log In as Administrator

The extracted administrator password was used to authenticate successfully as the administrator.

## Result

Successfully extracted the administrator's password using conditional time delays and authenticated as the administrator.

## Why It Worked

Although the application did not reveal query results or database errors, it executed SQL statements synchronously. By using PostgreSQL's `CASE` expression together with `pg_sleep()`, it was possible to create measurable differences in response times based on TRUE or FALSE conditions. Repeating this process for the password length and each character allowed the administrator's password to be reconstructed one character at a time.

## Impact

An attacker may be able to:

- Enumerate database contents
- Determine usernames and passwords
- Extract sensitive information without visible output
- Compromise administrator accounts
- Gain unauthorized access to protected resources

## Mitigation

- Use parameterized queries (prepared statements)
- Avoid constructing SQL queries using user input
- Validate and sanitize all user input
- Apply the principle of least privilege
- Monitor abnormal database execution times
- Conduct regular security testing

## Key Concepts Learned

- Blind SQL Injection
- Time-Based SQL Injection
- Conditional Time Delays
- PostgreSQL `pg_sleep()`
- CASE Expressions
- LENGTH() Function
- SUBSTRING() Function
- Character-by-Character Data Extraction
- Burp Suite Intruder

## Tools Used

- Burp Suite Repeater
- Burp Suite Intruder
- Browser Developer Tools

## Conclusion

This lab demonstrated how sensitive information can be extracted from a vulnerable application even when database errors and query results are completely hidden. By exploiting conditional time delays and automating requests with Burp Suite Intruder, it was possible to reconstruct the administrator's password one character at a time. This technique highlights the effectiveness of time-based Blind SQL Injection attacks against applications that rely solely on suppressing error messages instead of properly preventing SQL Injection.
