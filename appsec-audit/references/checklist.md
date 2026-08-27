# Audit Checklist

Table of contents: 1. Registration/Login/Auth · 2. Session & Token Security · 3. AI Route Endpoint Security · 4. General API Authz/Authn · 5. Input Validation, XSS & Injection · 6. CSRF, CORS & Headers · 7. Database Query Security · 8. Secrets, Dependencies & Supply Chain · 9. Logging, Errors & Monitoring · 10. Business Logic & Race Conditions · 11. Real-Time & Alternative Protocols · 12. Infrastructure & Deployment · 13. Advanced Injection & Parsing · 14. Cryptography & Randomness · 15. Data Privacy & Compliance · 16. Infrastructure Exposure & Misconfiguration · 17. Client-Side Trust Boundary & Rendering Leaks · 18. Memory Leak Review

Lines marked **[ADDED]** are checks folded in beyond the original brief — call these out to the user as "added" points in your summary so they know what's new versus what they specified.

## 1. Registration / Login / Auth API Security

For every endpoint related to register, login, password reset, magic link, OTP, and session refresh:
- **Rate limiting**: per-IP and per-account throttling for both brute-force and request-volume DoS (middleware, API gateway, or Redis token bucket). Flag any endpoint with none.
- **Rate-limit bypass**: can the limiter be bypassed by spoofing `X-Forwarded-For`/other client-supplied IP headers? Is real client IP derived correctly behind a proxy/CDN?
- **[ADDED] Account lockout / progressive delay**: distinct from rate limiting — after N failed attempts, is there escalating delay or temporary lockout on the *account* (not just the IP), with a safe unlock path that itself can't be abused to lock out a legitimate user (lockout-as-DoS)?
- **Disposable/temporary email blocking**: validation against disposable email domains. Flag if registration accepts any syntactically valid email with no domain reputation check.
- **Anti-bot protection**: CAPTCHA/Turnstile enforced on register/login/reset forms, verified **server-side**.
- **Honeypot fields**: hidden field checked server-side, silently soft-failing.
- **Account enumeration**: do login/register/reset error messages or response timing leak whether an email exists?
- **Password policy**: minimum length/complexity; bcrypt/argon2/scrypt for storage (never plain SHA/MD5).
- **[ADDED] Password reuse/history**: is a changed password checked against the user's previous N passwords where a policy claims to enforce this?
- **Credential stuffing protection**: check against known-breached password lists (e.g. HaveIBeenPwned API)?
- **MFA/2FA**: enforceable if offered; backup/recovery codes hashed and single-use.
- **[ADDED] OTP expiry & single-use**: OTP codes expire quickly (minutes, not hours) and are invalidated after one successful use or after N wrong attempts.
- **Password reset token strength**: long, random, single-use, time-limited — not predictable/sequential.
- **[ADDED] Step-up re-authentication**: sensitive account changes (email change, password change, payment method, 2FA disable) require re-entering the current password or a fresh MFA challenge, not just an active session.

## 2. Session & Token Security

- **JWT specifics**: algorithm explicitly allow-listed server-side (reject `alg: none`); signing secret/key strong, not hardcoded; `exp` enforced; no sensitive PII in payload (JWTs are base64, not encrypted).
- **[ADDED] JWT algorithm confusion / `kid` injection**: if both RS256 and HS256 are accepted, confirm an attacker can't sign a token with HS256 using the public key as the secret. If a `kid` (key ID) header is used to look up the verification key, confirm it can't be manipulated to point at an attacker-controlled key or path (e.g. `kid: "../../../etc/passwd"`).
- **Session/token flags**: HttpOnly, Secure, SameSite=Strict/Lax on cookies; proper expiry; refresh-token rotation and revocation on logout/password change.
- **[ADDED] Cookie domain/path scoping**: cookies aren't scoped broader than necessary (e.g. set on the parent domain when only one subdomain needs them), which would expose them to other subdomains.
- **[ADDED] Session fixation**: session ID/token is regenerated on login and on any privilege-level change — confirm the app doesn't keep a pre-login session ID valid after authentication.
- **[ADDED] Token storage location**: if using token-based auth (not just cookies), confirm access tokens aren't stored in `localStorage`/`sessionStorage` where they're readable by any injected script (an XSS-then-token-theft chain) — HttpOnly cookies or in-memory storage with short-lived tokens are preferred.
- **OAuth/SSO** (if used): `redirect_uri` validated against an allow-list (not prefix-matched); `state` parameter used and verified against CSRF on the callback; tokens from provider validated (signature, issuer, audience), not blindly trusted.
- **Logout**: confirm logout actually invalidates the session/token server-side, not just clears it client-side.

