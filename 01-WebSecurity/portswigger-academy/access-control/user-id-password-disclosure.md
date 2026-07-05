# Lab: User ID Controlled by Request Parameter with Password Disclosure
**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Apprentice  
**Domain:** Access Control (Insecure Direct Object Reference - IDOR) & Information Disclosure  

## 📝 Vulnerability Overview
The application exhibits an Insecure Direct Object Reference (IDOR) flaw on the user account page by relying entirely on a client-supplied parameter (`id`) to determine which profile to render. Furthermore, the application features an Information Disclosure vulnerability by pre-filling and rendering the user's plain-text password within the HTML source of the masked input field. Combined, these flaws allow an unprivileged attacker to extract secrets from any account, including the administrator.

## 🎯 Objective
Exploit the IDOR flaw to access the administrator's account page, retrieve their plain-text password, log in with administrative privileges, and delete the user `carlos`.

## 🛠️ Exploitation Steps

### 1. Analysis of Baseline Behavior
* Logged into the application using the low-privilege testing account credentials (`wiener:peter`).
* Navigated to the "My Account" page and observed the URL structure:
  ```text
  /my-account?id=wiener
* Inspected the page source and noticed that the account details section included a password input field. While masked in the UI (type="password"), the underlying HTML source code disclosed the plain-text password within the value attribute: HTML  <input type="password" name="password" value="peter">
    
### 2. Parameter Manipulation (IDOR)
* Modified the id parameter directly in the browser address bar (or intercepted the request via Burp Suite Proxy) to target the administrative account: Plaintext  /my-account?id=administrator    
* The server accepted the input without enforcing proper session-to-object authentication checks, rendering the administrative dashboard layout.
### 3. Password Exfiltration
* Right-clicked the page and selected View Page Source (or reviewed the raw HTTP response in Burp Suite).
* Located the masked password input component within the DOM tree and successfully extracted the administrator's plain-text password from the value attribute: Plaintext  dj5f596gjay02yeu8w3b
   
### 4. Privilege Escalation & Target Execution
* Logged out of the wiener session and authenticated into the application using the username administrator and the newly exfiltrated password: dj5f596gjay02yeu8w3b.
* Navigated to the administrative command center (/admin) and successfully deleted the user carlos to solve the lab.
## 🛡️ Remediation & Prevention
* Enforce Strict Access Control Matrices: The backend server must never trust client-controlled parameters implicitly. Implement server-side validation to ensure that the user identity linked to the current active session matches the data requested in the object parameter (request.id === session.username).
* Mask / Redact Sensitive Secrets: Plain-text passwords should never be passed back to the client interface or embedded within HTML source attributes, even if visually masked. If the input field requires formatting, it should be left empty or populated with a dummy placeholder string.
