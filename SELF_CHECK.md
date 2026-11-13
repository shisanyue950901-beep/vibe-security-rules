# The 10-Minute Pre-Launch Self-Check

For AI-built apps on Supabase / Firebase-style backends. Eight quick tests you
can run on **your own app**, using only the public `anon` key your frontend
already ships — no tools to install, nothing to attack. Each test takes about
a minute. If any test comes back "bad", fix that one before you take payments.

You only need: your app's URL, your Supabase project URL, and the `anon` key
(both are visible in your frontend's network tab or your Supabase client
setup — they are public by design).

---

## 1. Can a stranger read your tables with the public key?

Pick a table that holds real user data (orders, profiles, messages). Call the
REST endpoint with the anon key and **no login**:

```bash
curl "https://<project>.supabase.co/rest/v1/orders?select=*" \
  -H "apikey: <ANON_KEY>" -H "Authorization: Bearer <ANON_KEY>"
```

- ✅ **Good:** `[]`, or a `401/403` — anonymous callers get nothing.
- ❌ **Bad:** real rows come back. Anyone who opens your site can pull the
  same data.
- **Fix:** enable RLS on the table and add owner-scoped policies
  (`auth.uid() = user_id`) instead of `USING (true)`.

## 2. Is the service-role key anywhere near the browser?

Search your repo and your built frontend for `service_role`:

```bash
grep -r "service_role" --include="*.ts" --include="*.tsx" . 
# and check for it behind NEXT_PUBLIC_ in your env files
grep -r "NEXT_PUBLIC_.*SERVICE\|NEXT_PUBLIC_.*SECRET\|NEXT_PUBLIC_.*PRIVATE" .
```

- ✅ **Good:** the service-role key appears only in server-only code and in a
  non-public env var.
- ❌ **Bad:** it shows up in a client component, or behind any `NEXT_PUBLIC_`
  variable — every visitor can extract it and bypass all your RLS policies.
- **Fix:** move the key to a server-only env var, access it only from route
  handlers / server actions, and **rotate the key** — it must be treated as
  leaked.

## 3. Can anyone list your storage buckets?

```bash
curl "https://<project>.supabase.co/storage/v1/object/list/<bucket>" \
  -H "apikey: <ANON_KEY>" -H "Authorization: Bearer <ANON_KEY>" \
  -H "Content-Type: application/json" -d '{"prefix":""}'
```

- ✅ **Good:** empty result or an authorization error for buckets holding
  user files.
- ❌ **Bad:** file names of user uploads, invoices, or exports come back —
  they're downloadable too.
- **Fix:** set the bucket to private and serve files through signed URLs with
  owner-scoped storage policies.

## 4. Did any real secret ever land in the repo?

Look for real-format keys in the files that usually get committed by accident
(`.env.example`, fixtures, docs, example configs):

```bash
grep -rE "sk-[A-Za-z0-9_-]{20,}|service_role|STRIPE_SECRET|sk_live_" --include="*.example" --include="*.md" .
```

- ✅ **Good:** only placeholders like `sk-...` or `your-key-here`.
- ❌ **Bad:** a value that looks real. `.env.example` with a working key in it
  is a common accident — the author pasted the real file and renamed it.
- **Fix:** replace the value with a placeholder **and rotate the key** —
  deleting it from the file isn't enough once it's in git history.

## 5. Does your app trust the browser to say who is an admin?

Check your API routes for authorization that reads identity or role from the
request itself — a header, a body field, localStorage:

```bash
grep -rn "x-admin\|isAdmin\|role.*req\.\|req\.json.*userId" app/ src/ 2>/dev/null
```

- ✅ **Good:** routes resolve the session server-side and read the role from
  the database.
- ❌ **Bad:** a route grants access because the request *says* so. Anyone can
  add that header or field with curl.
- **Fix:** derive identity from the session and role from your own tables;
  ignore identity/role/price fields sent by the client.

## 6. Is one user able to open another user's resource by changing an ID?

Log in as user A, open one of A's items (an order, a document), then change
the ID in the URL or request to one belonging to user B.

- ✅ **Good:** `404` (or `403`) — B's item is invisible to A.
- ❌ **Bad:** B's data loads. This is the classic broken-access-control hole
  in AI-generated CRUD code.
- **Fix:** every route must check that the resource's owner matches the
  session user before reading or writing it — and your RLS policies should
  enforce the same rule at the database layer.

## 7. Can your AI endpoints be run in a loop?

Find your routes that call a paid API (chat, image, TTS, embeddings) and ask
three questions: does the route know **who** is calling? Is there a **rate
limit**? Is there a **daily cap** per user?

- ✅ **Good:** all three answers are yes, and oversized inputs are rejected.
- ❌ **Bad:** an unauthenticated route with no caps — one script can spend
  your provider balance while you sleep.
- **Fix:** require auth, add a per-user rate limit and a daily usage ceiling
  that is checked *before* the provider call, and cap input size.

## 8. Do your webhooks check signatures?

Open your Stripe (or other payment) webhook handler and read the first lines.

- ✅ **Good:** it builds the event with the provider's signing secret
  (`stripe.webhooks.constructEvent` or the provider's equivalent) and
  rejects the request when verification fails.
- ❌ **Bad:** it parses the JSON body and acts on `payment_status` or similar
  fields directly — anyone can POST a fake "paid" event and get your product
  for free.
- **Fix:** verify the signature against the **raw** request body before any
  business logic runs.

---

## If something came back "bad"

Fix order: **2 and 4 first** (rotate the exposed keys — everything else is
moot while a live key is public), then **1 and 6** (who can read whose data),
then the rest.

The rules pack in this repo (`cursor-rules.mdc` / `CLAUDE.md`) stops your AI
assistant from writing these problems into *new* code. For the state your
project is already in — dashboard-side policies, keys already in git history
— a one-time human pass is the reliable way to catch what's left. That's what
the [Go-Live Security & Cost-Cap Check](https://shisanyue950901-beep.github.io/security-checkup/)
does, and you can read a [full sample report](https://shisanyue950901-beep.github.io/security-checkup/sample-report.html)
before deciding whether it's worth it for your launch.
