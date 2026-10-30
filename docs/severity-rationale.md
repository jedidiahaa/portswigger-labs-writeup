# Severity rationale

Ratings describe each flaw as it would appear in a real application, using CVSS 3.1 base scores as a guide. The labs themselves contain no real data. Where the demonstrated impact in the lab was smaller than the potential impact, the report says so.

| # | Finding | Rating | CVSS 3.1 vector | Score | Why |
|---|---|---|---|---|---|
| 1 | SQLi, data retrieval | High | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N | 7.5 | Query logic is attacker-controlled. Only unreleased products were shown in the lab. |
| 2 | SQLi, login bypass | Critical | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H | 9.8 | Unauthenticated admin access. |
| 3 | Reflected XSS | Medium | AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N | 6.1 | Victim must click a crafted link. |
| 4 | Stored XSS | Medium | AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N | 6.1 | Victim must load the page. High if privileged users view it. |
| 5 | Unprotected admin panel | Critical | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N | 9.1 | Unauthenticated administrative control. |
| 6 | Role in cookie | High | AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H | 8.8 | Any user becomes admin. |
| 7 | IDOR | Medium | AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N | 6.5 | Authenticated read of other users' data. High if the data enables takeover. |
| 8 | Path traversal | High | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N | 7.5 | Arbitrary file read. |
| 9 | OS command injection | Critical | AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H | 9.8 | Unauthenticated code execution. |
| 10 | Username enumeration | Medium | AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N | 5.3 | Low alone. Missing rate limiting enabled the brute force. |
| 11 | CSRF, change email | Medium | AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:H/A:N | 6.5 | Victim must load a page. No takeover was shown. |

The `UI:R` (user interaction required) metric is the main reason findings 3, 4 and 11 score lower than findings 2 and 9.
