# Lab: User Role Controlled by Request Parameter
**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Apprentice  
**Domain:** Access Control / Server-Side Vulnerabilities  

## 📝 Vulnerability Overview
The application relies on a client-side cookie (`Admin`) to determine user privileges. Because cookies are fully controlled by the user, an attacker can modify this parameter to elevate their privileges and access restricted areas of the site (Privilege Escalation).

## 🎯 Objective
Access the restricted administration panel at `/admin` and delete the user `carlos`.

## 🛠️ Exploitation Steps

### 1. Reconnaissance
* Navigated to `/admin` while unauthenticated; the server returned a `401 Unauthorized` or redirect response, indicating direct access is blocked.
* Logged into the application using the provided low-privilege credentials (`wiener:peter`).

### 2. Interception & Analysis
* Captured the login traffic using **Burp Suite Proxy**.
* Enabled **Response Interception** to inspect how the server establishes the user session.
* Observed the HTTP response setting a highly suspicious cookie:
  ```http
  Set-Cookie: Admin=false; Secure; HttpOnly
###  3. Manipulation (Privilege Escalation)
* Modified the cookie value in the intercepted response from Admin=false to Admin=true.
* Forwarded the response to the browser.
* With the modified cookie active in the browser session, navigated back to /admin. The administrative dashboard loaded successfully.
* Triggered the deletion action for the user carlos.
## 🛡️ Remediation & Prevention
The root cause of this flaw is relying on untrusted client-side inputs for access control decisions.
1. Server-Side Verification: User roles and permissions must be tracked securely on the server-side session object, not stored in modifiable client-side flags or cookies.
2. Cryptographic Integrity: If roles must be stored client-side (e.g., in a JWT), the token must be cryptographically signed by the server using a strong, secret key to prevent unauthorized tampering.
