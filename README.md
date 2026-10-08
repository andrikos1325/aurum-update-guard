# Aurum Update Guard (free)

Two questions before and after every Home Assistant update, answered by Home Assistant itself:
**"Is it safe to update now?"** and **"Did the update break anything?"**

Built only with Home Assistant's own features (template sensors, an automation, helpers and the built-in backup
sensors): **no custom integration, no frontend plugin, nothing loaded from the internet** - so it keeps working after updates.

## What it does
- **`binary_sensor.update_guard_safe_to_update`** - `on` when your last automatic backup is less than 26 hours old,
  the last backup attempt did not fail, the backup system is idle, a recent snapshot exists and no problems are open.
  The `reasons` attribute says what is missing.
- **Snapshot** (`sensor.update_guard_snapshot`) - which automations and scripts exist (with their names) and which
  entities are unavailable. Taken every night at 03:30 and whenever you press **"Update Guard - take snapshot now"**.
  It survives restarts, so the "before" state is still there after the update.
- **Check after every restart** - waits until the integrations have loaded (5 minutes, adjustable), then compares
  with the snapshot. If something is missing it creates a notification like this (real example from our test instance):

  > **Update Guard: something changed after the restart**
  > **Automations missing or broken (1):** Living room window - heating pause (automation.test_window_pause). They
  > are in your backup from before the update. If this is expected, press "Update Guard - accept changes".

- **`binary_sensor.update_guard_problem`** - `on` while there are changes you have not accepted; press
  **"Update Guard - accept changes"** when a change was on purpose.

- **Database too big? (new in 1.1)** - `sensor.update_guard_database_size` (MB, via Home Assistant's built-in SQL
  integration - a tiny query, no table scan) and `binary_sensor.update_guard_database_too_big` (above 1500 MB by default;
  change "Update Guard - database warning above"). While it is too big you get one notification a day with what helps:
  keep less history (`purge_keep_days`), stop recording noisy entities (`recorder: exclude:`), shrink the file once
  (`recorder.purge` with `repack: true`). The safe-to-update sensor shows it as a **note** (`notes` attribute) - not a
  reason to wait, but updates that change the database take longer on big databases. Default query = SQLite (Home
  Assistant's default); MariaDB/MySQL and PostgreSQL queries are in `update_guard.yaml`.

## Install (5 minutes)
1. Turn on **automatic backups** (Settings > System > Backups).
2. **The template macros:** with [HACS](https://hacs.xyz) (category "Template"), install **Aurum Update Guard** - HACS
   puts `update_guard.jinja` into `/config/custom_templates/`. Without HACS: copy `update_guard.jinja` from this
   repository to `/config/custom_templates/` (create the folder if needed).
3. **The package** (HACS cannot install packages): make sure `configuration.yaml` loads packages (add it once, then
   check the configuration):
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
   and copy `packages/update_guard.yaml` to `/config/packages/`.
4. Restart Home Assistant. Add the two binary sensors to a dashboard, or use them in your own automations.

(`custom_templates/update_guard.jinja` is the same file as `update_guard.jinja`, kept for older install links.)

Requirements: Home Assistant 2025.2 or newer (backup sensors). Tested on 2026.9.4 in a separate test instance (1.1: 36 function checks + 9 database checks).
Limits: Home Assistant cannot prove that a backup restores; checks that scan all entities refresh at most once a minute.

## Pro edition
**[Aurum Update Guard Pro](https://localhavenstore.gumroad.com/l/aurum-update-guard)** (EUR 5, also
[on Etsy](https://www.etsy.com/listing/4584935364)) adds:
- a **"Prepare update"** button - fresh automatic backup + new snapshot, then READY / not ready (refuses while
  problems are open),
- the report **grouped by device** and sent to your **phone** (any notify action), quiet after normal restarts,
- a **last report** sensor, an **updates waiting** list, a **restore-drill reminder**,
- an **Update check dashboard** and two short guides: "Update Home Assistant safely" and "The 10-minute restore drill".

The free core fires the event `update_guard_report` after each check, so you can also build your own extensions.

## Licence
MIT - see [LICENSE](LICENSE).
Made with AI assistance: the YAML and templates were written with the help of AI tools, then tested and refined on a
real Home Assistant test instance. Not affiliated with or endorsed by Home Assistant / Nabu Casa.
