# Satoe-Hub landing page — design

## Purpose

`satoe-web` is a new static site, hosted on GitHub Pages at the custom domain
`satoe.app`, that markets the Satoe-Hub iOS app (see `satoe-ios/`). It replaces
the `satoe.app/support` and `hub.satoe.app/privacy` placeholders referenced in
`satoe-ios/docs/app-store/store-listing.md` with real pages.

Audience: someone who lands on the site from an App Store link, a search
result, or word of mouth, and needs to understand what the app does and how
to get it in under a screen's worth of scrolling.

## Template

Adapted from Start Bootstrap's "Landing Page" template (MIT-licensed):
Bootstrap 5 via CDN, a fixed navbar, a full-height hero/masthead, an icon
feature grid, and a footer. No JS framework, no build tooling beyond Jekyll
itself — kept close to the template's original structure so it stays easy to
reason about and re-theme later.

Built with the `github-pages` gem (matches GitHub Pages' supported plugin
and Jekyll versions exactly) so what builds locally is what deploys.

## Pages and content mapping

Content is drawn from `satoe-ios/docs/app-store/store-listing.md` rather than
invented fresh, so the site and the App Store listing stay consistent.

- `index.md` — hero (tagline adapted from the promotional text), six feature
  blocks (Plan your day, Build habits, Goals & to-dos, Review your week,
  Reminders, Private by design — condensed from the description's bullet
  sections), a screenshot gallery with placeholder image slots (no exports
  exist yet), and an "Available on the App Store" badge linking to a
  placeholder href (`#`) until the app is live.
- `support.md` — a contact email for support requests, replacing the
  `satoe.app/support` placeholder.
- `privacy.md` — a real privacy policy written from the facts already
  committed to in the App Review notes: no account/sign-up, data lives on
  device and syncs only through the user's own private iCloud (CloudKit),
  calendar access is optional and calendar data never leaves the device, two
  optional local notification types, no ads, no tracking, no third-party
  data sharing.

## File structure

```
satoe-web/
  _config.yml
  Gemfile
  .ruby-version          # 3.2.2, matches an rbenv version already installed
  CNAME                  # "satoe.app"
  .gitignore             # _site/, .sass-cache/, .bundle/, .jekyll-cache/
  index.md
  support.md
  privacy.md
  _layouts/
    default.html
  _includes/
    navbar.html
    footer.html
  assets/
    css/
      styles.scss        # Sass entry point; template overrides live here
    img/
      icon.png           # copied from satoe-ios's 1024x1024 app icon
      screenshot-placeholder-*.png  # labeled placeholders, swappable later
```

## Domain

`CNAME` file contains `satoe.app` (apex domain, per the answered question).
DNS configuration itself is out of scope — that's a registrar-side change
the user makes separately; this repo only needs the `CNAME` file GitHub
Pages expects.

## Out of scope

- Real screenshots (placeholders only, until exports exist)
- The actual App Store link (placeholder `#` until the app ships)
- DNS record changes at the registrar
- Analytics/tracking scripts (would also contradict the "no tracking"
  privacy claim made in the app's own store listing)

## Testing

`bundle exec jekyll serve` locally (Ruby 3.2.2 via rbenv) to confirm the site
builds without errors and all three pages render and link to each other
correctly.
