# Title: Business logic vulnerabilities: Excessive trust in client-side controls

## Description

The application contains a business logic vulnerability in the purchasing workflow. Product pricing is partially enforced on the client side, and the server trusts price values supplied by the client during cart operations.

When a product is added to the shopping cart, the request includes a user-controlled `price` parameter. The server fails to validate that this value matches the legitimate product price stored on the backend.

As a result, an attacker can manipulate the price parameter and purchase products at arbitrary prices, bypassing intended business rules and causing financial loss.

---

## Steps to Exploit

### 1. Identify the Purchase Workflow

1. Log in using the provided credentials:

```text id="7b3r2p"
Username: wiener
Password: peter
```

2. Browse the product catalog.
3. Attempt to purchase the **Lightweight l33t leather jacket**.
4. Observe that the purchase fails due to insufficient store credit.

---

### 2. Intercept Cart Requests

1. Add the jacket to the shopping cart.
2. Intercept the request using Burp Suite.
3. Locate the request responsible for adding items to the cart.

Example:

```http id="j6x8c5"
POST /cart HTTP/1.1

productId=1
price=133700
quantity=1
```

Notice that the product price is supplied by the client.

---

### 3. Manipulate the Price

Send the request to Burp Repeater.

Modify the price parameter:

```http id="3g2z4r"
POST /cart HTTP/1.1

productId=1
price=1
quantity=1
```

Forward the request.

---

### 4. Verify Price Manipulation

Refresh the shopping cart.

Observe that the application now displays the modified price rather than the legitimate product value.

Example:

```text id="k8f1q7"
Lightweight l33t leather jacket
Price: $1
```

This confirms that the server trusts client-supplied pricing information.

---

### 5. Complete the Purchase

1. Set the price to a value lower than the available store credit.
2. Proceed through checkout.
3. Complete the order successfully.

The purchase succeeds despite the manipulated price.

---

## Proof of Concept

### Original Request

```http id="z5m4p9"
POST /cart

productId=1
price=133700
quantity=1
```

### Modified Request

```http id="v2w7k1"
POST /cart

productId=1
price=1
quantity=1
```

### Result

```text id="n4r9e6"
Order completed successfully
```

The successful purchase demonstrates that client-side pricing controls can be bypassed.

---

## Impact

* Unauthorized price manipulation.
* Financial loss to the business.
* Circumvention of intended purchasing restrictions.
* Fraudulent purchases at arbitrary prices.
* Loss of integrity within the ordering system.
* Potential abuse at scale through automated requests.

---

## Mitigation / Remediation

### 1. Enforce Pricing Server-Side

The server must determine product pricing using trusted backend data.

Example:

```text id="f6t2m8"
price = database.getProductPrice(productId)
```

Never trust prices supplied by clients.

---

### 2. Ignore Client-Supplied Price Parameters

Remove pricing information from requests entirely where possible.

Example:

```http id="m3d8s5"
POST /cart

productId=1
quantity=1
```

The server should calculate pricing internally.

---

### 3. Validate Order Totals

Recalculate prices, discounts, taxes, and totals on the server before checkout.

---

### 4. Implement Integrity Checks

Verify that cart contents and pricing values have not been tampered with before processing payment.

---

### 5. Monitor Transaction Anomalies

Detect suspicious purchases involving:

* Extremely low prices
* Negative values
* Unusual discounts

---

### 6. Conduct Business Logic Security Testing

Regularly assess workflows for client-side trust issues and logic flaws.

---

## CVSS Score

**CVSS v3.1 Score:** 6.5 (Medium)

**Vector:**

```text id="x5m7v2"
CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:H/A:N
```

---

## CVSS Justification

| Metric              | Value     | Justification                                         |
| ------------------- | --------- | ----------------------------------------------------- |
| Attack Vector       | Network   | Exploitable remotely through web requests             |
| Attack Complexity   | Low       | Requires only modification of a request parameter     |
| Privileges Required | Low       | Requires a standard authenticated account             |
| User Interaction    | None      | No victim interaction required                        |
| Scope               | Unchanged | Impact remains within the application                 |
| Confidentiality     | None      | No information disclosure occurs                      |
| Integrity           | High      | Product pricing and business rules can be manipulated |
| Availability        | None      | No direct service disruption observed                 |

---

## Conclusion

The application is vulnerable to a business logic flaw because it relies on client-supplied pricing information during the purchasing workflow. Attackers can manipulate product prices and complete purchases at unauthorized values, resulting in financial loss and compromised transaction integrity. All pricing decisions should be enforced and validated exclusively on the server side.
