# Vibe Security Rules — for Next.js + Supabase + Stripe

A small, opinionated set of security rules for AI coding assistants, written
for apps built with tools like Cursor, Windsurf, Claude Code, Bolt, or
Lovable on the stack most of those apps share: **Next.js + Supabase + Stripe**.

AI assistants write working code fast. They also, left to themselves, write
policies like `USING (true)`, put the service-role key in the browser, and
leave AI endpoints wide open — because nobody told them not to. This pack
tells them not to, in instructions they actually follow.

Free. Copy it in, and your assistant holds itself to the rules below on every
file it writes from then on.

## What it covers

1. **Row Level Security** — RLS on every table, policies scoped to the owner
   or tenant, never `USING (true)` as a catch-all.
2. **Server secrets stay server-side** — no `service_role` (or any secret key)
   in client code or `NEXT_PUBLIC_` variables.
3. **Private storage by default** — buckets are not public unless there's a
   stated reason.
4. **Paid-API routes are capped** — auth, rate limits, input caps, and a usage
   ceiling on anything that calls a metered API, so a script can't run up
   your bill.
5. **Webhooks verify signatures** — Stripe (and every other provider) events
   are only trusted after the signature checks out.
6. **Every route checks who's calling** — identity, role, and price come from
   the server, never from fields the client sent.
7. **Secrets stay out of git** — `.env` ignored, `.env.example` holds
   placeholders only, committed keys get rotated.
8. **Errors don't leak** — generic messages to clients, no stack traces, no
   tokens in logs.

Each rule comes with a bad/good code example, so both you and the assistant
can see exactly what changes.

## Install

**Cursor** — copy `cursor-rules.mdc` into your project's `.cursor/rules/`
folder as `security.mdc`:

```text
your-project/
  .cursor/
    rules/
      security.mdc   ← the cursor-rules.mdc file from this repo
```

**Windsurf** — copy the same file into `.windsurf/rules/security.md`, or paste
its contents (below the frontmatter) into your project's rules.

**Claude Code** — copy `CLAUDE.md` from this repo into your project root, or
merge its rules section into your existing `CLAUDE.md`.

Then start a new assistant session so the rules load. That's it — no build
step, no dependencies.

## Before you launch: the 10-minute self-check

Rules guard the code your assistant writes *next*. They say nothing about the
state of the project you already have. Run through
[`SELF_CHECK.md`](./SELF_CHECK.md) once before you take payments — six quick
tests you can do yourself, each with what "good" and "bad" look like.

## What this pack can't do

Prompt rules only guard coding habits. They can't check the policies you
already configured in the Supabase dashboard, and they can't find a key that
was committed to git history three weeks ago. Before a project starts taking
real money, it's worth having a person go through it once, end to end.

That's the service behind this repo: the **Go-Live Security & Cost-Cap
Check** — a fixed-price, human review of your Next.js / Supabase / Firebase
app, delivered within 24 hours, with findings written in plain English and
fixes you can paste straight back into your AI builder.

- **Go-Live Security & Cost-Cap Check — $199, 24h:**
  https://shisanyue950901-beep.github.io/security-checkup/
- **Read a full sample report first** (a real audit, fictional app):
  https://shisanyue950901-beep.github.io/security-checkup/sample-report.html

No pressure either way — if the self-check comes back clean, you may not need
it. If something on the list looks off and you'd rather not guess, that's
exactly what the check is for.

## License / reuse

Use it, fork it, adapt it to your stack. If you fix a rule or add one for
another backend (Firebase, Appwrite, a plain Postgres API), contributions are
welcome.

---

Maintained by CapsAGI — security checkups for AI-built apps.
