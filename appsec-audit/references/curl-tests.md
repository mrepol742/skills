# Verification curl Commands

Every finding needs a runnable command that proves the issue (or, after a fix, verifies it's resolved). Adapt these to the codebase's actual routes, params, and auth scheme — don't paste placeholders verbatim into the report. Each command gets a one-line comment above it explaining what it proves.

## Auth & session (Section 1–2)

```bash
# Hammer the login endpoint to confirm rate limiting triggers a 429
for i in $(seq 1 20); do curl -s -o /dev/null -w "%{http_code}\n" -X POST https://target/api/login \
  -d '{"email":"test@example.com","password":"wrong"}' -H "Content-Type: application/json"; done

# Submit a disposable email to confirm registration rejects it
curl -X POST https://target/api/register -d '{"email":"test@mailinator.com","password":"..."}' -H "Content-Type: application/json"

# Submit the honeypot field filled in to confirm the form soft-fails
curl -X POST https://target/api/register -d '{"email":"a@b.com","password":"...","website":"filled-in-by-bot"}' -H "Content-Type: application/json"

# [ADDED] Confirm account lockout triggers after N failed attempts on the SAME account from DIFFERENT IPs
for i in $(seq 1 10); do curl -s -X POST https://target/api/login \
  -H "X-Forwarded-For: 10.0.0.$i" -d '{"email":"victim@example.com","password":"wrong"}'; done

# [ADDED] Confirm sensitive account changes require re-authentication, not just an active session
curl -X POST https://target/api/account/email -H "Authorization: Bearer $STALE_SESSION_TOKEN" \
  -d '{"newEmail":"attacker@evil.com"}' # should fail without a fresh password/MFA challenge
```

## Session fixation & JWT (Section 2)

```bash
# [ADDED] Confirm session ID changes after login (capture cookie before and after auth)
curl -c pre_login.txt https://target/
curl -b pre_login.txt -c post_login.txt -X POST https://target/api/login -d '{"email":"a@b.com","password":"correct"}'
diff <(grep session pre_login.txt) <(grep session post_login.txt) # should differ

# [ADDED] Confirm the server rejects an unsigned JWT (alg: none)
curl https://target/api/me -H "Authorization: Bearer eyJhbGciOiJub25lIn0.eyJzdWIiOiJhZG1pbiJ9."
```

## AI routes (Section 3)

```bash
# Send an oversized payload to an AI route to confirm it's rejected before hitting the model
curl -X POST https://target/api/ai/chat -d "{\"message\":\"$(python3 -c 'print("a"*500000)')\"}" -H "Content-Type: application/json"
```

## General authz/authn (Section 4)

```bash
# Call a non-public endpoint with no/invalid token to confirm 401/403
curl -i https://target/api/admin/users

# Sending a PATCH with an extra role field to confirm mass-assignment is blocked
curl -X PATCH https://target/api/users/me -H "Authorization: Bearer $TOKEN" \
  -d '{"name":"Alice","role":"admin"}' -H "Content-Type: application/json"

# Sending a TRACE/PUT to a route only guarded on POST to confirm verb tampering doesn't bypass auth
curl -X TRACE https://target/api/admin/settings
curl -X PUT https://target/api/admin/settings -d '{}'

# [ADDED] Request a large page size / no pagination bound to confirm enumeration limits exist
curl "https://target/api/users?limit=100000" -H "Authorization: Bearer $TOKEN"
```

## Injection & SSRF (Sections 5, 13)

```bash
# Attempt a classic SQLi payload against a query param to confirm it's neutralized
curl "https://target/api/search?q=' OR '1'='1"

# Requesting a URL pointing at cloud metadata via an "import from URL" feature to confirm SSRF protection
curl -X POST https://target/api/import -d '{"url":"http://169.254.169.254/latest/meta-data/"}' -H "Content-Type: application/json"

# [ADDED] Confirm a ReDoS-prone regex endpoint doesn't hang on a crafted string
time curl "https://target/api/validate?input=$(python3 -c 'print("a"*40+"!")')"
```

## CSRF/CORS (Section 6)

```bash
# Sending a request with a spoofed Origin header to confirm CORS rejects it
curl -i https://target/api/data -H "Origin: https://evil.com"
```

## Webhooks & replay (Section 10)

```bash
# Replaying a webhook payload with an invalid/missing signature to confirm it's rejected
curl -X POST https://target/api/webhooks/stripe -d '{"type":"payment_intent.succeeded", ...}' \
  -H "Content-Type: application/json" # no Stripe-Signature header — should be rejected

# [ADDED] Replay a previously-valid signed request after its timestamp window to confirm replay protection
curl -X POST https://target/api/internal/action -H "X-Signature: $OLD_VALID_SIGNATURE" -H "X-Timestamp: $OLD_TIMESTAMP" -d '{}'
```

## GraphQL (Section 11)

```bash
# [ADDED] Confirm introspection is disabled in production
curl -X POST https://target/graphql -H "Content-Type: application/json" \
  -d '{"query":"{ __schema { types { name } } }"}'

# [ADDED] Confirm alias-based batching can't multiply cost past rate limits
curl -X POST https://target/graphql -H "Content-Type: application/json" \
  -d '{"query":"{ a1: expensiveField a2: expensiveField a3: expensiveField ... a50: expensiveField }"}'
```

## Infra exposure (Section 16)

```bash
# Requesting /.git/config or /.env directly to confirm it 404s
curl -i https://target/.git/config
curl -i https://target/.env

# Uploading a deeply nested/large JSON body to confirm size/depth limits reject it
curl -X POST https://target/api/data -H "Content-Type: application/json" \
  -d "$(python3 -c 'print("{\"a\":"*10000 + "1" + "}"*10000)')"
```

## SSR hydration leakage (Section 17)

```bash
# [ADDED] Fetch a server-rendered page as an unauthenticated/low-privilege user and inspect the embedded state blob
curl -s https://target/dashboard | grep -o '<script id="__NEXT_DATA__"[^<]*</script>'
# then manually inspect the JSON for fields the current user shouldn't see
```
