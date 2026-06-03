# SSRF with Blacklist-Based Input Filter

## 📌 Summary

The application is vulnerable to **Server-Side Request Forgery (SSRF)** via its stock check feature. The developers implemented a blacklist-based input filter to block dangerous keywords like `localhost`, `127.0.0.1`, and `admin`. However, because blacklist-based defenses are inherently weak, an attacker can bypass these restrictions using alternative IP representations (like short-hand IP addressing) and obfuscating blocked words with **double URL encoding**.

---

## 🧾 Description

Blacklist filtering looks for specific forbidden strings instead of validating that an input conforms to a strict, safe format. In this application, if you try to enter `http://127.0.0.1` or `http://localhost` into the `stockApi` parameter, the server rejects it with a "Missing parameter" or "Invalid URL" style block error.

We can bypass the host filter by exploiting how operating systems interpret IP addresses; using the short-hand decimal format `127.1` resolves locally exactly like `127.0.0.1`.

Furthermore, the server filters out the keyword `admin`. To defeat this second layer of defense, we can use **double URL encoding** on characters inside the blocked string (for example, encoding the character `a` to `%2561`). When the input initially passes through the front-end security filter, it only decodes it once (turning `%2561` into `%61`), which looks completely harmless. When it safely reaches the backend template or routing engine, it gets decoded a second time into the letter `a`, allowing us to successfully execute administrative path actions.

---

## 🔁 Steps to Reproduce

1. Navigate to a product page, click **"Check stock"**, and intercept the transaction using **Burp Suite**. Send this request over to **Burp Repeater**.
2. Change the target server path in the `stockApi` parameter to `http://127.0.0.1/` or `http://localhost/`. Send the request and observe that the server blocks it.
3. Compress the loopback IP structure by changing the parameter to use the shortened IP alias address:
```text
stockApi=http://127.1/

```


Send the request and notice that the initial IP blacklist rule is successfully bypassed (returning a 200 OK or showing a directory path context).
4. Now append the administration path directory block (`/admin`) to your working short-form URL link:
```text
stockApi=http://127.1/admin

```


Send the payload and notice that the security system triggers again, blocking access to the word "admin".
5. Obfuscate the character `a` inside the word "admin" using double URL encoding (`a` $\rightarrow$ `%61` $\rightarrow$ `%2561`):
```text
stockApi=http://127.1/%2561dmin

```


Send the request. The application strips away the outer layer, reads it as `%61dmin` (which passes the filter check), and then executes it locally as `/admin` to reveal the private **Admin Panel interface**.
6. Locate the specific query endpoint path string used to drop active user records, and use the same double-encoding trick to deliver the deletion parameter string for user `carlos`:
```text
stockApi=http://127.1/%2561dmin/delete?username=carlos

```


7. Send the finalized query request. The backend server maps out the target interface locally, completes the command sequence, and resolves the lab challenge successfully.

---

## 📸 Proof of Concept (PoC)

1. Reviewing the initial default stock API check parameter configuration
![Responses](../images/Lab4/vulnRequest.png?v=1)

2. Bypassing the local IP address restriction mechanism using short-hand notation `127.1`
![Responses](../images/Lab4/bypassing1.png?v=1)

3. Utilizing double URL encoding (`%2561dmin`) to fully evade the keyword detection engine
![Responses](../images/Lab4/bypassing2.png?v=1)

4. Forwarding the encoded payload string structure to target and delete the user Carlos
![Responses](../images/Lab4/deleting.png?v=1)


---

## 💥 Impact

* **Evasion of Security Filters** Reliance on blacklists provides a false sense of security, as attackers can easily utilize alternative encodings or network definitions to bypass filters.
* **Access to Private Administration Interfaces** Attackers can bypass perimeter access blocks to manipulate system operations, manage restricted accounts, or view hidden dashboards.
* **Complete Administrative Compromise** By gaining access to critical back-end functionality, attackers can modify databases, destroy sensitive business parameters, or execute unauthorized operations (like deleting user data).

---

## 🛠️ Remediation

To secure the application against filter evasion techniques:

* **Implement Strict Allow-lists (Whitelisting)** Completely scrap blacklist configurations. Instead, configure input validation that matches against a strict regular expression pattern containing only explicitly trusted host targets.
* **Normalize Inputs Before Validation** If validation steps must occur, ensure the web infrastructure completely decodes, standardizes, and flattens all incoming query inputs *before* passing them to any string-matching security rules.
* **Adopt Safe Indirect Routing Layouts** Avoid letting user parameters control target destination strings directly. Use localized, hard-coded integer indexes on the backend server that map to specific trusted target servers.