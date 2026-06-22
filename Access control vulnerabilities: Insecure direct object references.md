# Title: Access control vulnerabilities: Insecure direct object references

## Description

The application stores live chat transcripts as files on the server and exposes them through predictable, directly accessible URLs. Access to these resources is controlled solely by the filename provided in the URL, without verifying whether the requesting user is authorized to access the requested transcript.

Because transcript filenames follow a predictable numbering scheme, an attacker can modify the file identifier and access chat logs belonging to other users.

This vulnerability is an example of an **Insecure Direct Object Reference (IDOR)**, where direct access to internal objects is granted without proper authorization checks.

---

## Steps to Exploit

### 1. Access the Live Chat Feature

1. Navigate to the **Live Chat** functionality.
2. Send a message to generate a chat session.
3. Click **View Transcript**.

The application redirects to a transcript file.

Example:

```text id="vfx6r8"
/download-transcript/5.txt
```

---

### 2. Analyze the URL Structure

Observe that transcripts are stored as text files with sequential numeric identifiers.

Example:

```text id="2t8n1g"
1.txt
2.txt
3.txt
4.txt
5.txt
```

This suggests that transcript filenames are predictable.

---

### 3. Access Another User's Transcript

Modify the transcript identifier in the URL.

Example:

```text id="xrw7d6"
/download-transcript/1.txt
```

Request the file.

---

### 4. Retrieve Sensitive Information

The response contains another user's chat transcript.

Example:

```text id="l3k7bt"
User: carlos
Support Agent: Your password is ********
```

Sensitive credentials are disclosed within the transcript.

---

### 5. Authenticate as the Target User

Use the exposed credentials to log in.

Example:

```text id="6j4hy9"
Username: carlos
Password: <retrieved_password>
```

Successful authentication confirms account compromise.

---

## Proof of Concept

### Original Transcript URL

```text id="fqu2t4"
/download-transcript/5.txt
```

### Modified Transcript URL

```text id="67j2tx"
/download-transcript/1.txt
```

### Response

```text id="a74m3y"
Chat transcript belonging to another user
```

### Result

```text id="jlwmc4"
Unauthorized access to sensitive user information
```

The ability to access another user's transcript confirms the presence of an IDOR vulnerability.

---

## Impact

* Unauthorized access to sensitive user data.
* Disclosure of credentials and personal information.
* Account takeover opportunities.
* Violation of confidentiality requirements.
* Exposure of customer support communications.
* Potential compromise of multiple user accounts.

---

## Mitigation / Remediation

### 1. Implement Authorization Checks

Verify that users are authorized to access a resource before serving it.

Example:

```text id="h1kr0v"
if (transcript.owner == currentUser)
    allowAccess();
else
    denyAccess();
```

---

### 2. Avoid Predictable Object References

Replace sequential identifiers with unpredictable values.

Example:

```text id="gq2fx0"
7f8a1d1d-c4d5-43a2-b4fa-6b6d9f94a9c2
```

instead of:

```text id="ztwy1s"
1.txt
2.txt
3.txt
```

---

### 3. Store Sensitive Files Outside Web Root

Prevent direct URL access to sensitive documents whenever possible.

---

### 4. Apply Principle of Least Privilege

Ensure users can access only resources explicitly associated with their accounts.

---

### 5. Conduct Access Control Testing

Regularly test object references and resource identifiers for authorization bypass vulnerabilities.

---

## CVSS Score

**CVSS v3.1 Score:** 7.5 (High)

**Vector:**

```text id="t4x4pm"
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N
```

---

## CVSS Justification

| Metric              | Value     | Justification                                          |
| ------------------- | --------- | ------------------------------------------------------ |
| Attack Vector       | Network   | Exploitable remotely through URL manipulation          |
| Attack Complexity   | Low       | Requires only modification of a predictable identifier |
| Privileges Required | None      | No authentication required to access files             |
| User Interaction    | None      | No victim interaction required                         |
| Scope               | Unchanged | Impact remains within the application                  |
| Confidentiality     | High      | Sensitive user data and credentials disclosed          |
| Integrity           | None      | No modification of data observed                       |
| Availability        | None      | No service disruption observed                         |

---

## Conclusion

The application is vulnerable to an Insecure Direct Object Reference (IDOR) because chat transcripts are exposed through predictable URLs without authorization checks. Attackers can enumerate transcript identifiers and access sensitive information belonging to other users, potentially leading to credential disclosure and account compromise. Proper access control validation and the use of unpredictable resource identifiers are required to mitigate this vulnerability.
