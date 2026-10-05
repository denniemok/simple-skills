---
name: simple-supabase
description: Guide the user through a secure Supabase setup, including project, schema, row level security, keys, auth, and environment variables. Use when the user asks to set up, audit, or secure Supabase, or asks about RLS or key exposure. Not for ordinary queries in app code.
metadata:
  author: denniemok
  version: "1.2.2"
license: MIT
---

# Simple Supabase

Guide the user through setting up or auditing Supabase for an app. You cannot change the dashboard, so the user makes each change. Give one step at a time with the exact path and value, wait for them to confirm, and verify when you can. Never ask for keys, passwords, or tokens, and never print them.

## 1. Gather facts

Read the repo first (README, `.env.example`, existing SQL, client and server code), then ask only what is missing:

- What the data is, and who should read or write it
- Whether the browser talks to Supabase directly, or only a server does
- Whether users sign in, and whether sign-up is open or invite only
- Whether Supabase is required, or the app should still run without it

## 2. Project

1. Create the project in the dashboard and choose the region closest to the users or the server.
2. In the creation options:
   - Turn on Automatic RLS, so new tables get row level security enabled by default. It only applies to tables created afterwards, so still enable RLS on any existing table (section 5).
   - Turn on the Data API, which `supabase-js` and the REST API need. Only leave it off if the app talks to Postgres directly through a connection string and never uses `supabase-js`.
   - Leave automatic exposure of new tables off (the Data API option that grants new tables to the API roles). Each table then needs explicit grants (section 5), so nothing is reachable by accident.
3. Free projects pause after about a week of inactivity, so note that for low-traffic apps.
4. Find the URL and keys under Project Settings, API.

## 3. Keys

| Key | Use | Exposure |
|-----|-----|----------|
| Publishable (anon) key | Browser client, limited by RLS | Public, safe in the client bundle |
| Secret (service role) key | Trusted server only, bypasses RLS | Never in the client, never in the repo |

- Any variable with a client prefix (`VITE_`, `NEXT_PUBLIC_`, `PUBLIC_`) ends up in the browser. The secret key must never have one.
- Name variables by where they run, for example `SUPABASE_SECRET_KEY` on the server and `VITE_SUPABASE_PUBLISHABLE_KEY` in the frontend.
- Keep `.env.example` in sync with placeholder values, and keep real `.env` files out of git.
- If a secret key leaks, rotate it in the dashboard, then update every deployment.

Which key does the work depends on the architecture:

- Client only (no backend): the browser uses the publishable key. Signed-in requests carry the user's token and run as `authenticated`, so RLS policies are the whole authorization layer.
- With a backend: prefer forwarding the user's token to Supabase, so queries run as `authenticated` and RLS still applies if server code misses a check. Keep the secret key for work that must cross users, such as cron jobs, admin tools, and webhooks.
- If the backend uses the secret key for everything, RLS is bypassed and all authorization lives in server code. Every query must scope itself to the current user.

## 4. Schema

- Keep the schema in a SQL file in the repo, such as `supabase/schema.sql`, and run it in the SQL editor.
- Make it safe to re-run: `create table if not exists`, `create index if not exists`.
- Add checks and constraints (`not null`, `check (... in (...))`), and indexes on columns you filter or sort by.
- Use `timestamptz` for times, and `jsonb` for flexible state.

## 5. Row level security

- Enable RLS on every table in the `public` schema, even with Automatic RLS on: `alter table public.<table> enable row level security;` It is harmless to repeat, and it keeps the schema file safe when run on another project.
- With RLS on and no policy, the publishable key and user tokens can read and write nothing. That is the safe default for tables only the secret key touches, since it bypasses RLS.
- Add a policy only when requests with a user's token must reach a table (from the browser, or from a backend that forwards the token), and scope it to the user, for example `using ((select auth.uid()) = user_id)`.
- Never write `using (true)` on a table with private data.
- Run the Security Advisor in the dashboard (Advisors) and fix what it flags.

Grants decide which roles can touch a table at all, and RLS decides which rows. With automatic exposure off, grant only the role that actually reaches each table, in the schema file next to the table. Pick by who talks to the table:

```sql
-- Requests carry the user's token, from the browser or a forwarding backend (RLS policies scope the rows):
revoke all on public.<table> from anon;
grant select, insert, update, delete on public.<table> to authenticated;

-- Only the server touches it, with the secret key (browser uses the publishable key for auth only):
revoke all on public.<table> from anon, authenticated;
grant select, insert, update, delete on public.<table> to service_role;
```

- Grant `anon` only for data that is meant to be public, and only `select`.
- Grant `service_role` only on tables that server code reaches with the secret key. It bypasses RLS, so an unused grant only widens the damage if the key leaks. The dashboard Table Editor and SQL editor run as `postgres`, so they work without it.
- Grant only the actions the app uses, for example skip `delete` if rows are never deleted.
- With automatic exposure on, `anon` and `authenticated` get every table, and RLS with no policy is the only thing stopping them. That works, but one table with RLS off is then fully readable with the public key. Explicit grants add a second layer.

