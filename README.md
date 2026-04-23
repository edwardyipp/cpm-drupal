# Centre Point Medan — Drupal 11 CMS

Headless Drupal 11 backend for the Centre Point Medan website. Serves JSON:API content to the Next.js frontend at [`edwardyipp/cpm`](https://github.com/edwardyipp/cpm).

**Production:** `https://v3.centrepoint.co.id` (cPanel, PHP 8.3 via CloudLinux Alt-PHP)
**Local dev:** DDEV
**Architecture notes:** `../cpm/.claude/rules/cms-architecture.md`

---

## Prerequisites

- macOS (Homebrew) or Linux
- [DDEV](https://ddev.com) — `brew install ddev/ddev/ddev`
- [mkcert](https://github.com/FiloSottile/mkcert) — `brew install mkcert nss && mkcert -install`
- Docker Desktop (DDEV needs it)

## Local setup

```bash
# 1. Clone
git clone git@github.com:edwardyipp/cpm-drupal.git
cd cpm-drupal

# 2. Start DDEV (first run pulls images — ~2 minutes)
ddev start

# 3. Install Composer deps (downloads Drupal 11 + contrib modules)
ddev composer install

# 4. Install Drupal
ddev drush site:install standard \
  --account-name=admin \
  --account-pass=admin \
  --site-name="CPM CMS (local)" \
  -y

# 5. Enable our contrib modules
ddev drush en -y \
  gin admin_toolbar admin_toolbar_tools \
  pathauto metatag jsonapi jsonapi_extras \
  simple_oauth scheduler focal_point \
  media media_library content_moderation workflows

# 6. Set Gin as the admin theme
ddev drush theme:enable gin
ddev drush config:set system.theme admin gin -y

# 7. Open in browser
ddev launch
# Login at /user → admin / admin
```

## Common DDEV commands

```bash
ddev start              # Start the project
ddev stop               # Stop the project
ddev restart            # Apply config changes
ddev ssh                # Shell into the web container
ddev composer <cmd>     # Run composer inside the container
ddev drush <cmd>        # Run Drush inside the container
ddev mysql              # MySQL client
ddev launch             # Open site in browser
ddev launch -m          # Open MailHog (catches all outgoing email)
ddev describe           # Project info + URLs + credentials
```

## Module stack (why these, not the Drupal CMS distribution)

We use Drupal Core + a curated set of contrib modules instead of the Drupal CMS distribution. The distribution's headline features (Experience Builder, Haven theme, SEO tools, Forms) are for Drupal-rendered pages — we render in Next.js, so they'd be wasted weight.

| Module | Purpose |
|---|---|
| `gin` + `gin_toolbar` | Modern admin theme — biggest single UX win |
| `admin_toolbar` + `admin_toolbar_tools` | Fast nested admin navigation |
| `pathauto` | Auto URL aliases from patterns |
| `metatag` | SEO meta tags (exposed via JSON:API to Next.js) |
| `jsonapi_extras` | Customize JSON:API output — hide system fields, rename aliases |
| `simple_oauth` | OAuth2 auth for Next.js API calls |
| `scheduler` | Future-dated publish/unpublish — promotions auto-expire |
| `focal_point` | Image crop anchor for responsive thumbnails |

Core modules we rely on: `jsonapi`, `media`, `media_library`, `content_moderation`, `workflows`, `ckeditor5`.

## Deploy

Push to `main` → GitHub Actions runs `composer install --no-dev` → FTP-syncs `web/` + `vendor/` to `public_html/v3/` on cPanel.

Required GitHub secrets (set via `gh secret set`):

- `FTP_SERVER` — e.g. `ftp.centrepoint.co.id`
- `FTP_USERNAME` — the FTP account created in cPanel, scoped to `public_html/v3/`
- `FTP_PASSWORD`
- `DRUPAL_HASH_SALT` — generated once: `openssl rand -base64 55`

First deploy must be followed by opening `https://v3.centrepoint.co.id/install.php` and walking through the installer with the cPanel MySQL credentials. After that, the site is live and future deploys replace code without touching DB or `sites/default/files/`.

## What's NEVER in this repo

```
/vendor/
/web/core/
/web/modules/contrib/
/web/themes/contrib/
/web/sites/default/settings.php
/web/sites/default/settings.local.php
/web/sites/default/files/
/.env
```

Composer manages core + contrib; environment-specific files live only on the machine that needs them.

## Related issues

- **CPM-48** — this setup
- **CPM-91** — content types + JSON:API configuration
- **CPM-92** — marketing-team admin UX polish
- **CPM-49 / 50 / 51** — wire the Next.js frontend to this CMS