## 3. AI Route Endpoint Security

For every endpoint that forwards input to an LLM or AI service:
- **Prompt injection defenses**: user input clearly delimited from system instructions (structured roles, input tagging, or a guard model). Flag string-concatenation of raw user input directly into a system prompt.
- **Jailbreak resistance**: guardrails (prompt hardening, output filtering, moderation pass) against instruction-override attempts.
- **[ADDED] System prompt / secret leakage**: confirm the model can't be induced to reveal its system prompt, internal tool definitions, or any embedded credentials/API keys via prompt injection.
- **Token bomb / resource exhaustion**: hard input length limit (char/token count) enforced server-side *before* the request reaches the model; output `max_tokens` capped. Flag any AI route with no input size ceiling.
- **Rate limiting on AI routes**: per-user/per-IP limits and/or quota system separate from general API limits.
- **[ADDED] Denial-of-wallet**: beyond simple rate limits, is there a spend cap/budget alert per user or globally, given AI calls (and other metered cloud services — storage, third-party APIs) have real per-request cost that a rate limit alone doesn't bound over time?
- **Output handling**: model output sanitized before rendering as HTML/markdown (prevents stored XSS via LLM output) or before use in downstream function calls/code execution/shell commands.
- **Tool/function-calling risk**: if the AI can call tools/functions, confirm those tools enforce their own authz — the model choosing to call a tool isn't itself authorization.

## 4. General API Authorization & Authentication

For every route in the API surface:
- Classify each endpoint as **public** or **non-public**.
- For every non-public endpoint, confirm:
  - **Authentication**: valid identity check occurs server-side before any logic runs.
  - **Authorization / BOLA / IDOR**: the authenticated identity is actually permitted to access *that specific resource* (ownership/role checks), not just "logged in." Flag `/api/orders/:id`-style routes with no check that `:id` belongs to the requester.
  - **Mass assignment**: do update/create endpoints blindly accept and persist whatever fields are in the request body (e.g. a user PATCH setting `role: "admin"`)? Confirm an explicit allow-list of updatable fields.
  - Middleware/guard applied consistently — check for routes that forgot to attach auth middleware.
- Flag any admin-only or internal routes reachable without role checks.
- **[ADDED] Enumeration / scraping limits**: list endpoints enforce pagination with a sane max page size, and don't allow a single request (or trivially-scripted loop) to dump an entire user/resource table.
- **[ADDED] Stale debug/internal routes**: confirm feature-flagged or "temporary" debug endpoints from development aren't still reachable in production (check route tables against what's actually documented/intended).
- **API versioning**: are old/deprecated API versions still live and less protected than the current one?
- **HTTP verb tampering**: does changing the HTTP method on a protected route (`POST` → `PUT`/`DELETE`/`TRACE`) bypass auth middleware bound only to specific verbs?

## 5. Input Validation, XSS & Injection (beyond SQL)

