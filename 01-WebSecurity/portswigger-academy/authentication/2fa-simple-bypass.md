# Lab: 2FA Simple Bypass
**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Apprentice  
**Domain:** Authentication (Two-Factor Authentication Bypass)  

## 📝 Vulnerability Overview
The application implements a broken multi-stage authentication state machine. Upon verifying a user's primary credentials (username and password), the backend server initializes a fully authenticated or over-privileged session token *before* the user completes the secondary two-factor authentication (2FA) verification step. Because the application fails to validate the state of the 2FA completion on subsequent private endpoints, an attacker can completely bypass the 2FA requirement via direct URL manipulation (Forced Browsing).

## 🎯 Objective
Leverage a known set of victim credentials (`carlos:montoya`) to bypass the secondary 2FA verification screen and directly access Carlos's private account dashboard.

## 🛠️ Exploitation Steps

### 1. Workflow Mapping & Baseline Analysis
* Authenticated into the application using the provided test account credentials (`wiener:peter`).
* The application redirected to a secondary screen prompting for a 2FA code.
* Checked the simulated Email Client, retrieved the temporary OTP code, and completed the verification.
* Once fully authenticated, documented the exact URL destination of the landing dashboard:
  ```text
  https://<LAB-ID>.web-security-academy.net/my-account
Logged out of the active session to clear tokens.

### 2. Exploitation (Authentication Bypass)
Initiated a new login sequence using the target victim's primary credentials:

Plaintext
Username: carlos
Password: montoya
The server accepted the credentials and served the 2FA prompt page (/login2). At this stage, the backend server had already quietly generated a valid tracking session cookie for Carlos's identity in the browser background.

Instead of submitting a verification code, manually forced the browser navigation address directly to the previously mapped endpoint:

Plaintext
https://<LAB-ID>.web-security-academy.net/my-account
### 3. Verification
The application's backend processed the request, read Carlos's session cookie, and failed to check if the 2FA flag was validated.

The application loaded Carlos's complete account dashboard directly, achieving complete authentication bypass without entering the required OTP token.

## 🛡️ Remediation & Prevention
Implement Strict Multiphase Authentication States: A session token generated after step 1 (password verification) must remain in a restricted, unverified state. The backend must enforce a strict server-side authorization check on all endpoint views to ensure the user has explicitly transitioned from an AWAITING_2FA status to FULLY_AUTHENTICATED.

Deny-by-Default Access Control: Protect user dashboard routes (like /my-account) using a global authentication middleware wrapper that verifies both the primary identity session and the successful evaluation of the multi-factor validation routine before returning data.
