# Title: Authentication: 2FA simple bypass

## Description

The application implements Two-Factor Authentication (2FA) as an additional security layer during the login process. However, access controls are improperly enforced after successful primary authentication.

Although users are prompted to provide a second authentication factor, the application grants access to protected resources before verifying completion of the 2FA workflow. An attacker possessing valid username and password credentials can bypass the verification step entirely by directly requesting authenticated resources.

This flaw effectively renders the second authentication factor ineffective and allows unauthorized access to protected user accounts.

---

## Steps to Exploit

### 1. Establish Normal Authentication Flow

1. Log in using a valid account:

```text id="hkhz3r"
Username: wiener
Password: peter
```

2. Observe that the application requests a 2FA verification code.
3. Complete the login process.
4. Navigate to the account page.

Example:

```http id="s7jtrn"
GET /my-account
```

5. Note the URL of the authenticated account page.

---

### 2. Authenticate as the Target User

Log out and log in using the target account credentials:

```text id="q1zg18"
Username: carlos
Password: montoya
```

After submitting valid credentials, the application redirects to the 2FA verification page.

---

### 3. Bypass the 2FA Verification Step

Without providing a valid verification code, manually navigate to:

```http id="tgmq1s"
GET /my-account
```

or directly modify the URL in the browser address bar:

```text id="5p14za"
/my-account
```

---

### 4. Observe Unauthorized Access

The application grants access to the protected account page despite the absence of successful 2FA verification.

This confirms that authorization checks rely solely on primary authentication and do not verify completion of the second authentication factor.

---

## Proof of Concept

### Valid Login

```text id="tahmft"
Username: carlos
Password: montoya
```

### Application Response

```text id="hj10l8"
Enter your 2FA verification code
```

### Bypass Request

```http id="0fo2kw"
GET /my-account HTTP/1.1
```

### Result

```text id="2s9z5q"
Carlos's account page loads successfully
```

The response confirms that protected resources can be accessed without completing the second authentication step.

---

## Impact

* Complete bypass of Two-Factor Authentication.
* Unauthorized access to protected user accounts.
* Increased risk of account takeover.
* Defeats the primary security benefit of 2FA.
* Exposure of sensitive account information.
* Potential abuse of account functionality and privileges.

---

## Mitigation / Remediation

### 1. Enforce 2FA at the Authorization Layer

Access to protected resources should require verification that the entire authentication process has been completed successfully.

Example logic:

```text id="m3mn4z"
if (authenticated && secondFactorVerified) {
    grantAccess();
}
```

---

### 2. Maintain Authentication State Properly

Store and validate a dedicated flag indicating successful completion of 2FA.

Example:

```text id="f0l2zn"
twoFactorVerified = true
```

Only grant access when this state is present.

---

### 3. Protect Sensitive Endpoints

Ensure that all authenticated resources verify both:

* Primary authentication status
* Second-factor verification status

---

### 4. Prevent Direct Object Access

Restrict access to authenticated pages until the complete login workflow is finished.

---

### 5. Perform Security Testing

Regularly assess authentication flows for logic flaws and authorization bypass vulnerabilities.

---

## CVSS Score

**CVSS v3.1 Score:** 8.8 (High)

**Vector:**

```text id="c50g6l"
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N
```

---

## CVSS Justification

| Metric              | Value     | Justification                                                                          |
| ------------------- | --------- | -------------------------------------------------------------------------------------- |
| Attack Vector       | Network   | Exploitable remotely through web requests                                              |
| Attack Complexity   | Low       | Requires only valid credentials and URL manipulation                                   |
| Privileges Required | None      | Attacker only requires knowledge of credentials; no prior authenticated session needed |
| User Interaction    | None      | No victim interaction required                                                         |
| Scope               | Unchanged | Impact remains within the application boundary                                         |
| Confidentiality     | High      | Unauthorized access to user account information                                        |
| Integrity           | High      | Attacker may perform actions as the compromised user                                   |
| Availability        | None      | No direct impact on service availability                                               |

---

## Conclusion

The application's Two-Factor Authentication implementation is ineffective because access controls do not verify successful completion of the second authentication step. An attacker possessing valid credentials can bypass 2FA simply by requesting protected resources directly, resulting in unauthorized account access and significantly weakening the application's authentication security model.