- **XSS**: user-generated content escaped/sanitized before rendering (server-rendered templates and client-side frameworks alike). Check `dangerouslySetInnerHTML`/`v-html`/`innerHTML` usage for unsanitized input.
- **Command injection**: any use of `exec`/`spawn`/`child_process`/shell-outs including unsanitized user input?
- **Path traversal**: file-serving/upload/download endpoints — is the file path built from user input without normalizing/restricting to an allowed directory?
- **[ADDED] Local/remote file inclusion**: for stacks that support dynamic file includes (PHP `include`/`require`, or template-path selection driven by user input), confirm the include path isn't user-controllable.
- **SSRF**: any endpoint that fetches a user-supplied URL (webhooks, image-proxy, "import from URL")? Confirm the target is allow-listed and internal/private IP ranges (169.254.x.x, 10.x, 127.0.0.1, cloud metadata endpoints) are blocked.
- **Open redirect**: any `redirect_to`/`next`/`returnUrl` param not validated against an allow-list of internal paths.
- **Insecure deserialization**: any unsafe deserialization (`pickle`, unguarded `eval`, unsafe YAML load) on user-controlled data.
- **File upload security**: file type validated by content/magic-bytes (not extension or client-supplied MIME type); size limits enforced; files stored outside the web root or served with safe headers (`Content-Disposition: attachment`); AV scanning for arbitrary uploads; uploaded SVGs sanitized or rendered without script execution.

## 6. CSRF, CORS & Security Headers

- **CSRF**: state-changing requests (POST/PUT/PATCH/DELETE) protected by CSRF tokens or SameSite cookies + custom header checks, especially for cookie-authenticated sessions.
- **CORS**: `Access-Control-Allow-Origin` is not `*` alongside `Access-Control-Allow-Credentials: true`; origin allow-list explicit, not reflected/echoed unchecked from the request.
- **Security headers**: `Content-Security-Policy`, `Strict-Transport-Security` (HSTS), `X-Content-Type-Options: nosniff`, `X-Frame-Options`/`frame-ancestors`, `Referrer-Policy`.
- **Subresource Integrity (SRI)**: third-party `<script>`/`<link>` tags from a CDN include `integrity` hashes.
- **TLS**: HTTPS enforced everywhere (HTTP redirected, no mixed content); certificate/hostname validation not disabled in any server-to-server call.

## 7. Database Query Security

- **SQL injection**: all queries parameterized/prepared/ORM-safe. Flag any raw string interpolation into SQL.
- **NoSQL injection**: for MongoDB/similar, confirm query operators (`$where`, `$gt`) can't be injected via unsanitized user input in query objects.
- **[ADDED] Query timeouts / statement limits**: long-running or unbounded queries (e.g. a search endpoint with no result cap, or a report query with no timeout) can't be used to exhaust DB resources as a DoS vector.
- **Supabase / RLS-specific checks** (if applicable):
  - Row Level Security **enabled** on every table containing user data (flag tables with RLS disabled — the most common critical Supabase misconfig).
  - Each RLS policy correctly scopes `SELECT`/`INSERT`/`UPDATE`/`DELETE` to `auth.uid()` or the appropriate ownership column; flag overly permissive policies (`USING (true)`) on sensitive tables.
  - `service_role` key never exposed client-side (it bypasses RLS entirely).
  - Policies applied to all four operations, not just SELECT.
- **Multi-tenancy isolation**: in shared-database multi-tenant setups, confirm every query is scoped by `tenant_id`/`org_id` and this can't be omitted or spoofed by the client.
- **Least privilege DB accounts**: the app's DB user doesn't have superuser/admin rights it doesn't need.

## 8. Secrets, Dependencies & Supply Chain

- **Hardcoded secrets**: search for API keys, DB credentials, private keys committed in source, config, or git history.
- **.env handling**: `.env` files git-ignored; no secrets exposed to the client bundle (check server-only env vars accidentally used client-side, e.g. `NEXT_PUBLIC_`/`VITE_`-prefixed vars leaking sensitive keys).
- **Dependency vulnerabilities**: report on `npm audit`/`pip-audit`/equivalent; flag known-CVE packages and abandoned/unmaintained dependencies.
- **Dependency confusion / typosquatting risk**: internal package names that could collide with a public registry package.
- **[ADDED] SBOM / provenance**: is there any software bill of materials or lockfile-integrity check (`npm ci` with committed lockfile, `pip-compile` hashes) preventing a silent dependency swap?
- **Third-party API keys**: any payment, storage, or AI provider key exposed in client-side code instead of proxied through a backend.
- **Cloud storage permissions**: S3/GCS/Supabase Storage buckets aren't publicly listable/writable when they shouldn't be.
- **[ADDED] Pre-signed URL scope/expiry**: if the app issues pre-signed upload/download URLs, confirm they're scoped to a single object/action and expire quickly, rather than being long-lived or bucket-wide.
- **Secrets rotation**: is there any process/capability to rotate leaked or expiring credentials without a full redeploy?