RLS only protects tables. The publishable key is public (it sits in the bundle when it is in a `NEXT_PUBLIC_` or `VITE_` variable), so anything in an exposed schema that it can reach is public too:

- Views run with their owner's rights by default, so a view can bypass the RLS of the tables under it. Create views with `security_invoker = true`, or keep them out of the exposed schema.
- Functions in `public` can be called through the API. A `security definer` function runs with its owner's rights and can bypass RLS, so avoid it unless needed, and set `search_path` on it. Revoke execute from `anon` and `authenticated` on functions the browser should not call.
- Prefer a private schema for helper views and functions, and expose only what the browser needs.

## 6. Auth

- Create users under Authentication, Users, or enable the sign-in methods the app needs.
- For an invite-only app, turn off public sign-ups.
- Verify the user's token on the server, and check their role or an email allowlist there. Never trust a client-side check alone.
- Set the site URL and redirect URLs under Authentication, URL Configuration, for every environment.

For roles (admin, editor, and so on), keep them in a roles table keyed by `user_id`, or in `app_metadata` set by the server. Use both layers:

- Server: decide which actions a user may take, and enforce business rules (plan limits, who can invite).
- RLS: decide which rows a user may touch, as the last line of defense. A policy can read the roles table:

  ```sql
  using (exists (
    select 1 from public.user_roles
    where user_id = (select auth.uid()) and role = 'admin'
  ))
  ```

  For heavy use, a Custom Access Token Hook can put the role in the JWT, so policies read `auth.jwt()` instead of querying the table each time. The claim only updates when the token refreshes, so a removed role keeps working until then. Query the table where revocation must be instant.

## 7. Storage (if used)

- Buckets are private by default. Make one public only for content that is meant to be public.
- Add storage policies for who can upload, read, and delete.

## 8. App behavior

- If Supabase is optional, detect a missing config and fall back gracefully instead of crashing.
- Create one shared client per runtime and reuse it.
- Handle errors from every query, and do not log secrets.
- Select only the columns the screen needs, for example `.select('id, title, status')` instead of `.select('*')`. Skip large `jsonb` or text columns in list views and fetch them when a single row opens, so payloads stay small.

## 9. Verify

- Check grants and RLS in the SQL editor. Expect only the roles and actions from section 5, and `rowsecurity` true on every table:

  ```sql
  select grantee, table_name, privilege_type
  from information_schema.role_table_grants
  where table_schema = 'public'
  order by table_name, grantee;

  select tablename, rowsecurity
  from pg_tables
  where schemaname = 'public';
  ```

  The owner role (`postgres`) has every privilege, which is normal. Look for unexpected `anon` or `service_role` rows.
- With only the publishable key, query a private table through the REST API. Expect an empty result or a permission error, never rows. Do the same for every view, and try calling each function.
- Sign in as a normal user and confirm they cannot read another user's rows.
- Check the built client bundle for the secret key. It must not appear.
- Run the app once with Supabase unconfigured, if it is meant to work that way.

## Common traps

- Policies: `using` filters the rows a user can read or change, `with check` validates rows they write. An update needs a select policy too. Write `(select auth.uid())` instead of `auth.uid()` for speed, and index the columns that policies use.
- Never authorize from `user_metadata`, users can edit it. Use `app_metadata` set on the server, or a roles table.
- On the server (Next.js), trust `getUser()`, not `getSession()`, which reads the cookie without verifying it.
- Anyone can sign up with the publishable key even if the UI hides the form. Turn off sign-ups and anonymous sign-ins if they are not wanted.
- The built-in email sender is for testing and heavily rate limited. Set up custom SMTP before launch.
- The REST API returns at most 1000 rows by default, with no error. Paginate with `.range(from, to)` until a page comes back short, or data will silently go missing. The limit is Max rows under Project Settings, API (Data API settings). Raising it only delays the problem.
- Serverless hosts (Vercel, Next.js) that connect to Postgres directly should use the pooled connection string, or connections run out.
- To store a profile per user, use a trigger on `auth.users` that inserts into a `public` table. This is a fair use of `security definer`, with a fixed `search_path`.
- Realtime and Storage have their own access rules. Add policies, and use private channels and buckets for private data.
- Keep dev and prod as separate projects, change the schema only through the schema file, and take your own backups of data that matters.

## Checklist

- Schema in the repo and safe to re-run
- Automatic RLS, Data API, and automatic table exposure chosen at project creation
- RLS on for every public table, no `using (true)` on private data
- Explicit grants per table, no `anon` on private data, no unused `service_role`
- Views use `security_invoker`, functions are not needlessly `security definer`, and execute is revoked where the browser should not call them
- Secret key server only, publishable key in the client
- `.env.example` complete, real env files ignored by git
- Auth URLs, sign-up setting, and server-side token checks set
- Security Advisor clean
- Verification steps pass
