The targets in this repository are intentionally vulnerable training labs from PortSwigger's Web Security Academy; this is a learning write-up, not a real security engagement.

# PortSwigger Labs Write-up

A report on 11 Apprentice-level labs covering SQL injection, XSS, broken access control, path traversal, OS command injection, username enumeration and CSRF. Each finding has a description, how I exploited it, the demonstrated and potential impact, a severity rating with reasoning, and a fix.

## Contents

- [`REPORT.md`](REPORT.md): the full report
- [`docs/severity-rationale.md`](docs/severity-rationale.md): CVSS 3.1 vectors and reasoning for each rating

## What this shows

- Manual exploitation of common web vulnerabilities with Burp Suite Community Edition (Proxy, Repeater, Intruder)
- Writing findings with impact, severity reasoning and remediation
- Separating what I demonstrated from what is only possible in a real application

## What this does not show

- Real-world testing. Every lab named its vulnerability class in advance.
- Any Practitioner or Expert level work.
- Any testing against systems other than the labs.

## Attribution and terms

The Web Security Academy and its labs are created by PortSwigger (https://portswigger.net). This repository is not affiliated with or endorsed by PortSwigger. It contains my own notes and findings, with no lab descriptions or official solutions reproduced, and no PortSwigger logos or marks. The Academy's terms are on the PortSwigger website.
