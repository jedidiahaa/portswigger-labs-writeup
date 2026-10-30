# Lab Report: PortSwigger Web Security Academy, Apprentice Labs

> The targets in this report are intentionally vulnerable training labs. This is a learning write-up, not a real security engagement.

**Date:** October 2026
**Author:** Jedidiah (GitHub: jedidiahaa)
**Tool:** Burp Suite Community Edition (Proxy, Repeater, Intruder) and browser developer tools

## Summary

I completed 11 Apprentice-level labs on the PortSwigger Web Security Academy, covering seven vulnerability categories: SQL injection, cross-site scripting (XSS), broken access control, path traversal, OS command injection, authentication flaws (username enumeration), and cross-site request forgery (CSRF).

Each lab told me which vulnerability class to look for, so this was guided exploitation, not black-box discovery. For each one I exploited the flaw manually, wrote down what happened, and rated the severity as if the same flaw existed in a real application. Ratings use CVSS 3.1 as a guide (see `docs/severity-rationale.md`).

| Severity | Count | Findings |
|---|---|---|
| Critical | 3 | SQL injection login bypass (2), unprotected admin panel (5), OS command injection (9) |
| High | 3 | SQL injection data retrieval (1), role controlled by cookie (6), path traversal (8) |
| Medium | 5 | Reflected XSS (3), stored XSS (4), IDOR (7), username enumeration (10), CSRF change email (11) |

## Scope and method

- **Targets:** 11 lab instances on the Academy (`*.web-security-academy.net`), each a small shop or blog application built to demonstrate one vulnerability. Labs are isolated and reset between sessions.
- **Testing window:** October 2026.
- **Method:** Manual testing as an unauthenticated or low-privilege user. No vulnerability scanner was used. Burp Intruder was used for the Finding 10 wordlist attacks.
- **Out of scope:** Denial-of-service testing, network and infrastructure testing, and anything on the Academy platform outside the lab applications. Practitioner and Expert labs are not covered.

## Findings

### SQL injection

#### Finding 1: SQL injection in the WHERE clause (hidden data retrieval)

**Severity:** High (CVSS 7.5, potential impact; demonstrated impact was limited)
**Description:** The product category filter inserted user input directly into a SQL `WHERE` clause (`... WHERE category = 'Gifts' AND released = 1`) without sanitization.
**Exploitation:** Submitting `category=Gifts'+OR+1=1--` closed the string, forced the condition to always be true, and commented out the `released = 1` check, so unreleased products were returned.
**Demonstrated impact:** Disclosure of products the application meant to hide.
**Potential impact:** The same injection point could be used to read other tables, such as user data.
**Recommendation:** Use parameterized queries everywhere user input reaches SQL. Never build queries by string concatenation.

#### Finding 2: SQL injection allowing login bypass

**Severity:** Critical (CVSS 9.8)
**Description:** The login form built its query by concatenating the username and password into SQL (`... WHERE username = '<input>' AND password = '<input>'`).
**Exploitation:** A username of `administrator'--` with any password commented out the password check, and the application logged me in as the administrator.
**Demonstrated impact:** Full authentication bypass as an administrator, with no valid password.
**Recommendation:** Parameterized queries, as in Finding 1. Also log and alert on malformed login attempts.

### Cross-site scripting (XSS)

#### Finding 3: Reflected XSS into HTML context with nothing encoded

**Severity:** Medium (CVSS 6.1)
**Description:** The search function echoed the search term into the results page without encoding.
**Exploitation:** Searching for `<script>alert(1)</script>` made the script run in the browser, confirmed by an alert box.
**Demonstrated impact:** Arbitrary JavaScript execution in my own browser session.
**Potential impact:** A crafted link could run script in a victim's session. The victim has to click it, which is why this is Medium.
**Recommendation:** Context-appropriate output encoding, plus a Content Security Policy as a secondary control.

#### Finding 4: Stored XSS into HTML context with nothing encoded

**Severity:** Medium (CVSS 6.1; High if administrators or other privileged users routinely view the affected content)
**Description:** Blog comments were stored and shown to every visitor of the post without encoding.
**Exploitation:** Posting `<script>alert(1)</script>` as a comment made the script run whenever the post was viewed, including by me.
**Demonstrated impact:** Persistent script execution for anyone who views the post.
**Potential impact:** Session token theft or actions performed as the victim. I did not demonstrate this. Unlike Finding 3 no crafted link is needed, but the victim still has to load the page.
**Recommendation:** The same encoding and CSP controls as Finding 3, plus validation of stored user content.

### Broken access control

#### Finding 5: Unprotected admin functionality

**Severity:** Critical (CVSS 9.1)
**Description:** An admin panel sat at an unlinked URL with no authentication or authorization check. Security relied on the URL not being linked from the interface.
**Exploitation:** `/robots.txt` listed `Disallow: /administrator-panel`, which was meant to hide the path but revealed it. Visiting it directly gave unauthenticated admin access, including the ability to delete user accounts.
**Demonstrated impact:** Full administrative control without logging in.
**Recommendation:** Enforce authentication and role-based authorization on the server for every admin route. Never treat an unlisted URL as a security boundary, and never use `robots.txt` to hide sensitive paths.

