# Blind SQL Injection with Time Delays

## Overview

This lab demonstrates how a Blind SQL Injection vulnerability can be exploited using time delays. Unlike previous SQL Injection techniques, the application neither displays database errors nor returns different responses based on injected conditions. Instead, the SQL query executes synchronously, allowing an attacker to infer whether injected SQL statements execute successfully by observing delays in the server's response time.

## Objective

Trigger a 10-second delay in the application's response by exploiting a Blind SQL Injection vulnerability.

## Vulnerability

The application was vulnerable to SQL Injection through the `TrackingId` cookie.

**Vulnerable Parameter:**

```http
Cookie: TrackingId=<value>
```

The cookie value was directly incorporated into an SQL query without proper sanitization.

## Methodology

### Step 1: Verify Blind SQL Injection

A payload was injected into the `TrackingId` cookie to invoke PostgreSQL's `pg_sleep()` function.

Payload:

```sql
x'||pg_sleep(10)--
```

### Observation

The application took approximately **10 seconds** to respond.

### Conclusion

The injected SQL statement was executed successfully, confirming the presence of a Blind SQL Injection vulnerability.

---

### Step 2: Verify the Delay

The request was sent multiple times to ensure the delay was consistent and not caused by network latency.

### Observation

Each request consistently delayed the response by approximately **10 seconds**.

### Conclusion

The delay was caused by execution of the injected SQL statement rather than external factors.

## Result

Successfully triggered a **10-second delay** in the application's response, confirming that arbitrary SQL statements could be executed through the vulnerable parameter.

## Why It Worked

The application executed the SQL query synchronously, meaning it waited for the database query to finish before returning a response. By injecting PostgreSQL's `pg_sleep(10)` function, the database intentionally paused execution for ten seconds. Although no data was returned and no errors were displayed, the response delay confirmed that the injected SQL statement had been executed.

## Impact

An attacker may be able to:

- Confirm the presence of Blind SQL Injection
- Execute arbitrary SQL statements
- Perform time-based database enumeration
- Extract sensitive information using conditional delays
- Use the vulnerability as a foundation for more advanced Blind SQL Injection attacks

## Mitigation

- Use parameterized queries (prepared statements)
- Avoid constructing SQL queries using user input
- Validate and sanitize all user input
- Apply the principle of least privilege
- Monitor for abnormal database execution times
- Conduct regular application security testing

## Key Concepts Learned

- Blind SQL Injection
- Time-Based SQL Injection
- PostgreSQL `pg_sleep()`
- Response Time Analysis
- SQL Query Execution
- Vulnerability Verification

## Tools Used

- Burp Suite Repeater
- Browser Developer Tools

## Conclusion

This lab demonstrated how response timing can be used to detect and confirm Blind SQL Injection vulnerabilities when neither query results nor database errors are visible. By exploiting PostgreSQL's `pg_sleep()` function, it was possible to intentionally delay database execution and verify that injected SQL statements were successfully executed.
