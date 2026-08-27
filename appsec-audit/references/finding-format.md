# Finding Format, Severity & ISO 27001 Mapping

## Finding Format

Use this for every issue found — don't collapse it into prose:

```
[SEVERITY] Title
Location: path/to/file.ts:42
ISO 27001 Control: A.8.xx – <control name>
Issue: <what's wrong, concretely>
Impact: <what an attacker could do>
Fix: <concrete remediation>
Test: <curl command proving the issue / verifying the fix>
```

For passing checks, use a shorter form so the report stays readable but the review is still visible:

```
✅ [Section 7 — SQL injection] Reviewed db/queries/*.ts and prisma schema — all queries use Prisma's parameterized query builder, no raw string interpolation found.
```

## Severity rubric

- **Critical** — remote code execution, full auth bypass, mass data exfiltration, or anything exploitable at scale with no prerequisite access (e.g. exposed `service_role` key, SQLi on an unauthenticated endpoint).
- **Severe** — significant impact but needs some precondition (a valid low-privilege account, specific timing, or a less-common configuration) — e.g. IDOR on an authenticated endpoint, stored XSS in an admin-only view, session fixation.
- **Medium** — real weakness with limited blast radius or requiring significant attacker effort — e.g. missing rate limiting on a low-value endpoint, verbose error messages, missing SRI.
- **Low** — best-practice gaps, defense-in-depth items, or issues needing an unusual combination of circumstances to matter — e.g. missing security headers with no demonstrated bypass path, outdated but non-vulnerable dependency.

Use judgment over the letter of these definitions — e.g. an IDOR that exposes payment details is Critical even though IDOR is listed under Severe by default; adjust for what the specific data or action actually is.

## ISO/IEC 27001:2022 Annex A control reference

Cite the most relevant control(s) per finding:

- A.5.15 Access control
- A.5.23 Information security for cloud services
- A.5.34 Privacy and protection of PII
- A.8.2 Privileged access rights
- A.8.3 Information access restriction
- A.8.5 Secure authentication
- A.8.9 Configuration management
- A.8.12 Data leakage prevention
- A.8.13 Information backup
- A.8.16 Monitoring activities
- A.8.23 Web filtering
- A.8.24 Use of cryptography
- A.8.26 Application security requirements
- A.8.28 Secure coding
- A.8.29 Security testing in development
- A.8.31 Separation of dev/test/prod environments

If none of these fit cleanly (e.g. a business-logic race condition), pick the closest one (A.8.26 or A.8.28 are reasonable defaults) and say so rather than forcing a bad fit silently.

## Final summary table

Close every audit with:

| # | Endpoint/Area | Severity | ISO Control | Status |
|---|---|---|---|---|

One row per finding, ordered by descending severity. "Status" is typically "Open" for a fresh audit — if this is a re-audit after fixes, use "Fixed" / "Open" / "Won't fix" as reported by the person.
