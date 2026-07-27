## Authentication Vulnerabilities

### What is authentication?
Authentication is the process used to confirm a user's identity. Web services are exposed to the internet, so strong authentication controls are essential to prevent unauthorized access.

Authentication commonly relies on one or more of these factors:
- Knowledge: something the user knows (passwords, secrets).
- Possession: something the user has (a device, token, or phone).
- Inherence: something the user is (biometrics or behavioral traits).

Combining multiple factors (MFA) increases resilience against account compromise.

### Authentication vs Authorization
Authentication verifies identity; authorization determines what that identity may do. Both must be correctly implemented and kept separate in design and testing.

### How authentication flaws appear
Flaws typically result from either insufficient defenses against automated guessing (brute-force or credential stuffing) or from logic/implementation errors that let attackers bypass the authentication flow. Because authentication controls access, small mistakes often have large consequences.

### Impact of broken authentication
If an attacker obtains or bypasses credentials, they inherit the victim's privileges and access. Compromised administrative accounts can allow full takeover of the application and further access into internal systems. Even non-privileged compromises can reveal sensitive data or internal resources.

### Password-based login weaknesses
Password systems are vulnerable when credentials can be guessed, reused, or obtained by other means. Failures often stem from weak rate-limiting, poor password hygiene, predictable user modifications to required patterns, or exposed credentials from other breaches.

#### Brute-force and credential stuffing
Automated attacks try many username/password combinations. Credential stuffing reuses leaked credentials from other sites. Attackers use targeted lists and common password transformations to increase success rates. Effective throttling, lockouts, and MFA are primary defenses.

#### Username enumeration
Applications sometimes reveal whether a username exists through different responses in login, registration, or account-recovery flows. This information enables attackers to focus only on valid accounts and speeds up attack campaigns.

### Auditing checks
- Confirm rate-limiting, progressive delays, or lockouts for repeated failures.
- Test registration and recovery endpoints for user-existence leakage.
- Check for availability and proper enforcement of multi-factor authentication.
- Inspect session handling, alternate identifier logic, and password reset token management.

### Mitigations
- Require or strongly encourage MFA, especially for privileged users.
- Apply rate-limiting, progressive delays, or temporary account lockouts.
- Return generic error messages that avoid confirming user existence.
- Use breach/compromised-password checks and educate users on strong, unique passwords.
- Ensure secure issuance, expiration, and invalidation of password-reset tokens.

---


## Username enumeration — continued

When testing a login page for enumeration, pay attention to subtle differences in the application's responses. Three reliable indicators are:

- **HTTP status codes:** Most failed login attempts should produce consistent status codes. A request that returns a different code than the majority may indicate a different processing path (for example, the server only proceeds to password verification for known usernames).
- **Error text and page content:** Distinct messages for “unknown username” versus “incorrect password” reveal valid accounts. Even tiny variations — whitespace, punctuation, or an extra word — can be used to distinguish results programmatically.
- **Timing differences:** If the server performs extra work (such as checking a password) only for valid usernames, those requests may take measurably longer. Supplying an unusually long password can amplify timing differences for easier detection.

Attackers chain these signals: first identify valid accounts, then run targeted password attempts (or credential stuffing) against the discovered usernames.

### Practical lab: enumerating a username and cracking the password
This workflow describes a common hands-on approach using an intercepting proxy and automated attack tooling.

1. Capture a normal login attempt with your proxy and locate the `POST /login` request in the proxy history.
2. In the recorded request, mark the `username` field for payload injection and send that request to your attack tool. Leave `password` static for now.
3. Run a single-position attack (e.g., Sniper) with a list of candidate usernames. Sort or inspect results for outliers in length, status, or content.
4. When one request stands out (for example, the response body contains “incorrect password” while others show “invalid username”), note the corresponding username.
5. Reconfigure the attack: set the identified username as a constant and place a payload marker on the `password` field.
6. Launch the password attack using a candidate-password list. Monitor response indicators — a redirect (302) or a change in status code often denotes successful authentication.
7. Log in with the valid credentials and verify access to the target account page.

Note: while it's possible to brute-force usernames and passwords in a single multi-position attack, profiling the username space first is usually faster and reduces noise.

### PortSwigger lab: username enumeration via subtly different responses
This lab is a good example of how a login endpoint can leak account existence through small response differences. The key idea is to first identify a valid username by comparing the error messages returned for different candidate usernames. One of them may contain a slightly different message, such as a trailing space or punctuation change, which reveals that the username exists.

Typical workflow:
1. Capture the login request and send it to Intruder.
2. Attack the username parameter with a list of candidate usernames and extract the error message from the response body.
3. Look for the one response that is subtly different from the others; that usually indicates a valid account.
4. Fix that username and attack the password parameter with a password wordlist.
5. A successful login is often indicated by a redirect, a different status code, or a change in the response content.

This approach is more efficient than a full cluster bomb attack. A 100-by-100 combination test would require 10,000 attempts, while enumerating the username first narrows the problem dramatically and saves time.

---
### Quick checklist for practice labs
- Ensure your proxy is intercepting and that traffic is routed through it.
- Use conservative request rates to avoid unintentionally damaging target systems outside of authorized labs.
- Always document how you detected valid usernames (status, message, length, or timing) so you can reproduce findings.