## 9. Logging, Error Handling & Monitoring

- **Sensitive data in logs**: passwords, tokens, full credit card numbers, or PII logged in plaintext.
- **Stack trace leakage**: production error responses don't leak stack traces, internal file paths, or DB error details.
- **Debug mode**: frameworks aren't running debug/verbose mode in production (Django `DEBUG=True`, Express `NODE_ENV` unset).
- **Audit trail**: security-relevant events (login, password change, role change, admin actions) logged with enough context to investigate later.
- **Alerting**: any mechanism to detect anomalous spikes in failed logins, 429s, or 5xxs?

## 10. Business Logic & Race Conditions

- **Race conditions**: payment, coupon/discount redemption, inventory decrement, "claim this offer" endpoints — can concurrent requests double-spend or bypass a one-time-use check (TOCTOU)? Look for missing DB-level locking/transactions around check-then-act logic.
- **Price/quantity tampering**: are price, discount, or quantity values trusted from the client anywhere instead of recalculated server-side?
- **Webhook security**: incoming webhooks (Stripe, etc.) verify a signature before trusting the payload; endpoints aren't reachable to forge fake events.
- **[ADDED] Replay protection beyond webhooks**: for any signed/HMAC-authenticated request (not just webhooks — internal service-to-service calls, signed URLs), confirm a nonce and/or timestamp window prevents a captured valid request from being replayed later.

## 11. Real-Time & Alternative Protocols (if applicable)

