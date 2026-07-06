# Lab: Basic SSRF Against the Local Server
**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Apprentice  
**Domain:** Server-Side Request Forgery (SSRF)  

## 📝 Vulnerability Overview
The application features a stock check component that accepts user-supplied URLs via a backend API parameter (`stockApi`) to query an inventory system. Because the backend server fails to validate or restrict the destination of this input, an attacker can manipulate the parameter to launch a Server-Side Request Forgery (SSRF) attack. This allows an external user to coerce the trusted server into making HTTP requests to internal-only infrastructure (such as the loopback address `http://localhost`) that is typically protected by network firewalls.

## 🎯 Objective
Exploit the stock check endpoint to access a localized administrative panel at `http://localhost/admin` and delete the user `carlos`.

## 🛠️ Exploitation Steps

### 1. Reconnaissance & Baseline Mapping
* Attempted to navigate directly to `/admin` via the browser. The server rejected the request with a standard access denial message, confirming that external perimeter routing blocks direct access to the administrative panel.
* Navigated to a product catalog item and triggered the "Check stock" feature.
* Intercepted the traffic in **Burp Suite Proxy** and analyzed the raw payload structure. The application handles stock checks via a `POST` request with a URL-encoded tracking parameter:
  ```http
  POST /product/stock HTTP/1.1
  ...
  stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
### 2. Internal Network Probing (SSRF)
Sent the captured request to Burp Suite Repeater.

Replaced the external stock API endpoint value with the internal loopback target:

HTTP
stockApi=http://localhost/admin
Dispatched the request. The application server processed the query internally, fetched the administrative page from its local interface, and reflected the full HTML structure of the admin console back in the HTTP response body.

### 3. Payload Delivery & Target Execution
Reviewed the returned administrative HTML code to isolate the user deletion endpoint routing. Located the following target URL layout:

Plaintext
http://localhost/admin/delete?username=carlos
Injected this destructive administrative URL directly back into the vulnerable parameter:

HTTP
stockApi=http://localhost/admin/delete?username=carlos
Dispatched the request through Repeater. The server successfully processed the administrative routing on behalf of the attacker, deleted the account for carlos, and resolved the challenge.

## 🛡️ Remediation & Prevention
Implement Network-Level Allowlists (Segmentation): Restrict the internal application layer from routing traffic to localized loopback endpoints (127.0.0.1 / localhost) or internal private IP space (RFC 1918) unless strictly necessary.

Input Validation & Domain Enforcement: Do not accept absolute, arbitrary URLs from client input. If the feature must call external resources, enforce a strict server-side allowlist containing only explicit, pre-approved external hostnames or relative routing paths.

Authentication on Internal Endpoints: Ensure that even localized internal interfaces (like /admin) require strong, explicit session authentication and authorization tokens rather than implicitly trusting requests simply because they originate from the local server (127.0.0.1).
