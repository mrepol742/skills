# Next.js Performance Patterns

Concrete before/after examples for each finding category. These are illustrative shapes, not the only correct answer — adapt to the actual code.

## 1. Data-fetching waterfalls

### 1a. Sequential independent awaits in a Server Component

**Before (waterfall — user, posts, and comments have no dependency on each other):**
```tsx
async function Page({ params }: { params: { userId: string } }) {
  const user = await getUser(params.userId);
  const posts = await getPosts(params.userId);
  const comments = await getRecentComments(params.userId);
  return <Profile user={user} posts={posts} comments={comments} />;
}
```

**After — kick off together, await together:**
```tsx
async function Page({ params }: { params: { userId: string } }) {
  const [user, posts, comments] = await Promise.all([
    getUser(params.userId),
    getPosts(params.userId),
    getRecentComments(params.userId),
  ]);
  return <Profile user={user} posts={posts} comments={comments} />;
}
```
If one of the three is much slower and the others could render immediately, prefer splitting into separate `<Suspense>`-wrapped child components over one big `Promise.all`, so fast data isn't held hostage by the slow one (see 1c).

### 1b. Chained `useEffect`s in a Client Component

**Before:**
```tsx
useEffect(() => {
  fetchUser(id).then(setUser);
}, [id]);

useEffect(() => {
  if (user) fetchOrgSettings(user.orgId).then(setSettings); // waits on user unnecessarily if orgId is already known
}, [user]);
```
If `orgId` is actually available from `id` or route params without waiting for `user`, this is a false dependency written out of habit.

**After (if truly independent):**
```tsx
useEffect(() => {
  fetchUser(id).then(setUser);
  fetchOrgSettings(orgIdFromParams).then(setSettings);
}, [id, orgIdFromParams]);
```
**Better still**, if this is in the App Router: move this fetch to the Server Component that renders this client island, and pass the data down as props, or fetch it in a parent Server Component and stream the client part in with `<Suspense>`. Client-side fetching should be reserved for data that genuinely depends on client-only state (e.g. user interaction after mount), not for data available at request time.

### 1c. Split Suspense boundaries so slow data doesn't block fast data

**Before (one boundary — the whole page waits for the slowest fetch):**
```tsx
async function Page() {
  const [fast, slow] = await Promise.all([getFastStat(), getSlowRecommendations()]);
  return <>
    <FastStat data={fast} />
    <Recommendations data={slow} />
  </>;
}
```

**After (independent streaming — fast content paints immediately):**
```tsx
function Page() {
  return <>
    <Suspense fallback={<FastStatSkeleton />}>
      <FastStatSection />
    </Suspense>
    <Suspense fallback={<RecommendationsSkeleton />}>
      <RecommendationsSection />
    </Suspense>
  </>;
}
async function FastStatSection() { return <FastStat data={await getFastStat()} />; }
async function RecommendationsSection() { return <Recommendations data={await getSlowRecommendations()} />; }
```
UI/UX check: this requires a real skeleton/fallback for each boundary — don't ship `<Suspense fallback={null}>` for content that has meaningful layout, since that reintroduces layout shift when the content pops in.

### 1d. Deduplicating repeated fetches (looks like a waterfall fix but is really a caching fix)

If the same data is fetched once in `generateMetadata` and again in the page body, don't try to "parallelize" it — dedupe it:
```tsx
import { cache } from 'react';
const getProduct = cache(async (id: string) => { /* fetch */ });
// now safe to call getProduct(id) from both generateMetadata and the page —
// React dedupes calls with the same arguments within one request
```
Native `fetch` in Next.js already dedupes identical requests within a render pass; wrap non-fetch data access (direct DB calls, ORM queries) in `cache()` to get the same benefit.

## 2. Rendering strategy mismatch

**Before (Client Component fetching data that's available at request time):**
```tsx
'use client';
function ProductPage({ id }: { id: string }) {
  const [product, setProduct] = useState(null);
  useEffect(() => { fetchProduct(id).then(setProduct); }, [id]);
  if (!product) return <Spinner />;
  return <ProductView product={product} />;
}
```
This forces: render empty shell → hydrate → fetch → render again. Three round trips of latency for data the server already had.

**After:**
```tsx
async function ProductPage({ params }: { params: { id: string } }) {
  const product = await fetchProduct(params.id);
  return <ProductView product={product} />;
}
```
Only keep the client fetch if the data genuinely depends on client-only state (e.g., geolocation, a user action, a value from localStorage).

## 3. Client/server boundary placement

**Before (`'use client'` at the top drags the whole tree, including static content, into the client bundle):**
```tsx
'use client';
export default function Page({ data }) {
  return (
    <div>
      <StaticHeader />       {/* doesn't need to be client */}
      <StaticArticleBody />  {/* doesn't need to be client */}
      <LikeButton />         {/* the only actually-interactive part */}
    </div>
  );
}
```

**After (push the boundary down to the leaf that needs interactivity):**
```tsx
export default function Page({ data }) {
  return (
    <div>
      <StaticHeader />
      <StaticArticleBody />
      <LikeButton /> {/* 'use client' lives inside LikeButton.tsx only */}
    </div>
  );
}
```
Smaller client bundle, less hydration work, and the static parts can stream/cache as server-rendered HTML.

## 4. Bundle bloat

- A heavy component only needed after user interaction (modal, chart, rich text editor) → wrap with `next/dynamic`:
```tsx
const ChartModal = dynamic(() => import('./ChartModal'), { ssr: false, loading: () => <Spinner /> });
```
Only use `ssr: false` when the component is genuinely client-only (e.g. depends on `window`) — don't reach for it by default, since it delays first paint of that section.
- Whole-library imports of icon packs or date libraries (`import * as Icons from 'a-huge-icon-lib'`, `import moment from 'moment'`) → check for tree-shakeable per-icon imports or lighter alternatives (`date-fns`, `dayjs`) with the same behavior.
- Check `next.config.js` for `experimental.optimizePackageImports` support for the libraries in use — it can shrink bundles for known-heavy packages without code changes.

## 5. Pages Router specifics

If the project uses `pages/` instead of `app/`:
- `getServerSideProps` waterfalls follow the same "parallelize independent awaits with `Promise.all`" fix as Server Components.
- There's no native `<Suspense>` streaming for data the way App Router has it — splitting slow sections usually means moving them to client-side fetches with SWR/React Query *after* first paint, with a visible loading state, rather than blocking `getServerSideProps` on everything. This is a real exception to "prefer server fetching" — explain the tradeoff (extra client round trip) rather than silently applying it.
- SWR/React Query: independent queries should be separate hook calls (they parallelize automatically); a dependent query should use the `enabled`/conditional-key pattern rather than nesting inside a `.then()` or effect.
