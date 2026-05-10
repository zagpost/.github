# Security Policy

## Our commitment

**Zag Post** is a privacy-first messaging app. Security vulnerabilities are taken
seriously and treated as the highest priority. We are committed to working with
security researchers in good faith and will never pursue legal action against
researchers who follow this policy.

---

## Supported versions

Only the latest production release receives security fixes.

| Version  | Supported |
|----------|-----------|
| Latest   | ✅        |
| Older    | ❌        |

---

## Reporting a vulnerability

**Do not open a public GitHub issue for security vulnerabilities.**

Report privately through one of the following channels:

- **GitHub Private Vulnerability Reporting** (preferred):  
  Use the "Report a vulnerability" button on the Security tab of this repository.
  GitHub keeps the report confidential and routes it directly to maintainers.

- **Email**: security@zagpost.org  
  Encrypt sensitive reports with our PGP key: https://keybase.io/thelukez

---

## What to include in your report

A useful report contains:

- A clear description of the vulnerability and its potential impact
- The affected component (web app, API, crypto layer, etc.)
- Step-by-step reproduction instructions or a minimal proof of concept
- Any relevant environment details (browser, OS, network conditions)

Reports without reproduction steps will still be reviewed, but take longer to triage.

---

## What to expect from us

| Milestone                         | Target timeline   |
|-----------------------------------|-------------------|
| Acknowledgement of your report    | Within 48 hours   |
| Triage and severity assessment    | Within 7 days     |
| Fix deployed for critical issues  | Within 14 days    |
| Public disclosure                 | After fix is live |

We will keep you updated throughout the process. If you do not hear back within
48 hours, please follow up - reports can occasionally land in spam.

---

## Coordinated disclosure

We follow a **90-day coordinated disclosure** policy.

- We aim to ship a fix well before the 90-day deadline.
- If a fix is released early, we will coordinate public disclosure with you.
- If 90 days pass without a fix, you are free to disclose.
- We will credit you in the release notes and our security hall of fame unless you prefer to remain anonymous.

---

## Safe harbor

If you act in good faith under this policy, we commit to:

- Not pursuing legal action against you
- Working with you to understand and fix the issue
- Treating your report confidentially until a fix is deployed
- Publicly crediting your contribution if you wish

We ask that you:

- Give us reasonable time to investigate and fix before disclosing publicly
- Avoid accessing, modifying, or deleting data that is not yours
- Limit testing to what is necessary to demonstrate the vulnerability
- Not run automated scanners or DoS attacks against production systems
- Delete any sensitive data discovered during research as soon as it is no longer needed to document the vulnerability

---

## Out of scope

The following are generally **not** considered valid reports:

- Self-XSS (attacks that require a victim to execute their own payload)
- Clickjacking on pages with no sensitive actions
- Missing HTTP security headers with no demonstrated impact
- Theoretical vulnerabilities with no working proof of concept
- Outdated browser or OS version issues
- Rate limiting on non-sensitive endpoints
- DNS/email configuration (SPF, DMARC, DKIM) without demonstrated impact
- Reports generated automatically by scanners without manual verification

---

## Scope

In scope for vulnerability reports:

- `zagpost.org` - the main page
- `app.zagpost.org` - the web application
- `api.zagpost.org` - the backend API
- The end-to-end encryption implementation (client-side crypto)
- Authentication and session management
- Attachment encryption and storage
- Phone number hashing

---

## Hall of fame

We publicly thank researchers who have reported valid vulnerabilities responsibly:

*(No entries yet - be the first.)*

---

## Contact

security@zagpost.org