- **WebSockets**: connection/auth handshake requires a valid token, not just an open unauthenticated socket; per-message authorization re-checked, not assumed from the initial handshake.
- **GraphQL** (if used): introspection disabled in production; query depth/complexity limiting in place; field-level authorization enforced (not just at the resolver's top level).
- **[ADDED] GraphQL batching/alias-based DoS**: a single request using many aliases of the same expensive field, or a batched array of queries, can't multiply cost past what rate limiting accounts for — confirm query cost analysis or a hard alias/batch limit.
- **[ADDED] Persisted queries / allow-listing**: for production GraphQL APIs, is arbitrary ad-hoc querying still open, or are only pre-approved persisted queries accepted (reduces both injection surface and cost-DoS risk)?

## 12. Infrastructure & Deployment (if in scope)

- **Container security**: Dockerfiles don't run as root unnecessarily; no secrets baked into image layers; base images pinned and reasonably current.
- **Environment separation**: dev/staging/prod isolated (separate credentials, no prod data in lower environments) — maps to ISO A.8.31.
- **Default credentials**: no default/sample admin credentials left active in any environment.
- **[ADDED] Cloud IAM least privilege**: service accounts/roles (AWS IAM, GCP service accounts, k8s service accounts) are scoped to only the resources/actions they need — flag wildcard (`*:*`) permissions or a single shared "god" role used across services.
- **[ADDED] Network segmentation**: internal services (databases, admin panels, internal APIs) aren't reachable from the public internet — confirm via security group/firewall rules or network policy, not just application-layer auth.
- **[ADDED] Secrets manager usage**: production secrets are pulled from a secrets manager/vault at runtime rather than baked into environment variables in a way that's visible in deployment configs or CI logs.

## 13. Advanced Injection & Parsing Vulnerabilities

- **Server-Side Template Injection (SSTI)**: user input passed into a template engine's render function (Jinja2, EJS, Handlebars) without being treated strictly as data — can lead to RCE.
- **XXE**: if any XML parsing occurs, confirm external entity resolution and DTD processing are disabled.
- **Prototype pollution** (JS/Node): any recursive merge/deep-clone/`Object.assign` of user-controlled JSON that could pollute `__proto__`/`constructor.prototype`.
- **ReDoS**: any regex applied to user input with catastrophic-backtracking patterns (nested quantifiers like `(a+)+`) that could hang the process on crafted input.
- **Zip bomb / decompression bomb**: file/archive upload or decompression endpoints enforce a max decompressed size and depth, not just max compressed size.
- **Zip slip**: archive extraction validates extracted file paths can't escape the target directory via `../` entries.
- **JSON/body bomb**: request body size limits and JSON parsing depth limits enforced.

## 14. Cryptography & Randomness

- **Insecure randomness**: tokens, password-reset codes, API keys, and session IDs generated with a cryptographically secure RNG (`crypto.randomBytes`, `secrets` module) — not `Math.random()` or similar.
- **Timing attacks**: sensitive comparisons (password hashes, API keys, HMAC signatures) use constant-time comparison, not `===`/`==`.
- **Encryption at rest**: highly sensitive fields (SSNs, payment details, health data) encrypted at the field/column level, not just relying on disk-level encryption.
- **Key management**: encryption keys aren't stored alongside the data they encrypt.

## 15. Data Privacy & Compliance

- **PII inventory**: is there clarity on what personal data is collected and where it's stored?
- **Data retention**: mechanism to delete/anonymize user data on account deletion (right to erasure), or does data persist indefinitely?
- **Backup security**: backups encrypted and access-controlled; restore access itself gated behind auth/authz.
- **Third-party data sharing**: analytics/marketing/AI providers receiving more user data than necessary (no data minimization).

## 16. Infrastructure Exposure & Misconfiguration

- **Exposed internal files**: `.git/`, `.env`, `.DS_Store`, backup files (`.bak`, `.old`), or config files aren't reachable over HTTP.
- **Exposed API documentation**: Swagger/OpenAPI UI, GraphQL playground, or admin panels aren't publicly reachable without auth in production (or don't allow "try it out" execution against prod without auth).
- **Health/status endpoints**: don't leak internal architecture, versions, stack traces, or environment variables.
- **Subdomain takeover**: no dangling DNS records (CNAMEs) pointing at deprovisioned third-party services.
- **HTTP request smuggling**: if behind a reverse proxy/load balancer, confirm consistent `Content-Length`/`Transfer-Encoding` handling between front-end and back-end.
- **Cache poisoning**: CDN/reverse-proxy caching not applied to authenticated/personalized responses (check `Cache-Control`/`Vary` on user-specific endpoints).
- **Email header injection**: contact-form/email-sending endpoints sanitize input used in email headers (To/From/Subject).

## 17. Client-Side Trust Boundary & Rendering Leaks

- Confirm all security-relevant validation and business logic enforced only in client-side JavaScript (price calculation, permission checks, form validation) is **also** enforced server-side. Client-side checks are UX only, never a security control.
- Sensitive data isn't sent to the client "just in case it's needed later" — over-fetching API responses that include fields the current user shouldn't see, even if the UI doesn't render them.
- **[ADDED] SSR hydration payload leakage**: for server-rendered frameworks (Next.js, Nuxt, SvelteKit, Remix), inspect the embedded state blob shipped to the client for hydration (e.g. Next.js `__NEXT_DATA__`, Nuxt's `__NUXT__`, Remix's loader data). These commonly serialize the *entire* server-side data object, not just what the UI renders — confirm no internal IDs, other users' data, unpublished content, or permission flags are present in that blob even when the visible UI doesn't display them, since anyone can view-source it.

## 18. Memory Leak Review

- Unclosed resources: DB connections/pools not released, file handles left open, event listeners added without removal (especially long-lived Node.js processes or React `useEffect` without cleanup).
- Unbounded in-memory caches, arrays, or maps that grow without eviction (e.g. caching every request/response with no TTL or size cap).
- Closures retaining large objects longer than needed; recursive/streaming handlers that don't release buffers.
