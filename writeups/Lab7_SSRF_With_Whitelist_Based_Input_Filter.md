# SSRF with Whitelist-Based Input Filter

## 📌 Summary

The application is vulnerable to **Server-Side Request Forgery (SSRF)** despite implementing a strict validation white-list filter. The protective input rule ensures that the parsed hostname matches a trusted domain name (`stock.weliketoshop.net`). However, because of discrepancies between how the front-end validation filter parses custom URL structures versus how the back-end routing client treats them, an attacker can use embedded credentials with a **double URL-encoded fragment identifier (`#`)** to spoof the destination and force local routing.

---

## 🧾 Description

A whitelist-based defense checks input against an explicit list of allowed safe values. In this scenario, the application expects the `stockApi` parameter to target the official domain string host.

However, standard URL specification structure allows embedding user account authentication details directly into the destination handle before the main server domain name like so: `http://username@expected-host`.

We can exploit this structure by passing `localhost` as the username component, followed by a fragment hashtag descriptor (`#`). The front-end filter extracts the trailing component and thinks the destination domain matches the whitelist rule perfectly.

But when sending the input parameter onward, we must **double-URL encode** the fragment tag to `%2523`. This ensures the initial request processing routine skips over it, leaving the secondary backend HTTP client to process the raw `%23` character down the line. The internal tool reads the string up to the decoded hash marker (`#`), treats everything after it as a comment segment, and routes the request execution strictly to the internal `localhost` root context instead.

---

## 🔁 Steps to Reproduce

1. Select an inventory product, hit the **"Check stock"** option, and route the traffic request using **Burp Suite** into **Burp Repeater**.
2. Tamper with the raw destination string inside the `stockApi` variable parameter fields. If you attempt an address switch like `http://127.0.0.1/`, notice that the security framework rejects it instantly due to strict hostname validation.
3. Check if the validation logic accepts embedded user credential components by appending an explicit host credential prefix block before the official target address:
```text
stockApi=http://username@stock.weliketoshop.net/

```


Send the payload. The application successfully returns a 200 OK connection sequence, showing it supports inline credential routing parameters.
4. Append a standard hash identifier (`#`) straight to the end of the user string parameter to see if we can cut off the remaining address field:
```text
stockApi=http://username#@stock.weliketoshop.net/

```


Notice that the request fails. The browser/application filter decodes the raw `#` instantly, causing parsing calculations to break or mismatch the target validation rules.
5. Now, bypass the parsing checks by **double URL-encoding** the hash character (`#` $\rightarrow$ `%23` $\rightarrow$ `%2523`). Inject the internal loopback IP address or `localhost` within the user zone segment along with the target port assignment details:
```text
stockApi=http://localhost:80%2523@stock.weliketoshop.net/

```


Send this request. The outer handler reads the entire string as a safe route, while the internal fetching software processes the backend string, drops the true host sequence, and yields access to the internal folder map directory structure.
6. Modify the path endpoints inside the query statement string layout to target the privileged user management system and execute account removal instructions for user `carlos`:
```text
stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos

```


7. Click the **Send** option command inside the editor workbench panel view. The underlying API tool handles the localized command string, targets the user block, and solves the expert challenge.

---

## 📸 Proof of Concept (PoC)

1. Reviewing the initial request parameter payload format to evaluate the target filter behaviors
![Responses](../images/Lab1/solved.png)

2. Attempting to inject embedded credentials to test the underlying URL parser engine structure
![Responses](../images/Lab1/solved.png)

3. Deploying double-encoded fragment syntax (`%2523`) to split the host rendering context
![Responses](../images/Lab1/solved.png)

4. Finalizing the internal command path string structure to wipe out Carlos from the admin profile logs
![Responses](../images/Lab1/solved.png)

---

## 💥 Impact

* **Complete Bypass of Solid Defense Rules** Whitelisting architectures can be completely circumvented if there are variations in how different code libraries parse complex URL strings.
* **Access to Restricted Admin Interfaces** Attackers gain unauthenticated administrative control channels, revealing hidden backend server consoles.
* **Privileged Execution** Attackers can delete profiles, modify configurations, or compromise data records across isolated back-end systems.

---

## 🛠️ Remediation

To secure the application against whitelist-evading parsing tactics:

* **Strictly Avoid Arbitrary URL Routing Parsers** Avoid letting the application accept raw, complete URL string layouts from an untrusted public user interface.
* **Enforce Strict Component Isolation and Extraction** Do not pass full user strings directly to backend HTTP fetching modules. Instead, parse out the target destination address explicitly using a single, unified parsing library *before* performing validation operations.
* **Implement Strict Character Filtering Blocks** Ensure that query handlers automatically isolate, block, or drop any incoming transmission scripts containing structural control parameters such as `@`, `:`, or `#` inside input fields meant solely for domain pathways.