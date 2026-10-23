# Davis WeatherLink Console 6313 cloud driver for WeeWX

Collect Console 6313 weather and health observations through the WeatherLink v2
cloud API. The console must upload to Davis and the WeeWX host needs Internet
access and API credentials. This driver does not read a local console API or
download historical archives. It includes Console and Health Console skins in
English and German, with optional AirLink and second-ISS observations.

This repository mirrors [WernerKr/davisconsoleapi](https://github.com/WernerKr/davisconsoleapi).
Use the separately repaired catalog release **0.43.post2**. Its schema is installed
as `user.wview_davisconsoleapi.schema`; the mirror source keeps the old
`bin/schemas` layout. A mirror branch download is not the catalog distribution.

## Install

The catalog release requires Python >=3.10,<3.15 and WeeWX >=5.5,<6.

Install `requests` in the WeeWX Python environment:

```sh
python -m pip install requests
```

Use the `weectl` from your WeeWX Python environment:

```sh
weectl extension install https://github.com/weewx-packages/catalog/releases/download/davisconsoleapi/v0.43.post2/davisconsoleapi-0.43.post2.tar.gz --config=/path/to/weewx.conf
```

Replace `/path/to/weewx.conf` with your station configuration. Stock WeeWX 5
accepts this fixed URL or a downloaded copy of the same archive. No catalog
integration is required. Restart WeeWX yourself after configuration changes.

## Configure cloud acquisition

The catalog installer supplies driver/helpers and skins. Its configuration hook
changes an active archive binding using WeeWX's default extended schema to
`user.wview_davisconsoleapi.schema`. Review the configuration before starting
WeeWX, especially on an existing station. It does not migrate database columns.
For a new, separate SQLite archive, merge these settings:

```ini
[Station]
    station_type = DavisConsoleAPI
[DavisConsoleAPI]
    driver = user.davisconsoleapi
    station_id = YOUR_STATION_ID
    api_key = YOUR_API_KEY
    api_secret = YOUR_API_SECRET
    polling_interval = 300
    txid_iss = 1
    airlink = 0
    packet_log = 0
[StdArchive]
    record_generation = software
    archive_interval = 300
[DataBindings]
    [[wx_binding]]
        database = davisconsoleapi_sqlite
        table_name = archive
        manager = weewx.manager.DaySummaryManager
        schema = user.wview_davisconsoleapi.schema
[Databases]
    [[davisconsoleapi_sqlite]]
        database_type = SQLite
        database_name = davisconsole.sdb
```

Obtain a v2 key and secret from your WeatherLink account and a station ID from
its station-list API. The source [davis_api_toolc.py](davis_api_toolc.py) is a
legacy URL-generation helper, not an installed `weectl` command. Protect its
credentials and generated URLs. The driver can log signed URLs at debug level
and configured credentials when required settings are missing; redact logs.

Set `txid_iss` to the ISS transmitter ID. Enable AirLink only when available in
the same account. `polling_interval` defaults to 300 seconds and has a 60-second
minimum; actual freshness depends on Davis permissions and source updates.
The runtime reports 0.43; `.post2` identifies the catalog packaging revision.

## Enable reports and migrate existing data

Configure report sections explicitly; installation does not enable these reports:

```ini
[StdReport]
    [[DavisConsole]]
        skin = console
        enable = true
        lang = en
    [[DavisHealthConsole]]
        skin = healthc
        enable = true
        lang = en
        HTML_ROOT = healthc
```

Use `de` for German. Review output destinations and other enabled reports, then
restart WeeWX. Existing databases need a backup and deliberate schema migration;
changing the schema setting alone does not add columns. The old `add_console.sh`
uses WeeWX 4 commands and fixed station paths, so do not run it unchanged.

The [reference](docs/reference.md) preserves upstream version history, API
samples, transmitter/skin settings and database migration background. Optional
sunshine/rain duration uses `user.sunrainduration.SunshineDuration` in
`[Engine][[Services]] process_services`; append it without removing other services.
The installed schema lives in `bin/user/wview_davisconsoleapi.py`, even though
the mirrored source file is [bin/schemas/wview_davisconsoleapi.py](bin/schemas/wview_davisconsoleapi.py).

Cloud account behavior and physical Console/AirLink coverage were not checked
live in this refresh. The decoder expects specific API fields; account access
does not establish support for every sensor. Original authors' notices, the
[GPL license](LICENSE) and bundled font licenses remain with the source/assets.

<!-- Documentation updated 2026-10-23 by weewx-packages. -->
