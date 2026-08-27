# Guardrails: run every fix through this before reporting or applying it

Performance fixes are cheap to justify and easy to get wrong in ways that only show up later. Check every proposed fix against both lists below.

## UI/UX guardrails

- **No new layout shift.** If you add or remove a `<Suspense>` boundary, it needs a real skeleton/fallback that reserves the same space as the eventual content — not `null`, not a fallback with different dimensions.
- **Don't lose an existing loading or error state.** If the original code had a spinner, error boundary, or empty state, the refactored version needs an equivalent. Parallelizing fetches with `Promise.all` means one rejection fails all of them by default — check whether the original handled partial failure (e.g., showed posts even if comments failed) and use `Promise.allSettled` if so, rather than silently making a partial-failure UI into an all-or-nothing one.
- **Don't change perceived load order for the worse.** Streaming fast content in before slow content is usually a win, but if the design intentionally shows things in a specific order (e.g., a summary that should appear after its supporting data so numbers don't jump), don't reorder it purely for a parallelization win — ask or flag it instead of assuming.
- **Don't over-apply `next/dynamic` with `ssr: false`.** This trades a smaller initial bundle for a blank space until the component mounts client-side. Fine for below-the-fold or interaction-gated content; a bad trade for anything visible on first paint.

## Security guardrails

- **Never move a secret to the client to enable parallelization.** If a "fix" would require an API key, DB credential, or internal service URL to be reachable from client code, it's not a valid fix — keep that call server-side (Server Component, Route Handler, or Server Action) even if it means the data isn't as easy to parallelize with client-fetched data.
- **Never remove or reorder an auth/permission gate.** If one await exists specifically to check that the user is allowed to see the data the other awaits would fetch, it must stay sequential (gate first, data after) even though this looks like an "unnecessary" waterfall. Flag it explicitly in the report as intentional so it doesn't get "fixed" later by someone who only reads the code, not this audit.
- **Don't widen what a Client Component receives just to save a fetch.** Passing a whole object (e.g., a full `user` record with fields the client never renders) down as props to avoid a second fetch can leak fields that shouldn't reach the browser. Pass only what the client component actually uses.
- **Caching (`cache()`, `unstable_cache`, `fetch` dedup) must respect per-user/per-request scope.** Don't recommend a shared cache key for data that's actually user-specific (e.g. caching a "current user" lookup keyed only by a static string) — that can leak one user's data to another's request in some deployment topologies. Keep cache keys scoped to the actual identity/params the data depends on.
