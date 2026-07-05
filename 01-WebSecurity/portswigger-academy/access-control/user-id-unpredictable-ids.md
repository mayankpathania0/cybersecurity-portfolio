# Lab: User ID Controlled by Request Parameter, with Unpredictable User IDs
**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Apprentice  
**Domain:** Access Control (Insecure Direct Object Reference - IDOR)  

## 📝 Vulnerability Overview
The application attempts to secure user account pages by referencing them via unpredictable Globally Unique Identifiers (GUIDs) instead of sequential integers. While this prevents simple enumeration attacks, it relies on "security through obscurity." Because the application exposes these GUIDs publicly and lacks server-side authorization checks on the `id` parameter, it remains highly vulnerable to Insecure Direct Object Reference (IDOR).

## 🎯 Objective
Locate the unique identifier for the user `carlos`, exploit the IDOR flaw to access his account dashboard, and retrieve his private API key.

## 🛠️ Exploitation Steps

### 1. Information Gathering & Reconnaissance
* Navigated to the public blog section of the application and located a post authored by user `carlos`.
* Clicked on the author profile link for `carlos` and inspected the resulting URL parameters.
* Extracted Carlos's publically exposed GUID from the query string:
  ```text
  29f7f4d7-4dac-4b5a-8ff7-21dd419e90c1
### 2. Authorization Bypass (IDOR)
* Logged into the application using the provided low-privilege credentials (wiener:peter).
* Navigated to the "My Account" page, which loaded the following URL structure: Plaintext  https://<LAB-ID>.web-security-academy.net/my-account?id=<WIENER-GUID>    
* Used Burp Suite Proxy to intercept the request (or modified the parameter directly in the browser address bar), replacing the account owner's ID with Carlos's extracted GUID: Plaintext  /my-account?id=29f7f4d7-4dac-4b5a-8ff7-21dd419e90c1
    
### 3. Exfiltration
* The server processed the manipulated request without verifying if the authenticated session token matched the requested resource ID.
* The application rendered Carlos's account page, exposing his sensitive account data.
* Retrieved the target API key: Plaintext  b5LwyOze9vRvqJDkyYaISV891C3WrD55
   
## 🛡️ Remediation & Prevention
* Implement Robust Access Controls: Never rely on the un-guessability of identifiers as a primary defense vector. The backend application must validate that the currently logged-in user session has explicit authorization to read or modify the resource specified by the incoming id parameter.
* Adopt Object-Level Access Control Frameworks: Use server-side verification checks (e.g., matching session.user_id === request.vulnerable_id) before returning sensitive data to the client.
