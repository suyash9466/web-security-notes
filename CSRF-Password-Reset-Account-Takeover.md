Cross-Site Request Forgery (CSRF) Password Reset Account Takeover

---

Vulnerability Summary
The application is vulnerable to Cross-Site Request Forgery (CSRF) in the password reset functionality.
The endpoint:
https://xyz/app/profile/reset_password

does not implement any CSRF protection mechanisms such as anti-CSRF tokens or request validation. The request is processed solely based on the victim’s authenticated session.

This allows an attacker to change a victim’s password without their knowledge.

---

Conceptual View
CSRF exploits the trust a web application places in a user’s browser.
- Victim is authenticated
- Attacker forces a request
- Server trusts it executes action
Here, the sensitive action is password reset, making this vulnerability critical.

---

Impact
- Full Account Takeover (ATO)
Attacker can:
 - Change victim password
 - Lock out legitimate user
 - Gain persistent unauthorized access
Severity: High / Critical

---

Root Cause
- No CSRF token implemented
- No Origin/Referer validation
- Sensitive functionality accessible via simple POST request
- Application relies only on session cookies

---

Steps to Reproduce
1. Login to the application using a valid account.
2. Keep the session active.
3. Create a malicious HTML file with an auto-submitting form.
4. Open the malicious file in the victim’s browser.
5. The request is automatically sent to the server.
6. Victim’s password is changed without any interaction.
7. Logout and verify:
   - Old password (does not work)
   - New password (works)

---

Proof of Concept (PoC)

<html>
  <body>
    <iframe name="hiddenFrame" style="display:none;"></iframe>
    <form action="https://xyz/app/profile/reset_password" method="POST" target="hiddenFrame">
      <input type="hidden" name="password" value="Pwned@123">
      <input type="hidden" name="password2" value="Pwned@123">
      <input type="hidden" name="reset" value="reset">
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>

---

Attack Scenario
An attacker hosts a malicious HTML page and tricks the victim into visiting it (via phishing or social engineering).
Since the victim is already authenticated:
- Browser automatically includes session cookies
- Request is treated as legitimate
- Password is changed silently
No user interaction is required.

---

Video Demonstration
The following video demonstrates the full exploitation flow:
- Victim logged in
- Malicious HTML execution
- Password changed without consent
- Successful login using attacker-defined password
   [Watch Full PoC](https://youtu.be/NA5vm83g7X0?si=39rCPqoK81faXBM4)

---

Mitigation / Fix
To prevent CSRF attacks:
- Implement anti-CSRF tokens (synchronizer token pattern)
- Enforce SameSite cookies (Strict/Lax)
- Validate Origin and Referer headers
- Require current password confirmation before reset
- Add re-authentication for sensitive actions

---

References
OWASP CSRF Prevention Cheat Sheet
OWASP Top 10 – A01/A05 (Broken Access Control / Security Misconfiguration context)
