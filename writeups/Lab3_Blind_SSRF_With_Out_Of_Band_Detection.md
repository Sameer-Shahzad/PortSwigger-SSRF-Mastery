# Blind SSRF with Out-of-Band (OOB) Detection

## 📌 Summary

The application is vulnerable to **Blind Server-Side Request Forgery (SSRF)** via its analytics software functionality. The application tracks analytics by blindly fetching whatever URL is provided inside the incoming HTTP `Referer` header. Because the backend system processes these requests asynchronously and does not print any raw server responses back to the front-end interface, an attacker can use **Out-of-Band (OOB)** techniques via Burp Collaborator to verify the vulnerability and catch the outbound ping back.

---

## 🧾 Description

Blind SSRF occurs when an application can be forced to issue an HTTP request to an arbitrary URL, but the response from the target server is never returned to the attacker's browser. In this lab, the flaw lies in the backend analytics system that monitors visitor traffic patterns. When a product page loads, the analytics script extracts the domain listed in the `Referer` header and attempts to resolve and establish an HTTP connection with it.

Since we cannot see the backend server's direct response text, we rely on an Out-of-Band (OOB) approach. By swapping out the legitimate source referrer with a unique **Burp Collaborator** sub-domain, we can successfully monitor external network interactions (DNS resolutions and HTTP callbacks) triggered directly from the target infrastructure.

---

## 🔁 Steps to Reproduce

1. Click on any product item on the shopping homepage and intercept the traffic using **Burp Suite**.
2. Send the captured product view request (`GET /product?productId=X`) to **Burp Repeater**.
3. Locate the `Referer` header field within the HTTP request payload layout.
4. Open the **Burp Collaborator** client interface and generate a unique public interaction payload domain string.
5. Replace the target domain address string within the `Referer` header using your newly generated Collaborator URL payload:
```text
Referer: https://your-unique-id.oastify.com

```


6. Click the **Send** button inside Burp Repeater to dispatch the modified request structure to the target server engine.
7. Switch back over to your active **Collaborator** panel utility and select **"Poll now"** to pull down the asynchronous communication history log files.
8. Observe the inbound tracking logs. You will see active **DNS query resolutions** followed quickly by formal **HTTP interactions** hitting your external listener client, confirming a successful backend blind SSRF compromise.

---

## 📸 Proof of Concept (PoC)

1. Intercepting the product page request to locate the target analytical Referer parameter header
![Responses](../images/Lab1/solved.png)

2. Lab solved successfully after verifying the external server interaction callback
![Responses](../images/Lab1/solved.png)

---

## 💥 Impact

* **Internal Scanning and Exploration** Even without seeing direct data streams, an attacker can use timing behaviors or network configurations to probe and identify up or down status markers on hidden internal nodes.
* **Out-of-Band Exfiltration and Reconnaissance** Blind callouts reveal infrastructure details (such as the server's external-facing IP addresses and local software user-agent signatures).
* **Distributed Denial of Service (DDoS)** A vulnerable system can be used as a proxy botnet to flood external websites or third-party web services with unwanted HTTP request traffic.

---

## 🛠️ Remediation

To secure the application against analytics-driven Blind SSRF:

* **Sanitize and Validate Header Metadata** Do not configure internal service engines or analytical tracking modules to communicate with untrusted URLs derived directly from standard client request headers like `Referer`.
* **Implement Domain Restrictive Allow-lists** If handling the referrer domain address is absolutely necessary for internal link validation, map inputs strictly against an explicit allow-list of internal domains.
* **Restrict Outbound Network Rules** Configure strict outbound network firewall filters to completely prevent public application hosts from executing unexpected out-of-band communication profiles to unverified external internet blocks.