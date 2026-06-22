# Title: Cross-site scripting: Stored XSS into HTML context with nothing encoded

## Description

The application is vulnerable to Stored Cross-Site Scripting (XSS) within the blog comment functionality. User-supplied comment content is stored by the application and later rendered to other users without proper sanitization or output encoding.

Because the submitted comment is permanently stored and displayed whenever the blog post is viewed, malicious JavaScript executes automatically in the browsers of all users who access the affected page.

Unlike reflected XSS, stored XSS does not require victims to click a specially crafted link. The malicious payload is persisted on the server and delivered directly to visitors.

---

## Steps to Exploit

### 1. Identify the Vulnerable Functionality

1. Navigate to any blog post.
2. Scroll to the comment submission form.
3. Observe that user comments are displayed publicly after submission.

---

### 2. Submit a Malicious Comment

Enter the following payload into the comment field:

```html id="z6ll2v"
<script>alert(1)</script>
```

Complete the remaining required fields:

```text id="zwct2q"
Name: Test User
Email: test@example.com
Website: https://example.com
```

Click **Post Comment**.

---

### 3. Trigger the Payload

1. Return to the blog post page.
2. View the submitted comment.

The application renders the stored comment directly into the page without encoding.

---

### 4. Observe Code Execution

The browser executes the injected JavaScript:

```javascript id="1v0vrg"
alert(1)
```

An alert dialog appears, confirming successful XSS execution.

---

## Proof of Concept

### Malicious Comment

```html id="mfsyyd"
<script>alert(1)</script>
```

### Stored Data

```html id="sjis6p"
<script>alert(1)</script>
```

### Rendered Response

```html id="cd06hh"
<div class="comment">
    <script>alert(1)</script>
</div>
```

### Result

```text id="t8g5eo"
JavaScript executes whenever the blog post is viewed
```

This confirms that the application stores and executes attacker-controlled scripts.

---

## Impact

* Persistent execution of arbitrary JavaScript.
* Session hijacking through theft of accessible session tokens.
* Credential harvesting and phishing attacks.
* Unauthorized actions performed on behalf of authenticated users.
* Defacement of application content.
* Malware distribution and malicious redirects.
* Large-scale impact affecting all visitors to the affected page.

---

## Mitigation / Remediation

### 1. Implement Output Encoding

Encode all user-generated content before rendering it within HTML pages.

Example:

```html id="jmn6gq"
&lt;script&gt;alert(1)&lt;/script&gt;
```

---

### 2. Sanitize User Input

Remove or neutralize dangerous HTML tags and JavaScript content before storing user submissions.

---

### 3. Use Context-Aware Escaping

Apply output encoding based on the rendering context:

* HTML Context
* HTML Attribute Context
* JavaScript Context
* URL Context

---

### 4. Implement Content Security Policy (CSP)

Deploy a restrictive CSP to reduce the impact of successful XSS attacks.

Example:

```http id="sccwbo"
Content-Security-Policy: default-src 'self'
```

---

### 5. Validate User-Generated Content

Where rich text functionality is required, use secure HTML sanitization libraries.

---

### 6. Conduct Security Testing

Perform regular penetration testing and code reviews to identify persistent XSS vulnerabilities.

---

## CVSS Score

**CVSS v3.1 Score:** 8.8 (High)

**Vector:**

```text id="j1j6c0"
CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:H/A:N
```

---

## CVSS Justification

| Metric              | Value    | Justification                                                        |
| ------------------- | -------- | -------------------------------------------------------------------- |
| Attack Vector       | Network  | Exploitable remotely through web requests                            |
| Attack Complexity   | Low      | Requires only submission of a malicious comment                      |
| Privileges Required | None     | No authentication required                                           |
| User Interaction    | Required | Victims must view the affected page                                  |
| Scope               | Changed  | Impact extends to other users' browsers                              |
| Confidentiality     | High     | Session information and user data may be exposed                     |
| Integrity           | High     | Attackers can manipulate page content and perform actions as victims |
| Availability        | None     | No direct service disruption observed                                |

---

## Conclusion

The application is vulnerable to Stored Cross-Site Scripting (XSS) because user-supplied comments are stored and rendered without proper sanitization or output encoding. Successful exploitation allows attackers to execute arbitrary JavaScript in the browsers of users viewing the affected page, potentially leading to session hijacking, credential theft, and unauthorized actions performed on behalf of victims.
