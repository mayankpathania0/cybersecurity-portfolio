# Lab: Username Enumeration via Different Responses
**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Apprentice  
**Domain:** Authentication (Username Enumeration & Brute-Force)  

## 📝 Vulnerability Overview
The application's login mechanism suffers from information disclosure via verbose error messages. When a login attempt fails, the server explicitly differentiates between a non-existent user account ("Invalid username") and an incorrect password for a valid account ("Incorrect password"). This discrepancy allows an attacker to enumerate valid usernames across the system and subsequently launch a targeted brute-force attack to compromise accounts.

## 🎯 Objective
Enumerate a valid username from a provided list, brute-force that specific user's password, and successfully authenticate to access their account page.

## 🛠️ Exploitation Steps

### 1. Username Enumeration (Phase 1)
* Navigated to the login interface and submitted a dummy request (`testuser:password`) to capture the baseline failure behavior.
* Intercepted the `POST /login` request in **Burp Suite Proxy** and forwarded it to **Burp Intruder** utilizing a **Sniper** attack type.
* Set the payload position on the `username` parameter while keeping the password field static.
* Loaded the provided *Candidate Usernames* wordlist into the payload configuration and executed the attack.
* Analyzed the attack results by sorting the HTTP response **Length** column. 
* Identified a clear anomaly: The payload `americas` returned a different response length compared to all other candidates.
* Inspected the raw HTTP response for `americas` and confirmed it returned the error message `Incorrect password.`, confirming **`americas`** is a valid system username.

### 2. Password Brute-Force (Phase 2)
* Returned to Burp Intruder, hardcoded the `username` parameter to the validated user: `username=americas`.
* Repositioned the payload markers onto the `password` parameter: `password=§invalid-password§`.
* Cleared the previous wordlist and loaded the *Candidate Passwords* wordlist.
* Initiated the second Sniper attack.
* Monitored the results for deviations in the HTTP **Status Code**.
* Located a request that returned an HTTP **302 Found** redirection status code rather than the standard HTTP 200 OK error responses.
* Extracted the successful password payload linked to the 302 redirect:
  ```text
  charlie
### 3. Authentication & Verification
Authenticated into the target portal using the discovered credentials (americas:charlie).

The application successfully established the session and granted access to the account dashboard, resolving the challenge.

## 🛡️ Remediation & Prevention
Generic Error Messages: Implement uniform, generic error messages for all authentication failures regardless of whether the username or password was incorrect (e.g., use "Invalid username or password" uniformly).

Consistent Response Timing: Ensure backend database queries take roughly the same amount of execution time regardless of whether a username lookup fails or passes, preventing timing-based enumeration.

Rate Limiting & Account Lockouts: Enforce defensive rate-limiting (IP-based throttling) or temporary account lockout policies to prevent automated brute-force tools (like Burp Intruder) from guessing passwords at scale.
