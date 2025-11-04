# 📝 Gmail Login Functionality — Test Case Matrix

This matrix provides structured test cases for Gmail login functionality. Test cases are grouped into Positive, Negative, and Edge categories and are written in Gherkin (Given / When / Then) to make them executable and automation-ready.

---

## 1. Positive Cases (Happy Path & Convenience)

| Test ID | Category | Test Case Title | Gherkin Steps (Given / When / Then) | Expected Result |
|---|---:|---|---|---|
| TC-P-01 | Positive | Successful Standard Login with Valid Credentials | Given the user is on the Gmail login page.  
When the user enters a valid registered email and correct password and submits the form.  
Then the user is redirected to the Inbox and a secure session token is issued. | Redirect to Inbox (HTTP 200); inbox content visible; secure session token created. |
| TC-P-02 | Positive | Successful Login with MFA (TOTP) | Given the user has TOTP-based MFA enabled.  
When the user enters valid primary credentials and the correct TOTP code.  
Then the user is granted access to the Inbox. | TOTP validated; user redirected to Inbox; MFA confirmation recorded. |
| TC-P-03 | Positive | "Remember Me" Session Persistence | Given the user checks "Remember Me" during a successful login.  
When the user closes the browser and re-opens it after 1 hour.  
Then the user remains logged in or is shown a pre-filled email without requiring the password. | Session persisted or email pre-filled; user does not need to re-enter password within configured persistence window. |

---

## 2. Negative Cases (Input Validation & Basic Security)

| Test ID | Category | Test Case Title | Gherkin Steps (Given / When / Then) | Expected Result |
|---|---:|---|---|---|
| TC-N-01 | Negative | Invalid Email Format (missing '@') | Given the user is on the Gmail login page.  
When the user enters an email missing the "@" symbol (e.g., userdomain.com) and tries to continue.  
Then the client validates the input and displays an error. | Client-side error: "Enter a valid email address"; no server submission occurs. |
| TC-N-02 | Negative | Non-Registered Email Attempt | Given the user is on the Gmail login page.  
When the user enters an email not registered in the system.  
Then the system shows a generic message indicating no matching account was found. | Generic, non-revealing error such as "Could not find your Google Account"; prevents information leakage. |
| TC-N-03 | Negative | Repeated Incorrect Password Attempts (Brute Force Protection) | Given the user has a registered email.  
When the user submits incorrect passwords repeatedly (exceeds threshold).  
Then the system warns and triggers anti-abuse controls (CAPTCHA, rate-limit, temporary lockout). | On threshold breach, present CAPTCHA or rate limit; subsequent attempts require additional verification. |

---

## 3. Edge Cases (Session Management & Resilience)

| Test ID | Category | Test Case Title | Gherkin Steps (Given / When / Then) | Expected Result |
|---|---:|---|---|---|
| TC-E-01 | Edge | Login with Saved Credentials on Slow Network | Given the user's device has saved credentials and a simulated slow network (3G).  
When the user clicks Login.  
Then the system shows a loading state and completes login within the defined performance SLA. | Login completes within SLA (e.g., < 5s on 3G); UI shows progress and does not time out. |
| TC-E-02 | Edge | Stale Session Token Usage after Logout | Given the user logs in and then explicitly logs out, invalidating the session token.  
When an attacker attempts to reuse the stale token.  
Then the system rejects the request and requires re-authentication. | Stale token rejected with authentication error (HTTP 401/403); no access granted. |
| TC-E-03 | Edge | System-Wide Account Lockout | Given an account has 10 consecutive failed login attempts.  
When the account owner tries to log in with correct credentials during the lockout period.  
Then the system prevents login and shows a lockout message until the period expires or remediation occurs. | Account remains locked; login blocked until lockout period ends or account recovery is performed. |

---

## Notes

- Use these Gherkin scenarios as the basis for automation tests (Cucumber, Playwright, Cypress with Cucumber preprocessor).
- Ensure NFRs for session timeout, rate limits, and MFA handling are captured in the test environment configuration.
- For performance tests, include network shaping (tc/netem or browser DevTools throttling) to simulate slow networks.


