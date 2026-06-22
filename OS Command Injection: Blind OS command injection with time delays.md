# Title: OS Command Injection: Blind OS command injection with time delays

## Description

The application is vulnerable to Blind OS Command Injection within the feedback submission functionality. User-supplied input is incorporated into a backend operating system command without proper validation or sanitization.

Although the output of the executed command is not returned in the HTTP response, an attacker can infer successful command execution through observable side effects such as response delays.

By injecting additional shell commands into the vulnerable parameter, an attacker can execute arbitrary operating system commands on the underlying server.

---

## Steps to Exploit

### 1. Identify the Injection Point

1. Navigate to the feedback page.
2. Complete and submit the feedback form.
3. Intercept the request using Burp Suite.
4. Locate the `email` parameter in the request body.

---

### 2. Inject a Time-Based Payload

Modify the value of the `email` parameter:

```http
email=x||ping+-c+10+127.0.0.1||
```

The payload uses shell command separators (`||`) to inject an additional operating system command.

The injected command:

```bash
ping -c 10 127.0.0.1
```

causes the server to perform ten ping requests to the local host, resulting in an approximate 10-second delay.

---

### 3. Observe the Response Delay

1. Forward the modified request.
2. Measure the response time.
3. Observe that the application takes approximately 10 seconds longer than normal to respond.

The delay confirms successful execution of the injected operating system command.

---

## Proof of Concept

### Payload

```http
email=x||ping+-c+10+127.0.0.1||
```

### Expected Behavior

**Normal Request**

```text
Response received immediately
```

**Injected Request**

```text
Response delayed by approximately 10 seconds
```

### Command Executed

```bash
ping -c 10 127.0.0.1
```

The response delay serves as evidence that the command was executed successfully on the server.

---

## Impact

* Arbitrary operating system command execution.
* Potential server compromise.
* Unauthorized access to sensitive files and configuration data.
* Ability to perform system enumeration and reconnaissance.
* Risk of privilege escalation depending on application permissions.
* Potential for remote code execution (RCE).
* Loss of confidentiality, integrity, and availability of the affected system.

---

## Mitigation / Remediation

### 1. Avoid Shell Command Execution

Do not construct operating system commands using user-supplied input.

Use safe APIs and library functions instead of invoking shell commands.

### 2. Implement Input Validation

Apply strict allow-list validation on all user-controlled input.

Reject special shell metacharacters such as:

```text
;
&
|
||
$
`
()
<>
```

### 3. Use Parameterized Command Execution

Where command execution is unavoidable, use APIs that separate command arguments from executable commands.

### 4. Apply Least Privilege

Run application processes with the minimum operating system permissions required.

### 5. Implement Security Monitoring

Monitor for abnormal command execution patterns and unusual response delays.

### 6. Conduct Security Testing

Perform regular penetration testing and secure code reviews to identify command injection vulnerabilities.

---

## CVSS Score

**CVSS v3.1 Score:** 9.8 (Critical)

**Vector:**

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
```

---

## CVSS Justification

| Metric              | Value     | Justification                                           |
| ------------------- | --------- | ------------------------------------------------------- |
| Attack Vector       | Network   | Exploitable remotely through web requests               |
| Attack Complexity   | Low       | Requires only crafted input in a form parameter         |
| Privileges Required | None      | No authentication required                              |
| User Interaction    | None      | No victim interaction required                          |
| Scope               | Unchanged | Impact remains within the vulnerable server environment |
| Confidentiality     | High      | Potential access to sensitive files and system data     |
| Integrity           | High      | Arbitrary command execution can modify system resources |
| Availability        | High      | Commands can disrupt services or exhaust resources      |

---

## Conclusion

The application is vulnerable to Blind OS Command Injection in the feedback functionality. Although command output is not directly returned to the attacker, successful exploitation can be confirmed through time-based side channels. This vulnerability enables arbitrary operating system command execution and may ultimately lead to complete server compromise if left unaddressed.
