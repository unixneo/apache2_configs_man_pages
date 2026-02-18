# unix.com Apache2 Configuration

`/etc/apache2` configuration for unix.com, version-controlled in git.

## Structure

- **`apache2.conf`** — Main Apache configuration
- **`sites-available/`** — Virtual host configs (unix.com, community, catchall, etc.)
- **`sites-enabled/`** — Symlinks to active virtual hosts
- **`conf-available/`** / **`conf-enabled/`** — Supplementary configs
- **`conf.neo/`** — Custom configs: bot blocking, bad user agents, protected areas, redirects
- **`unix_redirect/`** — Proxy and rewrite rules for routing traffic to Rails apps
- **`mods-available/`** / **`mods-enabled/`** — Apache modules

## Rails App Routing

Traffic is proxied to local Rails apps via `unix_redirect/proxy_blocks.conf`:

| Path | Port | App |
|---|---|---|
| `/man_pages`, `/man-pages`, `/man_page`, `/man-page`, `/fast_index` | 3001 | Unix Man Pages |
| `/stellar` | 3002 | Stellar Distance Calculator |

## Custom Configs (conf.neo/)

- `neo_badbots.conf` — Block known bad bots
- `neo_403_user_agents.conf` — Block bad user agents with 403
- `neo_protected_areas.conf` — Password-protected areas
- `oldsite_redirects.conf` — Legacy URL redirects
- `rewrite_rss.conf` — RSS feed rewrites
- `manpages.conf` / `condormanpages.conf` — Man page specific rules

## Deployment

After editing configs, test and reload:

```bash
apache2ctl configtest
systemctl reload apache2
```
