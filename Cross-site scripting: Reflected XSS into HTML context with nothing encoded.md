# Title: Cross-site scripting: Reflected XSS into HTML context with nothing encoded

## Description

The application is vulnerable to Reflected Cross-Site Scripting (XSS) within the search functionality. User-supplied input from the search parameter is reflected directly into the HTML response without proper validation, sanitization, or output encoding.

Because the application inserts user-controlled content directly into the page's HTML context, an attacker can inject arbitrary JavaScript code that executes in the victim's browser.

Successful exploitation allows execution of malicious scripts within the security context of the vulnerable application.

---

## Steps to Exploit

### 1. Identify the Vulnerable Functionality

1. Navigate to the application's search feature.
2. Enter a search term.
3. Observe that the search value is reflected in the response page.

Example:

```text
Search for: laptop
```

Response:

```html
<h1>Search results for laptop</h1>
```

---

### 2. Inject Malicious JavaScript

Enter the following payload into the search box:

```html
<script>alert(1)</script>
```

---

### 3. Submit the Request

1. Click **Search**.
2. The application reflects the payload directly into the HTML response.
3. The browser interprets the injected script as executable JavaScript.

---

### 4. Observe Code Execution

A JavaScript alert dialog appears:

```javascript
alert(1)
```

This confirms successful execution of attacker-controlled JavaScript.

---

## Proof of Concept

### Payload

```html
<script>alert(1)</script>
```

### Vulnerable Response

```html
<h1>Search results for <script>alert(1)</script></h1>
```

### Result

```text
JavaScript executes in the victim's browser
```

The appearance of the alert dialog confirms the presence of a reflected XSS vulnerability.

---

## Impact

* Execution of arbitrary JavaScript within a victim's browser.
* Session hijacking through cookie theft (where cookies are accessible).
* Credential theft via phishing or keylogging attacks.
* Defacement of application content.
* Unauthorized actions performed on behalf of authenticated users.
* Delivery of malware or malicious redirects.
* Loss of confidentiality and integrity of user sessions.

---

## Mitigation / Remediation

### 1. Implement Output Encoding

Encode user-controlled data before rendering it within HTML pages.

Example:

```html
&lt;script&gt;alert(1)&lt;/script&gt;
```

---

### 2. Validate User Input

Apply strict input validation and reject unexpected characters where appropriate.

---

### 3. Use Context-Aware Escaping

Apply escaping mechanisms appropriate to the output context:

* HTML Context
* HTML Attribute Context
* JavaScript Context
* URL Context

---

### 4. Implement Content Security Policy (CSP)

Deploy a restrictive Content Security Policy to reduce the impact of XSS attacks.

Example:

```http
Content-Security-Policy: default-src 'self'
```

---

### 5. Use Security Frameworks

Leverage modern templating engines and frameworks that automatically encode output by default.

---

## CVSS Score

**CVSS v3.1 Score:** 6.1 (Medium)

**Vector:**

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N
```

---

## CVSS Justification

| Metric              | Value    | Justification                                               |
| ------------------- | -------- | ----------------------------------------------------------- |
| Attack Vector       | Network  | Exploitable remotely through a web request                  |
| Attack Complexity   | Low      | Simple payload injection                                    |
| Privileges Required | None     | No authentication required                                  |
| User Interaction    | Required | Victim must load the malicious URL or page                  |
| Scope               | Changed  | Impact extends from the application to the victim's browser |
| Confidentiality     | Low      | Potential exposure of session information                   |
| Integrity           | Low      | Attacker can manipulate page content and user actions       |
| Availability        | None     | No direct service disruption                                |

---

## Conclusion

The application is vulnerable to Reflected Cross-Site Scripting (XSS) because user-supplied search input is reflected directly into the HTML response without encoding. Successful exploitation allows attackers to execute arbitrary JavaScript in a victim's browser, potentially leading to session hijacking, credential theft, and unauthorized actions performed on behalf of users.
