# Satoe-Hub Landing Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up a static Jekyll site in `satoe-web/` that markets the Satoe-Hub iOS app and serves real support/privacy pages, ready to deploy on GitHub Pages at `satoe.app`.

**Architecture:** A plain Jekyll site (no remote theme) built with the `github-pages` gem, adapted from Start Bootstrap's "Landing Page" template: one shared layout, a navbar/footer pair of includes, Bootstrap 5 + Bootstrap Icons via CDN, and three pages (home, support, privacy) whose copy is drawn from the existing App Store listing.

**Tech Stack:** Jekyll (via the `github-pages` gem), Ruby 3.2.2 (rbenv), Sass (Jekyll's built-in converter), Bootstrap 5.3.3 + Bootstrap Icons 1.11.3 (CDN, no local build step).

**Spec:** `docs/superpowers/specs/2026-09-27-landing-page-design.md`

## Global Constraints

- Ruby 3.2.2 via rbenv, pinned with a `.ruby-version` file (already installed locally).
- Build with the `github-pages` gem only — no other Jekyll plugins — so the local build matches GitHub Pages' build environment exactly.
- Custom domain is `satoe.app` (apex), set via a `CNAME` file.
- All marketing copy must trace back to `satoe-ios/docs/app-store/store-listing.md` — no invented feature claims.
- No analytics, tracking scripts, or third-party embeds — the app's own listing promises "no tracking," and the site shouldn't contradict it.
- Screenshots and the App Store badge link are placeholders until real assets/the shipped app exist — they must read as clearly placeholder, not presented as final.

## Review Focus

- Relative links between pages (nav → `/support/`, `/privacy/`, `/#features`) resolve to real routes once built, not 404s — covered by the link-grep check in Task 5.
- `assets/css/styles.scss` is missing its Sass front matter and Jekyll serves it as raw, uncompiled text instead of CSS — covered by the compiled-CSS check in Task 3.
- The favicon/app-icon asset 404s because the copy step was skipped or the path in the layout doesn't match — covered by the file-existence check in Task 2 and the `curl` check in Task 5.
- `support.md` / `privacy.md` render at the wrong URL (e.g. `/support.html` instead of `/support/`) because `permalink: pretty` isn't set — covered by the build-output path check in Task 4.
- The navbar doesn't collapse into a mobile menu because the viewport meta tag or Bootstrap's JS bundle is missing from the layout — covered by the head/markup grep in Task 3.

---

### Task 1: Toolchain and site config scaffold

**Files:**
- Create: `satoe-web/.ruby-version`
- Create: `satoe-web/Gemfile`
- Create: `satoe-web/_config.yml`
- Create: `satoe-web/.gitignore`
- Create: `satoe-web/CNAME`

**Interfaces:**
- Consumes: nothing (first task)
- Produces: a working `bundle exec jekyll <cmd>` toolchain that every later task's build/verify steps rely on; `_config.yml`'s `url: "https://satoe.app"` and `permalink: pretty` are relied on by Task 4's clean-URL check.

- [ ] **Step 1: Create `.ruby-version`**

```
3.2.2
```

- [ ] **Step 2: Create `Gemfile`**

```ruby
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins
```

- [ ] **Step 3: Create `_config.yml`**

```yaml
title: Satoe-Hub
description: >-
  Satoe-Hub connects what you do today with who you want to become — plan
  your day, build lasting habits, and watch your Life Map grow.
url: "https://satoe.app"
baseurl: ""
markdown: kramdown
permalink: pretty
```

- [ ] **Step 4: Create `.gitignore`**

```
_site/
.sass-cache/
.jekyll-cache/
.jekyll-metadata
.bundle/
vendor/
Gemfile.lock
```

- [ ] **Step 5: Create `CNAME`**

```
satoe.app
```

- [ ] **Step 6: Confirm rbenv picks up the pinned Ruby version**

Run:
```bash
cd satoe-web
rbenv version
```
Expected: output starts with `3.2.2` (not `system`).

- [ ] **Step 7: Install gems**

Run:
```bash
cd satoe-web
bundle install
```
Expected: ends with `Bundle complete!` and creates `Gemfile.lock` in the directory (it stays untracked per `.gitignore`).

- [ ] **Step 8: Commit**

```bash
cd satoe-web
git add .ruby-version Gemfile _config.yml .gitignore CNAME
git commit -m "Scaffold Jekyll toolchain and site config"
```

---

### Task 2: App icon and screenshot placeholder assets

**Files:**
- Create: `satoe-web/assets/img/icon.png` (copied from `satoe-ios/satoe-hub-iOS-Default-1024@1x.png`)
- Create: `satoe-web/assets/img/screenshot-placeholder.svg`

**Interfaces:**
- Consumes: nothing new
- Produces: `/assets/img/icon.png` and `/assets/img/screenshot-placeholder.svg`, referenced by the layout/navbar (Task 3) and the home page gallery (Task 3).

- [ ] **Step 1: Copy the app icon**

```bash
mkdir -p satoe-web/assets/img
cp "/Users/panca/Projects/satoe/satoe-ios/satoe-hub-iOS-Default-1024@1x.png" \
   "/Users/panca/Projects/satoe/satoe-web/assets/img/icon.png"
```

- [ ] **Step 2: Verify the icon copied correctly**

Run:
```bash
file satoe-web/assets/img/icon.png
```
Expected: `PNG image data, 1024 x 1024, ...` (matches the source file).

- [ ] **Step 3: Create the screenshot placeholder SVG**

`satoe-web/assets/img/screenshot-placeholder.svg`:
```svg
<svg xmlns="http://www.w3.org/2000/svg" width="375" height="812" viewBox="0 0 375 812">
  <rect width="375" height="812" rx="40" fill="#1b1b2f"/>
  <rect x="8" y="8" width="359" height="796" rx="34" fill="#2c2c4a"/>
  <text x="50%" y="50%" fill="#9a9ac0" font-family="sans-serif" font-size="20"
        text-anchor="middle" dominant-baseline="middle">Screenshot placeholder</text>
</svg>
```

- [ ] **Step 4: Verify the SVG is well-formed**

Run:
```bash
grep -q "<svg" satoe-web/assets/img/screenshot-placeholder.svg && echo OK
```
Expected: `OK`

- [ ] **Step 5: Commit**

```bash
cd satoe-web
git add assets/img/icon.png assets/img/screenshot-placeholder.svg
git commit -m "Add app icon and screenshot placeholder assets"
```

---

### Task 3: Layout, shared includes, styling, and home page

**Files:**
- Create: `satoe-web/_layouts/default.html`
- Create: `satoe-web/_includes/navbar.html`
- Create: `satoe-web/_includes/footer.html`
- Create: `satoe-web/assets/css/styles.scss`
- Create: `satoe-web/index.md`

**Interfaces:**
- Consumes: `assets/img/icon.png`, `assets/img/screenshot-placeholder.svg` (Task 2); `site.title`, `site.description` from `_config.yml` (Task 1)
- Produces: the `default` layout and `navbar`/`footer` includes that Task 4's `support.md`/`privacy.md` also use; the compiled `/assets/css/styles.css` route relied on by Task 5's asset check.

- [ ] **Step 1: Create the default layout**

`satoe-web/_layouts/default.html`:
```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% if page.title %}{{ page.title }} · {% endif %}{{ site.title }}</title>
  <meta name="description" content="{{ page.description | default: site.description }}">
  <link rel="icon" href="{{ '/assets/img/icon.png' | relative_url }}">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css" rel="stylesheet">
  <link rel="stylesheet" href="{{ '/assets/css/styles.css' | relative_url }}">
</head>
<body>
  {% include navbar.html %}
  {{ content }}
  {% include footer.html %}
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

- [ ] **Step 2: Create the navbar include**

`satoe-web/_includes/navbar.html`:
```html
<nav class="navbar navbar-expand-md navbar-dark bg-dark fixed-top">
  <div class="container px-4 px-lg-5">
    <a class="navbar-brand" href="{{ '/' | relative_url }}">
      <img src="{{ '/assets/img/icon.png' | relative_url }}" alt="" width="28" height="28"
           class="d-inline-block align-text-top rounded me-2">
      Satoe-Hub
    </a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse"
            data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent"
            aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarSupportedContent">
      <ul class="navbar-nav ms-auto py-4 py-md-0">
        <li class="nav-item"><a class="nav-link px-lg-3 py-3 py-md-4" href="{{ '/' | relative_url }}#features">Features</a></li>
        <li class="nav-item"><a class="nav-link px-lg-3 py-3 py-md-4" href="{{ '/' | relative_url }}#screenshots">Screenshots</a></li>
        <li class="nav-item"><a class="nav-link px-lg-3 py-3 py-md-4" href="{{ '/support/' | relative_url }}">Support</a></li>
        <li class="nav-item"><a class="nav-link px-lg-3 py-3 py-md-4" href="{{ '/privacy/' | relative_url }}">Privacy</a></li>
      </ul>
    </div>
  </div>
</nav>
```

- [ ] **Step 3: Create the footer include**

`satoe-web/_includes/footer.html`:
```html
<footer class="py-5 bg-dark">
  <div class="container px-4 px-lg-5">
    <p class="m-0 text-center text-white">
      &copy; {{ site.time | date: '%Y' }} satoe.app ·
      <a class="text-white" href="{{ '/privacy/' | relative_url }}">Privacy</a> ·
      <a class="text-white" href="{{ '/support/' | relative_url }}">Support</a>
    </p>
  </div>
</footer>
```

- [ ] **Step 4: Create the stylesheet (must keep the empty front matter so Jekyll compiles the Sass)**

`satoe-web/assets/css/styles.scss`:
```scss
---
---
body {
  padding-top: 56px;
}

header.masthead {
  padding-top: 10rem;
  padding-bottom: 10rem;
  text-align: center;
  color: #fff;
  background: linear-gradient(180deg, rgba(20, 20, 40, 1) 0%, rgba(50, 50, 90, 1) 100%);
}

header.masthead h1 {
  font-size: 3rem;
  font-weight: 700;
}

.feature-icon {
  font-size: 2.5rem;
  color: #4b3f9e;
}

.screenshot-frame {
  border-radius: 1rem;
  overflow: hidden;
  box-shadow: 0 0.5rem 1.5rem rgba(0, 0, 0, 0.15);
}
```

- [ ] **Step 5: Create the home page**

`satoe-web/index.md`:
```markdown
---
layout: default
title: Satoe-Hub — Habits, goals & your life map
description: >-
  Set yearly targets for the parts of your life that matter, link the daily
  habits that move them, and watch your Life Map fill in — synced privately
  through iCloud.
---

<header class="masthead">
  <div class="container px-4 px-lg-5">
    <h1>Satoe-Hub</h1>
    <p class="lead">Set yearly targets for the parts of your life that matter,
    link the daily habits that move them, and watch your Life Map fill in —
    synced privately through iCloud.</p>
    <a class="btn btn-light btn-lg mt-3" href="#" role="button">
      <i class="bi bi-apple"></i> Available on the App Store
    </a>
  </div>
</header>

<section id="features" class="py-5">
  <div class="container px-4 px-lg-5">
    <div class="row gx-4 gx-lg-5 row-cols-1 row-cols-md-2 row-cols-lg-3 text-center">
      <div class="col mb-5">
        <div class="feature-icon"><i class="bi bi-calendar3"></i></div>
        <h3>Plan your day</h3>
        <p>One timeline for every habit, to-do and calendar event due today.
        Move a habit for one day without breaking its streak.</p>
      </div>
      <div class="col mb-5">
        <div class="feature-icon"><i class="bi bi-check2-circle"></i></div>
        <h3>Build habits that last</h3>
        <p>Habits you want to do, and ones you want to avoid — daily, on
        chosen weekdays, or on chosen dates, with streaks and a 90-day rate
        for every habit.</p>
      </div>
      <div class="col mb-5">
        <div class="feature-icon"><i class="bi bi-flag"></i></div>
        <h3>Goals &amp; to-dos</h3>
        <p>Goals with target dates, filed under the life dimensions they
        serve, plus to-dos you can repeat daily, weekly, monthly or
        yearly.</p>
      </div>
      <div class="col mb-5">
        <div class="feature-icon"><i class="bi bi-graph-up"></i></div>
        <h3>Review your week</h3>
        <p>Your weekly completion rate against last week's, a
        month-at-a-glance grid, and your longest streak.</p>
      </div>
      <div class="col mb-5">
        <div class="feature-icon"><i class="bi bi-bell"></i></div>
        <h3>Reminders that fit your day</h3>
        <p>A daily nudge at the time you choose, and a heads-up before each
        habit or to-do — nothing for the ones you've already done.</p>
      </div>
      <div class="col mb-5">
        <div class="feature-icon"><i class="bi bi-lock"></i></div>
        <h3>Private by design</h3>
        <p>No account, no ads, no tracking. Your data lives on your iPhone
        and syncs only through your own private iCloud.</p>
      </div>
    </div>
  </div>
</section>

<section id="screenshots" class="py-5 bg-light">
  <div class="container px-4 px-lg-5">
    <h2 class="text-center mb-5">See it in action</h2>
    <div class="row gx-4 gx-lg-5 row-cols-1 row-cols-md-3 text-center">
      <div class="col mb-5">
        <div class="screenshot-frame">
          <img src="{{ '/assets/img/screenshot-placeholder.svg' | relative_url }}"
               alt="Placeholder — Today screen screenshot coming soon" class="img-fluid">
        </div>
        <p class="mt-3">Today</p>
      </div>
      <div class="col mb-5">
        <div class="screenshot-frame">
          <img src="{{ '/assets/img/screenshot-placeholder.svg' | relative_url }}"
               alt="Placeholder — Goals and Dimensions screenshot coming soon" class="img-fluid">
        </div>
        <p class="mt-3">Goals &amp; Dimensions</p>
      </div>
      <div class="col mb-5">
        <div class="screenshot-frame">
          <img src="{{ '/assets/img/screenshot-placeholder.svg' | relative_url }}"
               alt="Placeholder — Weekly review screenshot coming soon" class="img-fluid">
        </div>
        <p class="mt-3">Weekly review</p>
      </div>
    </div>
  </div>
</section>

<section class="py-5">
  <div class="container px-4 px-lg-5 text-center">
    <h2>Get Satoe-Hub</h2>
    <a class="btn btn-primary btn-lg mt-3" href="#" role="button">
      <i class="bi bi-apple"></i> Available on the App Store
    </a>
  </div>
</section>
```

- [ ] **Step 6: Build the site**

Run:
```bash
cd satoe-web
bundle exec jekyll build
```
Expected: ends with `Done in ... seconds.` and no errors.

- [ ] **Step 7: Verify the stylesheet compiled (not served as raw Sass)**

Run:
```bash
head -c 200 satoe-web/_site/assets/css/styles.css
```
Expected: compiled CSS starting with a rule like `body {\n  padding-top: 56px;` — must **not** contain the literal `---` front-matter markers.

- [ ] **Step 8: Verify the home page contains the expected structure**

Run:
```bash
grep -c "feature-icon" satoe-web/_site/index.html
grep -q "Available on the App Store" satoe-web/_site/index.html && echo OK
grep -q 'name="viewport"' satoe-web/_site/index.html && echo OK
grep -q "navbar-toggler" satoe-web/_site/index.html && echo OK
```
Expected: the feature-icon count is `6`, and each `OK` echo prints.

- [ ] **Step 9: Commit**

```bash
cd satoe-web
git add _layouts/default.html _includes/navbar.html _includes/footer.html assets/css/styles.scss index.md
git commit -m "Add layout, navbar/footer includes, styling, and home page"
```

---

### Task 4: Support and privacy pages

**Files:**
- Create: `satoe-web/support.md`
- Create: `satoe-web/privacy.md`

**Interfaces:**
- Consumes: `default` layout, `navbar.html`/`footer.html` includes (Task 3)
- Produces: the `/support/` and `/privacy/` routes that the navbar (Task 3) already links to and that Task 5's `curl` checks exercise.

- [ ] **Step 1: Create the support page**

`satoe-web/support.md`:
```markdown
---
layout: default
title: Support
description: Get help with Satoe-Hub.
---

# Support

Need help with Satoe-Hub, found a bug, or have feedback? Email
**support@satoe.app** and we'll get back to you.

Before writing in, these might already answer your question:

- **No account is needed.** Satoe-Hub works entirely on your device; there's
  nothing to sign in to.
- **Sync uses your own iCloud.** If your iPhone isn't signed into iCloud, the
  app still works — it just won't sync between devices until it is.
- **Calendar access is optional**, requested only if you tap "Connect" in
  Settings → Calendar.
```

- [ ] **Step 2: Create the privacy policy page**

`satoe-web/privacy.md`:
```markdown
---
layout: default
title: Privacy Policy
description: Satoe-Hub's privacy policy.
---

# Privacy Policy

_Last updated: September 27, 2026_

Satoe-Hub is built to work without collecting anything about you. This page
explains exactly what that means.

## No account, no sign-up

Satoe-Hub does not require or offer an account, sign-up, or login of any
kind. There is nothing to register, and we have no way to identify you.

## Where your data lives

Everything you enter into Satoe-Hub — habits, goals, dimensions, to-dos, and
your history — is stored on your iPhone. If your device is signed into
iCloud, that data also syncs through **your own private iCloud account**,
using Apple's CloudKit private database. We never see it, and it is never
stored on any server we operate — we don't operate any servers.

If your device isn't signed into iCloud, Satoe-Hub works exactly the same;
it simply doesn't sync between devices.

## Calendar access

Calendar access is optional and is requested only when you tap "Connect" in
Settings → Calendar. If you grant it, Satoe-Hub reads events from the
calendars you choose, so it can show them beside your habits and warn you
about clashes. Satoe-Hub can only move a calendar event when you explicitly
reschedule one you organize yourself. Your calendar data is never sent off
your device.

## Notifications

Satoe-Hub asks for notification permission once, after onboarding. If you
allow it, you can enable two optional kinds of local notification in
Settings → Reminders: a daily nudge at a time you choose, and a heads-up a
few minutes before a habit or to-do with a start time. These notifications
are scheduled entirely on your device.

## What we don't do

- No account, no sign-up, no password
- No ads
- No analytics or tracking
- No third-party data sharing — we have no data to share
- No servers of ours store your data

## Changes to this policy

If this policy changes, the "last updated" date at the top of this page
will change with it.

## Contact

Questions about this policy: **support@satoe.app**
```

- [ ] **Step 3: Rebuild the site**

Run:
```bash
cd satoe-web
bundle exec jekyll build
```
Expected: ends with `Done in ... seconds.` and no errors.

- [ ] **Step 4: Verify pretty permalinks and content**

Run:
```bash
test -f satoe-web/_site/support/index.html && echo "support OK"
test -f satoe-web/_site/privacy/index.html && echo "privacy OK"
grep -q "support@satoe.app" satoe-web/_site/support/index.html && echo "support content OK"
grep -q "CloudKit" satoe-web/_site/privacy/index.html && echo "privacy content OK"
```
Expected: all four `OK` lines print.

- [ ] **Step 5: Commit**

```bash
cd satoe-web
git add support.md privacy.md
git commit -m "Add support and privacy policy pages"
```

---

### Task 5: Full-site local verification

**Files:**
- None (verification only — no files created or modified)

**Interfaces:**
- Consumes: the fully built site from Tasks 1–4
- Produces: nothing for later tasks — this is the final acceptance check

- [ ] **Step 1: Start the local server in the background**

```bash
cd satoe-web
bundle exec jekyll serve --port 4000 &
echo $! > /tmp/satoe-web-jekyll.pid
```

- [ ] **Step 2: Wait for the server to be ready**

```bash
for i in $(seq 1 20); do
  curl -sf http://127.0.0.1:4000/ >/dev/null && break
  sleep 1
done
```

- [ ] **Step 3: Verify every page and asset returns 200**

```bash
curl -s -o /dev/null -w "home: %{http_code}\n" http://127.0.0.1:4000/
curl -s -o /dev/null -w "support: %{http_code}\n" http://127.0.0.1:4000/support/
curl -s -o /dev/null -w "privacy: %{http_code}\n" http://127.0.0.1:4000/privacy/
curl -s -o /dev/null -w "icon: %{http_code}\n" http://127.0.0.1:4000/assets/img/icon.png
curl -s -o /dev/null -w "css: %{http_code}\n" http://127.0.0.1:4000/assets/css/styles.css
```
Expected: every line reads `200`.

- [ ] **Step 4: Verify internal links resolve to real routes**

```bash
curl -s http://127.0.0.1:4000/ | grep -o 'href="/[^"]*"' | sort -u
```
Expected: only `href="/"`, `href="/#features"`, `href="/#screenshots"`,
`href="/support/"`, `href="/privacy/"` (plus any asset paths already
confirmed in Step 3) — no path outside that set.

- [ ] **Step 5: Stop the server**

```bash
kill "$(cat /tmp/satoe-web-jekyll.pid)"
rm /tmp/satoe-web-jekyll.pid
```

No commit for this task — it verifies work already committed in Tasks 1–4.
