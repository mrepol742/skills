---
name: appsec-audit
description: Perform a full application security audit of a codebase — auth/session security, API authorization/BOLA/IDOR, injection (SQLi/NoSQLi/SSTI/XXE/command/path traversal/SSRF), CSRF/CORS/headers, database and RLS misconfiguration, secrets/dependency/supply-chain risk, AI/LLM endpoint hardening (prompt injection, token bombs), business-logic race conditions, cryptography, infra exposure, and memory leaks. Use whenever the user asks to "audit", "pentest", "security review", "harden", or "find vulnerabilities in" a codebase or specific endpoints, wants an OWASP-style or ISO 27001-mapped report, or asks about one specific risk category (rate limiting, IDOR, JWT security, RLS policies, SSRF, prompt injection, secrets exposure, etc.) even without asking for a "full audit." Produces a section-by-section report with file:line findings, severity, ISO 27001 mapping, concrete fixes, and runnable curl commands to verify each.
---

# Application Security Audit

This is a senior-appsec-auditor checklist, not a vulnerability-scanner wrapper: work through the codebase systematically, section by section, and reason about each check against the actual code rather than pattern-matching on keywords. A grep for `eval(` tells you where to look, not whether it's exploitable — read the surrounding code before flagging or clearing something.

## Ground rules

- **Work section by section**, in the order given in `references/checklist.md`. Don't skip around — thoroughness matters more than getting to a punchy conclusion fast.
- **Every check gets an explicit verdict.** If a check passes, say so ("✅ Reviewed — parameterized queries used throughout `db/queries/*.ts`, no raw interpolation found") rather than silently omitting it. The person needs to know what was actually checked, not just what's broken.
- **If a whole section doesn't apply** (no GraphQL in this stack, no file uploads, no AI routes), state that explicitly and move on — don't force-fit findings.
- **Cite exact file paths and line numbers** for every finding. If you can't pin down a line number, cite the function/route name and say so.
- **Every finding uses the Finding Format** in `references/finding-format.md` — don't summarize findings away into prose. Specifics are the entire value of an audit.
- **Every finding gets a runnable curl command** (or short set of commands) that demonstrates the issue or verifies the fix. See `references/curl-tests.md` for the pattern and examples per category — adapt them to the actual routes/params found, don't paste generic placeholders.
- **Assign severity and an ISO/IEC 27001:2022 Annex A control** to every finding — see `references/finding-format.md` for the severity rubric and the control list.
- **This is a defensive exercise.** The goal is to find and fix real weaknesses in code the person owns or is authorized to review, with concrete remediations — not to produce a generic attack how-to. Keep findings and curl commands scoped to *this codebase's* actual routes and params.

## Workflow

1. **Orient.** Identify the stack: language/framework, API style (REST/GraphQL/RPC), database and ORM, auth approach (session cookies, JWT, OAuth/SSO), and whether AI/LLM routes, file uploads, webhooks, or real-time (WebSocket) features exist. This determines which sections in the checklist apply.
2. **Map the API surface.** Enumerate routes/endpoints (from router files, OpenAPI/Swagger specs, or framework conventions) and classify each as public or non-public before diving into individual checks — several sections (authz, rate limiting, CORS) key off this map.
3. **Work through `references/checklist.md`** top to bottom. For large codebases, prioritize auth, payment/business-logic, and any route handling file uploads or user-supplied URLs first — these carry the highest severity findings most often — then work outward. Say explicitly if you're sampling rather than covering 100% of routes, and offer to go deeper on request.
4. **Write findings** using the format in `references/finding-format.md` as you go, not all at the end — this avoids losing track of specifics gathered along the way.
5. **Generate curl tests** per `references/curl-tests.md` for each finding and for a representative set of passing checks worth being able to re-verify (e.g., confirming a 401 on a protected route).
6. **Close with the summary table** specified in `references/finding-format.md`.

## Scope and safety notes

- If a check requires actually sending traffic (not just reading code) to confirm — e.g., firing repeated login requests to test rate limiting — construct the curl command for the person to run themselves against their own environment rather than executing it against a live target yourself, unless they've explicitly given you a sandboxed/local environment to run commands against.
- If the codebase reveals what looks like a currently-exploited vulnerability or live incident (not just a latent weakness), say so plainly and prioritize it at the top of the report rather than filing it alphabetically with everything else.
- Don't invent findings to pad the report. A short, accurate report that says "reviewed, no issues found" for most sections is more valuable than a long one stretching thin evidence into high-severity findings.

## Reference files

- `references/checklist.md` — the full audit checklist, organized by section. This is the core of the skill; read it in full before starting.
- `references/finding-format.md` — the exact Finding Format, severity rubric, and ISO 27001 Annex A control list to cite.
- `references/curl-tests.md` — patterns and examples for the verification curl command that accompanies every finding.
