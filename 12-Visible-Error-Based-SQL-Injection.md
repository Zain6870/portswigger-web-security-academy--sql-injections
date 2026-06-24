# Visible Error-Based SQL Injection

## Overview

This lab demonstrates how verbose database error messages can be exploited to retrieve sensitive information from a database. Instead of displaying query results directly, the application leaks database error messages to the user. By intentionally causing a type conversion error, sensitive data such as the administrator's password can be exposed within the error message.

## Objective

Retrieve the administrator's password by exploiting a visible error-based SQL Injection vulnerability and use it to log in as the administrator.

## Vulnerability

The application was vulnerable to SQL Injection through the `TrackingId` cookie.

**Vulnerable Parameter:**

```http
Cookie: TrackingId=<value>
```

The application incorporated the cookie value directly into an SQL query without proper sanitization and displayed verbose database error messages.

## Methodology

### Step 1: Verify SQL Injection

A single quote was appended to the TrackingId cookie.

Payload:

```sql
'
```

### Observation

The application returned a detailed SQL error indicating an unterminated string literal.

### Conclusion

The TrackingId cookie was vulnerable to SQL Injection.

---

### Step 2: Confirm Query Manipulation

The remaining portion of the SQL query was commented out.

Payload:

```sql
'--
```

### Observation

The application returned a normal response.

### Conclusion

The injected SQL statement was syntactically valid.

---

### Step 3: Verify Error-Based Injection

A CAST operation was performed using a valid integer.

Payload:

```sql
' AND 1=CAST((SELECT 1) AS int)--
```

### Observation

The request executed successfully without generating an error.

### Conclusion

The CAST function could be used to trigger controlled database errors.

---

### Step 4: Extract the Administrator Password

The CAST statement was modified to retrieve the administrator's password.

Payload:

```sql
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

### Observation

Since the password was a string and could not be converted into an integer, PostgreSQL generated an error similar to:

```text
ERROR: invalid input syntax for type integer: "mswi515la4qm3rftxxs0"
```

The administrator's password was disclosed within the database error message.

---

### Step 5: Log In as Administrator

The extracted administrator password was used to authenticate to the application.

## Result

Successfully extracted the administrator password through a visible database error message and logged in as the administrator.

## Why It Worked

The application exposed detailed database error messages to the client. By forcing PostgreSQL to convert a text value into an integer using the `CAST()` function, the conversion failed and the database included the original string value within the error message. Because the injected query retrieved the administrator's password, the password itself was leaked to the attacker.

## Impact

An attacker may be able to:

- Retrieve sensitive database information
- Extract usernames and passwords
- Enumerate database contents
- Compromise administrator accounts
- Gain unauthorized access to the application

## Mitigation

- Use parameterized queries (prepared statements)
- Disable verbose database error messages
- Validate and sanitize user input
- Apply least-privilege database permissions
- Implement secure exception handling
- Conduct regular security testing

## Key Concepts Learned

- Visible Error-Based SQL Injection
- CAST() Function
- PostgreSQL Error Messages
- Data Type Conversion
- Credential Disclosure
- Administrator Account Compromise

## Tools Used

- Burp Suite Repeater
- Browser Developer Tools

## Conclusion

This lab demonstrated how verbose database error messages can unintentionally disclose sensitive information. By exploiting a SQL Injection vulnerability and intentionally triggering a type conversion error with the `CAST()` function, it was possible to reveal the administrator's password directly within the application's error response. This highlights the importance of secure error handling alongside parameterized queries.
