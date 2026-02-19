# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the Apache2 configuration for **unix.com**, version-controlled in git. It serves the main unix.com site (PHP/Rails hybrid) and proxies traffic to local Rails microservices.

## Common Commands

```bash
# Test config syntax before reloading
apache2ctl configtest

# Apply config changes (safe, no downtime)
systemctl reload apache2

# Full restart (use sparingly)
systemctl restart apache2

# Enable/disable a site or module
a2ensite <site>   /   a2dissite <site>
a2enmod <mod>     /   a2dismod <mod>
a2enconf <conf>   /   a2disconf <conf>
```

## Architecture

### Request Flow for www.unix.com (port 443)

`001-unix-le-ssl.conf` is the primary VirtualHost. Requests pass through these included files in order:

1. **`unix_redirect/logging_blocks.conf`** — custom log rules
2. **`unix_redirect/kill_blocks.conf`** — 410/403/forbidden returns for bots, bad UAs, legacy paths (`.php`, `/cgi-bin/`, Discourse junk)
3. **`unix_redirect/proxy_blocks.conf`** — `ProxyPass` directives and PHP-FPM handler
4. **`unix_redirect/rails_static_aliases.conf`** — static asset aliases
5. **`unix_redirect/manpage_priority.conf`** — short-circuits man page routes to prevent downstream rewrites from intercepting them
6. **`unix_redirect/redirect_logic.conf`** — final fallback that sends most traffic to `/redirect.php`

### Rails App Proxy Routing

Traffic is proxied to local Rails apps via `unix_redirect/proxy_blocks.conf`:

| Path | Port | App |
|---|---|---|
| `/man_pages`, `/man-pages`, `/man_page`, `/man-page`, `/fast_index` | 3001 | Unix Man Pages |
| `/stellar` | 3002 | Stellar Distance Calculator |

PHP files are handled via PHP-FPM at `unix:/run/php/php8.3-fpm.sock`.

### Custom Configs (`conf.neo/`)

Included globally (not scoped to a VirtualHost):

- `neo_badbots.conf` — block known bad bots
- `neo_403_user_agents.conf` — 403 for bad user agents
- `neo_protected_areas.conf` — password-protected areas (credentials in `secure/.htpasswd`)
- `oldsite_redirects.conf` — legacy URL redirects
- `rewrite_rss.conf` — RSS feed rewrites
- `manpages.conf` / `condormanpages.conf` — man page specific rules

### Other Active Sites (`sites-enabled/`)

- `000-community.conf` / `000-community-le-ssl.conf` — community.unix.com
- `catchall-ssl.conf` — catch-all HTTPS vhost
- `loadalert*.conf`, `naruemon*.conf`, `nodevnull*.conf`, `op.conf`, `openphone*.conf` — auxiliary sites

## Key Conventions

- **Order matters** in `001-unix-le-ssl.conf`: `kill_blocks` → `proxy_blocks` → `manpage_priority` → `redirect_logic`. Adding a new proxy route goes in `proxy_blocks.conf`; adding a new kill/block goes in `kill_blocks.conf`.
- **New Rails-proxied paths** must be carved out in both `proxy_blocks.conf` (the `ProxyPass`) and `redirect_logic.conf` (the bypass exemptions before the `redirect.php` fallback).
- **`ServerTokens Prod` / `ServerSignature Off`** are set in `apache2.conf` — do not expose version info.
- The `conf.neo/` directory is a custom drop-in zone (not standard Debian `conf-available/conf-enabled`); files there are included directly from vhost configs or globally.
