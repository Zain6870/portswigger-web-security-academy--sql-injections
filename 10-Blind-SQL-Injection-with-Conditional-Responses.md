# Blind SQL Injection with Conditional Responses

## Overview

This lab demonstrates how a Blind SQL Injection vulnerability can be exploited even when the application does not display database errors or query results. Instead of relying on visible output, the attacker observes changes in the application's response to determine whether an injected SQL condition evaluates to TRUE or FALSE.

## Objective

Determine the password of the administrator user by exploiting a Blind SQL Injection vulnerability using conditional responses, then log in as the administrator.

## Vulnerability

The application was vulnerable to SQL Injection through the `TrackingId` cookie.

**Vulnerable Parameter:**

```http
Cookie: TrackingId=<value>
```

The cookie value was incorporated directly into an SQL query without proper sanitization.

## Methodology

### Step 1: Verify Blind SQL Injection

A boolean condition that always evaluates to TRUE was injected into the TrackingId cookie.

Payload:

```sql
' AND '1'='1
```

### Observation

The application's response remained normal.

### Step 2: Test a FALSE Condition

The injected condition was modified so that it always evaluated to FALSE.

Payload:

```sql
' AND '1'='2
```

### Observation

The application's response changed, indicating that the SQL query returned different results.

### Conclusion

The application behaved differently depending on whether the injected condition evaluated to TRUE or FALSE, confirming the presence of a Blind SQL Injection vulnerability.

### Step 3: Confirm the Administrator User Exists

Payload:

```sql
' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

### Observation

The application returned the TRUE response.

### Conclusion

The administrator user exists within the database.

### Step 4: Determine the Password Length

The LENGTH() function was used to identify the number of characters in the administrator's password.

Example Payload:

```sql
' AND (SELECT LENGTH(password) FROM users WHERE username='administrator')>10
```

The comparison value was gradually increased until the exact password length was identified.

### Observation

The response changed once the tested value exceeded the actual password length.

### Conclusion

The administrator password length was successfully determined.

### Step 5: Extract the Password

The SUBSTRING() function was used to retrieve one character of the password at a time.

Example Payload:

```sql
' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a
```

Each character position was tested individually until the complete password was recovered.

### Step 6: Log In as Administrator

The extracted administrator password was used to authenticate to the application.

## Result

Successfully determined the administrator password through conditional responses and logged in as the administrator.

## Why It Worked

Although the application did not display SQL errors or query results, it behaved differently depending on whether the injected SQL condition evaluated to TRUE or FALSE. By carefully crafting boolean conditions and observing the application's responses, sensitive information could be inferred one character at a time.

## Impact

An attacker may be able to:

- Enumerate database contents
- Determine usernames and passwords
- Bypass authentication
- Retrieve sensitive information without visible database errors
- Compromise privileged accounts

## Mitigation

- Use parameterized queries (prepared statements)
- Avoid constructing SQL queries using user input
- Validate and sanitize user input
- Implement generic application responses
- Apply least-privilege database permissions
- Conduct regular security testing

## Key Concepts Learned

- Blind SQL Injection
- Boolean-Based SQL Injection
- Conditional Responses
- LENGTH() Function
- SUBSTRING() Function
- Character-by-Character Data Extraction

## Tools Used

- Burp Suite Repeater
- Browser Developer Tools

## Conclusion

This lab demonstrated how sensitive information can be extracted from a database even when the application does not reveal SQL errors or query results. By exploiting differences in the application's responses to TRUE and FALSE conditions, it was possible to infer the administrator's password one character at a time.
