# Exploiting Basic Server-Side Request Forgery (SSRF)

## 📌 Summary

The application is vulnerable to **Server-Side Request Forgery (SSRF)** because its stock check feature trusts user-supplied URLs to fetch data from internal systems. By modifying this target URL, an attacker can force the server to make unauthorized HTTP requests to its own internal loops (`localhost`), gaining access to administrative functionalities that are otherwise restricted.

---

## 🧾 Description

Server-Side Request Forgery (SSRF) happens when a web server fetches external data based on a URL provided by the user without proper validation. In this application, the "Check stock" feature passes a URL via the `stockApi` parameter to retrieve warehouse data.

Because the server blindly trusts this parameter, we can manipulate the request to point to the server's internal loopback address (`http://localhost/`). This bypasses standard front-end access controls, giving us access to the private `/admin` panel and allowing us to perform administrative actions like deleting user profiles.

---

## 🔁 Steps to Reproduce

1. Browse the website, open any product page, and locate the **"Check stock"** button.
2. Intercept the request using **Burp Suite** when clicking the button, and send it to **Burp Repeater**. Notice the vulnerable `stockApi` parameter in the body data.
```text

```



POST /product/stock HTTP/2
...
stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1

```

3. Change the value of the `stockApi` parameter to target the internal admin panel directly:
   ```text
stockApi=http://localhost/admin

```

4. Send the modified request in Repeater. Review the response (or use the Render tab) to view the restricted **Admin panel** and find the specific endpoint path used to delete users.
5. Update the `stockApi` parameter once more with the deletion endpoint to target the user `carlos`:
```text

```



stockApi=http://localhost/admin/delete?username=carlos

```

6. Send the request. The server processes the internal URL locally, executes the command, and deletes the target user account successfully.

---

## 📸 Proof of Concept (PoC)

1. Intercepting the default stock check request via Burp Suite
![Responses](../images/Lab1/vulnRequest.png)

2. Modifying the parameter to access the hidden local admin interface
![Responses](../images/Lab1/exploiting.png)

3. Submitting the payload to delete the user Carlos
![Responses](../images/Lab1/deleting.png)

4. Lab solved successfully
![Responses](../images/Lab1/solved.png)
---

## 💥 Impact

* **Access to Restricted Interfaces** Attackers can access private internal portals, administration panels, and configuration dashboards meant only for local server environments.
* **Internal Network Scanning** The vulnerable server can be used as a proxy to scan the internal network architecture, mapping open ports and identifying other vulnerable machines behind the firewall.
* **Unauthorized Data Manipulation** Privileged endpoints can be triggered to modify infrastructure, change application state, or delete critical user data (as demonstrated by deleting the user account).

---

## 🛠️ Remediation

To secure the application against SSRF vulnerabilities:

* **Implement Strict Allow-lists** Restrict the domain inputs accepted by the server to a pre-defined, trusted list of warehouse endpoints. Reject any inputs containing `localhost`, `127.0.0.1`, or unexpected internal host schemes.
* **Disable URL Parsing Input** Where possible, avoid taking complete raw URLs from user input. Use an internal identifier or index keys mapping safely to backend destination configurations on the server side instead.
* **Network-Level Segregation** Enforce firewall policies or network access control lists (ACLs) to ensure that public-facing web servers cannot communicate back with internal management endpoints or administrative protocols.

```