#### Finding 6: User role controlled by a client-side cookie

**Severity:** High (CVSS 8.8)
**Description:** The application decided whether a logged-in user was an administrator from a client-supplied cookie (`Admin=false`).
**Exploitation:** Changing the cookie from `false` to `true` in browser developer tools and reloading the account page gave admin access, including deleting other users.
**Demonstrated impact:** Privilege escalation from a standard user to administrator.
**Recommendation:** Never trust client-supplied data (cookies, hidden fields, headers) for authorization. Check roles server-side against a trusted session record on every request.

#### Finding 7: User ID controlled by request parameter (IDOR)

**Severity:** Medium (CVSS 6.5; High if the exposed data allows account takeover)
**Description:** The account page loaded data for the user ID in the URL (`/my-account?id=wiener`) with no check that the logged-in user owned that ID.
**Exploitation:** Changing `id=wiener` to `id=carlos` while logged in as a different user returned carlos's account information.
**Demonstrated impact:** Any logged-in user can read another user's account data.
**Recommendation:** On every request for user-specific data, verify server-side that the session's user matches the requested record.

### Server-side vulnerabilities

#### Finding 8: File path traversal

**Severity:** High (CVSS 7.5)
**Description:** The image endpoint (`/image?filename=58.jpg`) passed the filename to the filesystem without checking that it stayed inside the images directory.
**Exploitation:** Requesting `/image?filename=../../../etc/passwd` used `../` sequences to leave the directory and return the server's `/etc/passwd`.
**Demonstrated impact:** Arbitrary file read, limited to what the web server process can access.
**Recommendation:** Normalize and validate paths server-side, reject traversal sequences, and resolve files against an allowlist.

#### Finding 9: OS command injection

**Severity:** Critical (CVSS 9.8)
**Description:** The "check stock" feature passed a user-supplied `storeId` into a server-side shell command without sanitization.
**Exploitation:** Submitting `storeId=1;whoami` appended a second command. Its output appeared in the response, confirming command execution.
**Demonstrated impact:** Arbitrary command execution as the web application's user.
**Recommendation:** Avoid passing user input to a shell. If unavoidable, use APIs that bypass the shell, allowlist the input strictly, and run the process with minimal privileges.

### Authentication flaws

#### Finding 10: Username enumeration via different responses

**Severity:** Medium (CVSS 5.3 alone; the missing rate limiting made the follow-up brute force possible)
**Description:** The login returned different messages for an unknown username ("Invalid username") and a wrong password ("Incorrect password"), which let me confirm valid usernames.
**Exploitation:** I ran a list of candidate usernames through the login endpoint with Burp Intruder. One response differed in length and message, confirming a valid username (redacted). A second Intruder attack guessed the password for that account. The correct password stood out as a `302` redirect among `200` failures.
**Demonstrated impact:** Access to a valid account by brute force, with no lockout or rate limiting.
**Recommendation:** Return identical generic errors, response lengths and timing for both cases. Add rate limiting and account lockout or CAPTCHA.

### Cross-site request forgery (CSRF)

#### Finding 11: CSRF with no defenses

**Severity:** Medium (CVSS 6.5)
**Description:** The "change email" request had no anti-CSRF token, no `Origin` or `Referer` validation, and no `SameSite` cookie restriction, relying only on the session cookie.
**Exploitation:** I built a self-submitting HTML form on a different origin (the lab's exploit server) that posted a new email address. When a logged-in victim loaded it, their browser attached the session cookie and the email changed.
**Demonstrated impact:** The victim's registered email was changed without their action beyond loading a page.
**Potential impact:** A changed email can lead to account takeover through a password reset. I did not test that.
**Recommendation:** Use a unique per-session or per-request anti-CSRF token validated server-side on every state-changing request. Set session cookies to `SameSite=Strict` or `Lax`, and validate `Origin` or `Referer` as a secondary control.

## Recommendations

| Theme | Findings | Priority action |
|---|---|---|
| Input validation and output encoding | 1, 2, 3, 4, 8, 9 | Parameterized SQL, context-aware output encoding, no user input in shell commands or file paths without allowlisting |
| Server-side access control | 5, 6, 7 | Enforce authentication and authorization on every request server-side |
| Authentication hardening | 10 | Identical error responses, rate limiting, lockout or CAPTCHA |
| CSRF defenses | 11 | Anti-CSRF tokens, `SameSite` cookies, `Origin` or `Referer` checks |

## Conclusion

All 11 labs were vulnerable to the flaw they were built to demonstrate. In each, one missing control was enough: input handling, server-side authorization, or a CSRF token. All were exploited manually with a free proxy tool and standard browser features.
