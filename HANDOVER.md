# Handover — VEXTR-fork af Board3 Portal

Dette er **phpBB-udvidelsen** `board3/portal` (mappen `ext/board3/portal/` på et board), ikke et helt phpBB-træ.

Bruges live på **rdeforum.dk**. Upstream er [board3/Board3-Portal](https://github.com/board3/Board3-Portal). Denne fork er [kmyram/Board3-Portal](https://github.com/kmyram/Board3-Portal).

## Hvor

| Hvad | Sti / URL |
| --- | --- |
| Lokal clone | `/Users/kmyram/CodeRepos/Github/Board3-Portal` |
| `origin` (vores fork, push) | `git@github.com:kmyram/Board3-Portal.git` |
| `upstream` (officielt, fetch) | `git@github.com:board3/Board3-Portal.git` |
| Arbejdsbranch | `vextr-php86` → `origin/vextr-php86` |
| `master` lokalt | tracker `upstream/master` (uren upstream, 2023-03-19 `49968461`) |
| Live extension | `/home/kmyram/htdocs/rdeforum.dk/ext/board3/portal` |
| phpBB root live | `/home/kmyram/htdocs/rdeforum.dk` |
| Caddy vhost | `/etc/caddy/sites/meetlele-rde.caddy` (backup `meetlele-rde.caddy.bak-20261028`) |
| Incident-log | `configurations/vextr/VEXTR-PLAYBOOK.md` (2026-10-28/29 rdeforum) |

GitLab-kopien `kmyram/board3-portal` er slettet (GitLab delayed deletion: `board3-portal-deletion_scheduled-88227905`). Brug ikke GitLab til denne kode.

Forken er **public** (GitHub-krav når parent er public).

## Checkout

```bash
cd /Users/kmyram/CodeRepos/Github/Board3-Portal
git status -sb          # skal være: vextr-php86...origin/vextr-php86
git remote -v           # origin = kmyram, upstream = board3
```

Hvis du cloner forfra:

```bash
cd ~/CodeRepos/Github
git clone git@github.com:kmyram/Board3-Portal.git
cd Board3-Portal
git remote add upstream git@github.com:board3/Board3-Portal.git
git fetch upstream
git checkout vextr-php86
```

## Vores commits (oven på upstream 2.3.0)

```
dfb06eee  2026-10-29  Skip portal side columns on UCP pages
5e0270b4  2026-10-28  Allow PHP 8.3–8.5 (vextr.1)
49968461              (upstream/master) Remove not needed file and update composer files
```

Diff vs upstream: `README.md`, `composer.json`, `ext.php`, `event/listener.php`.

Version i `composer.json`: **`2.3.0-vextr.1`**.

### 1. PHP-loft (`5e0270b4`)

Upstream 2.3.0 nægter enable når PHP ≥ `8.3.0-dev`. Host PHP på mailserveren er **8.3.36**, så portalen kunne ikke tændes efter phpBB-opgraderingen.

Ændret:

- `ext.php`: `$php_below_ver = '8.6.0-dev'` (var `8.3.0-dev`)
- `composer.json`: `"php": ">=7.1.0,<8.6.0"` (var `<8.3.0`)
- version `2.3.0-vextr.1`

phpBB-krav er uændret: `>=3.2.6,<3.4@dev` (`$phpbb_below_ver = '3.4.0-dev'`).

Begrundelse fra Board3-vedligeholder **cabot** på [upstream #721](https://github.com/board3/Board3-Portal/issues/721) (2026-01-19): portalen er kompatibel med PHP 8.5.x; skift `8.3.0-dev` → `8.6.0-dev` i `ext.php`. Vi har også rykket composer-constraint (ellers nægter composer/ACP stadig).

Upstream har **ikke** merged det. Sidste push på `board3/Board3-Portal`: **2023-03-19**. Repo er ikke archived (`updated_at` 2026-05-21). cabot svarer stadig på issues — et PR med PHP-loftet er det mest oplagte upstream-bidrag.

### 2. Ingen portal-kolonner på UCP (`dfb06eee`)

ACP på rdeforum:

- `board3_show_all_pages=1` — vis portal-sidekolonne på alle sider
- `board3_show_all_side=1` — det er **højre** kolonne (login, kalender, style)

Det satte højre kolonne (inkl. login-boks) på `ucp.php` — tilmelding, login, privacy, terms, user control panel. Ulæseligt og dobbelt login.

`event/listener.php` — `check_portal_all()` kalder `display_portal()` kun når `board3_show_all_pages` er sat, boardet ikke er disabled, og `skip_portal_on_page()` er falsk.

`skip_portal_on_page()` læser `$user->page['page_name']` (phpBB: script-basename, route hænges på for `app.php/…`). Sammenligningen er kun script-navnet:

- `ucp.php` (også `ucp.php/register`) — skip
- `mcp.php` — skip (samme fokuserede formularer: moderator-panelet)
- `index.php`, `memberlist.php`, tom `page_name` — kolonne vises stadig

Mangler `page` eller er den ikke et array, skjules kolonnerne ikke.

`display_portal()` viser kun én side ad gangen ud fra `board3_show_all_side`:

- `false` → venstre kolonne på “alle sider”
- `true` → højre kolonne på “alle sider”

Selve portalsiden (`/portal` / `/app.php/portal`) har **begge** kolonner (venstre styres af `board3_left_column` + moduler i ACP, ikke af `show_all_side`).

Skip: `ucp.php` og `mcp.php`. Forum-index (`index.php`) og `memberlist.php` har stadig sidekolonnen. `tests/unit/event/listener_test.php::test_show_all_pages_columns` dækker det (`board3_show_all_pages=1`, højre kolonne via `board3_show_all_side=1`). Den ældre `test_check_portal_all` sætter stadig ikke `page_name`.

## Live rdeforum (kontekst, ikke denne repo)

| | |
| --- | --- |
| Board | https://rdeforum.dk — phpBB **3.3.19**, dansk `language/da` |
| PHP | host **8.3.36**, pool `kmyram` (php8.3-fpm) |
| Aktive ext | `board3/portal` + `googleanalytics` + `icecorp/turnstile` 1.2.6 |
| Founder | `OZ8AAZ` (user_id 4). Login-item i 1Password Personal `rdeforum.dk` |
| SSH | `mailserver` / `mail.vextr.net` port **43227**, key `~/.ssh/id_ed25519.mailserver`, `IdentityAgent=none`. VPN blokerer ofte 43227. |

Caddy (2026-10-28): `rewrite /` og `/portal` → `/app.php/portal`, `redir /portal/ /portal 308`. Uden det ramte `/portal` `index.php` via `wp_shield` try_files, og venstre kolonne manglede. Verificer med `#portal-left` + “Fødselsdage” på origin `/` og `/portal`. `index.php` har kun “all pages”-højre kolonne.

ACP portal (da det virkede): `board3_left_column=1` (bredde 180), venstre-moduler birthday/search/clock/attachments.

### Deploy af denne extension

Repo-roden **er** `ext/board3/portal/`. Kopiér filer ind, overskriv ikke hele phpBB.

```bash
# eksempel — kun de filer vi har rørt, eller rsync af hele extensionen
rsync -av --delete \
  --exclude .git --exclude composer.phar --exclude tests \
  ./ mailserver:/home/kmyram/htdocs/rdeforum.dk/ext/board3/portal/

ssh mailserver 'cd /home/kmyram/htdocs/rdeforum.dk && php bin/phpbbcli.php cache:purge'
```

Efter PHP-loft-patch første gang: `php bin/phpbbcli.php extension:enable board3/portal` (allerede enabled).

Live `listener.php` fik UCP-skip 2026-10-29; hold den i sync med `vextr-php86`.

## Ikke i denne fork

Følgende ligger i phpBB-træet / andre ext, ikke her:

- **Register “Fejl!” / HTTP 500** — dansk `language/da/ucp.php` `PASS_TYPE_*_EXPLAIN` havde to sprintf-placeholders; phpBB 3.3.19 sender én. Også manglende `CHARACTERS_XY`, `PRIVACY_LINK`, `TERMS_LINK` i `language/da/common.php`. Icecorp manglede `ext/icecorp/turnstile/language/da/common.php`.
- **Turnstile** — `ext/icecorp/turnstile/`, keys i 1Password `rdeforum.dk Turnstile`. ACP `captcha_plugin=icecorp.turnstile.captcha`.
- **Logo** — prosilver `site_logo.png` / `colours.css` hash `f41a3b15`, `assets_version` 134.
- **Sessions** — `phpbb3_sessions` blev TRUNCATEt (27.6M rækker). Prune/cron er stadig et åbent driftsproblem.
- phpBB blev overlayet fra official `phpBB-3.3.19.zip`; `config.php`, `files/`, `store/`, `ext/`, `language/da`, avatars bevaret. `install/` fjernet.

## Downstream (træk upstream ind)

```bash
git checkout vextr-php86
git fetch upstream
git merge upstream/master          # eller rebase, hvis historikken skal være lineær
# løs konflikter i ext.php / composer.json (PHP-loft) og event/listener.php (UCP-skip)
git push origin vextr-php86
```

Forvent konflikter præcis i de to patch-filer. Behold `$php_below_ver = '8.6.0-dev'`, composer `<8.6.0`, og `skip_portal_on_page()`.

Upstream `master` har ikke rørt sig siden 2023-03-19, så merge er tom indtil de pusher.

## Upstream (send tilbage)

To separate, små PR’er mod `board3/Board3-Portal` er nemmere end ét VEXTR-bundt:

1. **PHP 8.3–8.5** — `ext.php` + `composer.json` (uden at tvinge `2.3.0-vextr.1` på dem; lad dem bumpe version). Citer cabot i #721.
2. **Skip portal on UCP** — `event/listener.php` + en unit-test. Produktbeslutning; de har ældre issues om kolonner på alle sider (#673, #684).

```bash
git fetch upstream
git checkout -b php-8.6-ceiling upstream/master
# cherry-pick 5e0270b4, strip vextr.1 version hvis ønsket
git push origin php-8.6-ceiling
# PR: https://github.com/kmyram/Board3-Portal/compare/board3:master...kmyram:php-8.6-ceiling
```

UCP/MCP-skip til upstream: branch fra `upstream/master` med diffen af `event/listener.php` + `tests/unit/event/listener_test.php` (ikke `HANDOVER.md`). `dfb06eee` mangler MCP og tests. Committen ovenpå cherry-picker ikke rent alene, fordi den forudsætter skippet fra `dfb06eee`.

## Tests

Der er ingen PHP på Mac. Test- og udviklingsmiljøet er Docker, ved siden af fork-en (ikke i dette git-repo):

`/Users/kmyram/CodeRepos/GitHub/board3-phpbb-dev`

phpBB **3.3.19** med `composer install` fra phpBB’s lock ligger i `phpbb/`. Extensionen bind-mountes fra denne clone.

```bash
cd /Users/kmyram/CodeRepos/GitHub/board3-phpbb-dev
docker compose run --rm test
# php -l / ad hoc på samme PHP som rdeforum:
docker compose run --rm php83 -l event/listener.php
```

`test` er `php:7.4-cli` + PHPUnit **7.5.20** (det phpBB 3.3.19 låser). Uden argumenter kører den `tests/unit/event/listener_test.php`. Flere argumenter sendes til PHPUnit. PHPUnit 7 starter ikke på PHP 8.3, så suiten bliver på `test`. `php83` er `php:8.3-cli` med extensionen i `/src`.

Kørsel mod en kopi af rdeforum (PHP 8.3, MariaDB, localhost:8088):

```bash
cd /Users/kmyram/CodeRepos/GitHub/board3-phpbb-dev
docker compose -f rdeforum-compose.yml up -d
docker compose -f rdeforum-compose.yml stop
```

Kopien er kode uden `files/` og `store/`, plus et dump uden `phpbb3_confirm`, `phpbb3_qa_confirm` og sessions-rækker. `config.php` og forum-URL peger på Docker-databasen, ikke live. Containeren mount’er denne clone ind over `ext/board3/portal`, så det er `vextr-php86` der kører, ikke snapshotet fra serveren.

2026-10-29: listener-testen, 12 tests, 15 assertions, OK. Fuld suite er ikke kørt. `bin/phpunit.sh` sletter `phpBB/cache/twig` før hver kørsel: phpBB 3.3.19 kalder `clearCacheFiles()`, som Twig 2.16 ikke har.

## Næste naturlige skridt

- PR til upstream for PHP-loftet (højest sandsynlighed for accept). Cherry-pick af `5e0270b4` uden `2.3.0-vextr.1`.
- PR til upstream for UCP/MCP-skip: kun `event/listener.php` + `tests/unit/event/listener_test.php` (ikke `HANDOVER.md`).
- Hold live `ext/board3/portal` = `vextr-php86` (deploy-kommandoen ovenfor). Live har endnu ikke MCP-skippet før næste deploy.
- phpBB 3.4 / PHP 8.6: loft og `$phpbb_below_ver` skal ses efter igen.

## Konventioner

- Bash/php som upstream. Ingen Salt, ingen CI i `configurations` til denne fork.
- Dokumentation: dansk + engelsk blandet er fint; match omkringliggende fil.
- Print aldrig 1Password-hemmeligheder, Turnstile-secret, eller SSH-nøgler.
- `configurations` git-push har fleet-effekt — den repo er et andet sted.

## Hurtig fil-guide

| Fil | Rolle |
| --- | --- |
| `ext.php` | `is_enableable()` — PHP/phpBB-loft. **VEXTR-patched.** |
| `composer.json` | version + php constraint. **VEXTR-patched.** |
| `event/listener.php` | sidekolonner på alle sider + UCP-skip. **VEXTR-patched.** |
| `acp/portal_module.php` | ACP-felter `board3_show_all_pages` / `board3_show_all_side` |
| `controller/` | `/app.php/portal` route |
| `modules/` | blokke (birthday, login, calendar, …) |
| `migrations/` | config-defaults (`show_all_pages` default 0) |
| `language/` | portal-sprog (ikke phpBB core da-pack) |
| `styles/` | portal CSS/HTML |
| `tests/` | unit + functional |
