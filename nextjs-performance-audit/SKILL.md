---
name: nextjs-performance-audit
description: Audit Next.js code (App Router or Pages Router) for performance problems tied to rendering strategy (SSR vs CSR vs static), server/client component boundaries, data-fetching waterfalls, and library/bundle bloat. Use this whenever the user shares Next.js components, pages, layouts, or a whole repo and asks about performance, "why is this page slow", slow TTFB/LCP, hydration cost, bundle size, too many client components, or wants a review of how data is fetched. Trigger especially when code depends on multiple independent async data sources (several `await`s, multiple `fetch`/DB calls, chained `useEffect`s, several SWR/React Query/Apollo hooks) — detecting and safely fixing these fetch waterfalls is this skill's specialty. Also trigger for general "review this Next.js code for performance" or "audit my Next.js app" requests. Produces a structured findings report first, then offers to apply the refactor.
---

# Next.js Performance Audit

Next.js performance problems usually come from a handful of recurring structural mistakes, not from any single slow line of code: independent data fetches that get serialized instead of parallelized, client/server component boundaries drawn in the wrong place, and libraries pulled into bundles that didn't need them. This skill walks through those categories systematically, and pays special attention to **async data-fetching waterfalls**, since they're the most common and most fixable source of slow pages.

The goal of every fix this skill proposes is to speed things up **without** making the UI worse (no new layout shift, no lost loading/error states, no regressed perceived performance) and **without** weakening security (no secrets or auth-gated logic moving to the client, no bypassing of auth checks to enable parallelization). Read `references/guardrails.md` before proposing any fix — it's short, and it's the thing most likely to save you from a fix that "works" but is worse than the original.

## Step 1: Figure out the scope

- **Single file/component pasted or uploaded** → audit that file plus its immediate data-fetching context (parent that renders it, any hooks/lib files it imports).
- **Whole repo/project** → first orient: check `package.json` for `next` version and data-fetching libraries (SWR, React Query/TanStack Query, Apollo, tRPC), and check whether `app/` (App Router) or only `pages/` (Pages Router) exists — the waterfall patterns and fixes differ between the two. Then prioritize: routes/pages with the most nested async dependencies or the most `'use client'` directives near the top of the tree are the highest-value targets. Don't try to deeply audit every file in a large repo in one pass — sample the highest-traffic or most complained-about routes first, say so, and offer to go deeper.

## Step 2: Detect issues by category

Read `references/patterns.md` for the full set of before/after code patterns. The categories, roughly in order of typical impact:

1. **Data-fetching waterfalls** (the specialty of this skill — see Step 3 below for extra depth)
2. **Rendering strategy mismatch** — using a Client Component (or `useEffect` fetch) for something that could be a Server Component fetching directly, forcing an extra client-server round trip
3. **Client/server boundary placement** — `'use client'` declared too high in the tree, dragging static or server-fetchable content into the client bundle along with the interactive leaf that actually needed it
4. **Missing streaming/Suspense** — a whole route blocked on its slowest data dependency instead of using `loading.tsx` / `<Suspense>` to stream in independent sections as they resolve
5. **Bundle bloat** — heavy libraries imported eagerly (charting, rich text editors, date libraries with huge locale sets, moment.js, unoptimized icon packs) where `next/dynamic` or a lighter alternative would help
6. **Duplicate/uncached fetches** — the same data fetched separately in a layout, a page, and `generateMetadata` because no request-level memoization (`cache()`, `unstable_cache`, or `fetch`'s built-in dedup) is in use

For each finding, note: file/line, why it matters (what it costs — extra round trip, blocked TTFB, blocked LCP, larger client bundle), and which pattern in `references/patterns.md` fixes it.

## Step 3: Waterfall analysis (the core skill)

When you see multiple independent async data dependencies, work out whether they're actually dependent or just written sequentially out of habit:

- List every `await`, `fetch`, data hook, or DB call in the component/route.
- For each pair, ask: **does B actually need a value produced by A, or was it just written after A?** Only true data dependencies (B needs A's output as an input) justify sequential execution. If B and A both start from props, params, or the session/user context, they're independent and are candidates for parallelization.
- In Server Components / route handlers: independent awaits should become `Promise.all` (or `Promise.allSettled` if one failing shouldn't block the others), or if they cross separate parts of the UI, separate `<Suspense>` boundaries so slow parts stream in without blocking fast ones.
- In Client Components: chained `useEffect`s that each set state and trigger the next fetch are a classic waterfall — check whether they can become parallel `useEffect`s, parallel data-hook calls (SWR/React Query support this natively), or whether the data belongs in a Server Component instead so it isn't fetched from the client at all.
- Check for accidental duplicate fetches (same URL/query fetched once for `generateMetadata` and again for the page body) — this looks like "more data" but is actually redundant work; the fix is memoization, not parallelization.
- Watch for a subtle trap: don't parallelize a fetch that exists specifically to gate the others (e.g., an auth/permission check that later calls depend on for their query parameters, or that determines whether the later calls should happen at all). Flag these explicitly as "must stay sequential" so a future editor doesn't undo the gate.

## Step 4: Write the report

Use this structure:

```
## Next.js Performance Audit — <scope>

### Summary
<1-3 sentences: overall picture, biggest win available>

### Findings

#### [severity: High/Medium/Low] <short title>
- **Where:** <file:line or component name>
- **Category:** <one of the Step 2 categories>
- **Impact:** <what it costs the user — e.g. "adds ~400ms sequential to TTFB", "ships a 90kb library to every visitor for one modal">
- **Fix:** <pattern name from references/patterns.md, 2-4 lines of code or a short diff sketch>
- **UI/UX check:** <confirms the fix doesn't remove a loading/error state or change perceived load order for the worse>
- **Security check:** <confirms the fix doesn't move secrets/auth logic client-side or skip a required gate — write "n/a" if genuinely not applicable, but check for every finding involving fetch reordering>

<repeat per finding, highest severity first>
```

End the report by asking whether they'd like the refactor applied — don't apply it unprompted, since some fixes (especially reshuffling Suspense boundaries) change loading UX and the person may want to see the plan before it touches their code.

## Step 5: Apply the refactor (only once requested)

- Apply changes with the normal file-editing tools, one finding at a time if the person wants to review incrementally, or all at once if they say "just do it."
- After editing, re-state which findings were fixed and re-check each one against the UI/UX and security guardrails in `references/guardrails.md` — a rushed edit is exactly where those get violated.
- If a fix is genuinely risky (e.g., the only way to parallelize also removes a security gate), don't apply it — explain the tradeoff and propose the safer partial fix instead.

## Reference files

- `references/patterns.md` — before/after code for every issue category (waterfalls, boundary placement, streaming, bundle bloat, dedup/caching), for both App Router and Pages Router where they differ.
- `references/guardrails.md` — short checklist to run every proposed fix against before including it in the report or applying it.
