---
name: simple-launch
description: Pre-launch checklist for a website or web app, covering metadata, SEO, social cards, content, performance, and production config. Use only when the user asks to prepare for launch.
disable-model-invocation: true
metadata:
  author: denniemok
  version: "1.0.0"
license: MIT
---

# Simple Launch

Check a site is ready to go live. Read the repo and the built output, then report each item as pass, fail, or not applicable, with the file. Fix only when the user asks. Start with the secrets sweep, since a leak blocks everything else. For DNS, SSL, caching, and security headers, use `simple-cloudflare`. For the database, use `simple-supabase`.

## Workflow

1. Detect the framework (Astro, Next.js, Vite SPA, or other) and the hosting target.
2. Ask for the production URL if the config does not have the real one.
3. Go through the sections below.
4. Report failures first, with the file and the fix. Then list what passed.

## 1. Secrets sweep

A leaked key is the highest impact failure, so treat any hit as a launch blocker. Search four places:

1. Working tree. Look for known key shapes: `sk-`, `sk_live_`, `AKIA`, `ghp_`, `github_pat_`, `xoxb-`, `AIza`, `-----BEGIN PRIVATE KEY-----`, `service_role`, `secret`, `password=`, and connection strings with credentials such as `postgres://user:pass@`. Also check for JWTs starting with `eyJ`.
2. Tracked files. Run `git ls-files` and look for `.env`, `*.pem`, `*.key`, credentials files, and database dumps. `.gitignore` must cover them, and `.env.example` must hold placeholders only.
3. Git history. A secret deleted in a later commit is still in history. Use `gitleaks detect` or `trufflehog git file://.` if installed, otherwise `git log -p -S "<pattern>"`.
4. Built output. Search `dist/`, `build/`, or `.next/static` for the same patterns. Client-prefixed variables (`VITE_`, `NEXT_PUBLIC_`, `PUBLIC_`) are shipped to the browser, so none of them may hold a secret key. Check that production source maps are off, or do not expose anything sensitive.

Also check for:
- Hardcoded admin or test accounts, debug flags, and open debug endpoints
- Wildcard CORS (`*`) on an API that uses cookies or private data
- Third-party keys meant for the browser (maps, analytics) restricted to your domain in that provider's settings
- Secrets in the README, docs, screenshots, CI config, and Docker files

If you find one:
- Never print the full value. Show the file, line, and type, with the value masked.
- Tell the user to rotate the key first, then remove it. Removing it from the code alone does not make it safe.
- After rotating, update every deployment that used it.

## 2. Identity and metadata

- Each page has a unique `<title>` and meta description
- `lang` is set on `<html>`
- Canonical URL is set and uses the production domain
- Favicon exists, with an Apple touch icon if the site is used on mobile
- Social card: `og:title`, `og:description`, `og:image` (1200 x 630), `twitter:card`
- The social image URL is absolute and loads

## 3. Discoverability

- `sitemap.xml` is generated and lists the real URLs
- `robots.txt` exists and does not block the whole site by mistake
- Staging or preview builds are `noindex`, production is not
- A custom 404 page exists, and old URLs have redirects
- Semantic structure: one `h1` per page, headings in order

## 4. Framework config

| Framework | Check |
|-----------|-------|
| Astro | `site` in `astro.config` is the production URL, since the sitemap and canonical URLs depend on it |
| Next.js | `metadataBase` is set, `app/sitemap.ts` and `app/robots.ts` exist, `metadata` on each route |
| Vite SPA | Meta tags in `index.html`, SPA fallback route on the host, and consider prerendering for SEO pages |

Watch for leftover `TODO`, localhost, and default `.github.io` or `.vercel.app` URLs in the config.

## 5. Content

- No placeholder text, lorem ipsum, `TODO`, or test data
- Links and buttons work, and external links open safely
- Images have `alt` text, correct sizes, and modern formats
- Contact details, social links, and legal pages (privacy, terms) are correct
- Copyright year and site name are right

## 6. Performance and accessibility

- Run Lighthouse on the built site, mobile first. Aim for 90 or more in each category.
- Images are sized and lazy loaded below the fold, fonts are preloaded or self-hosted
- Keyboard navigation works, contrast is fine, forms have labels
- Check the main pages at phone, tablet, and desktop widths

## 7. Production config

- Production env vars are set on the host, and none are missing
- HTTPS works and redirects from HTTP
- Analytics is set up, plus a consent notice if the audience needs one
- Error tracking or logs are reachable
- The build succeeds from a clean install

## 8. After launch

- Open the live site and click through the main flows
- Test the social card with the sharing debugger of each platform
- Submit the sitemap in Google Search Console
- Set up uptime monitoring if the site matters

## Output

- Failures, each with file and fix
- Warnings worth a look
- Passed items in one line
- Verdict: ready to launch, or not, and the blockers
