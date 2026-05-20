# readyapis/curated-us-zips

[![License](https://img.shields.io/badge/data%20license-CC%20BY%204.0-blue.svg)](#license)
[![Rows](https://img.shields.io/badge/rows-33%2C100-informational.svg)](#whats-in-the-csv)
[![Updated](https://img.shields.io/badge/last%20updated-2026--05-informational.svg)](CHANGELOG.md)

> A clean, comprehensive, machine-readable CSV of every U.S. ZIP code — with city, state, county, lat/lng, timezone, metro area, population, and U.S. Census ACS demographic enrichment. **One file, MIT-friendly, free to use.**

This is the dataset that backs [Ready APIs](https://readyapis.com)' `/api/v1/geo/zip/<zip>` endpoint. We curate it from public sources, normalize the shapes (a Census `county_fips` like `01001` always renders as a zero-padded 5-character string; coordinates are always `lat,lng` in decimal degrees; etc.), and re-publish here so anyone can use it without hitting an API.

```bash
# 3.0 MB, 33,100 rows. Use it for anything.
curl -L -o us-zips.csv https://raw.githubusercontent.com/ReadyAPIs-com/curated-us-zips/main/data/us-zips.csv
```

If you need the live API surface (per-request lookup, batch validation, cross-checks against city / state / metro), use Ready APIs at [readyapis.com/apis/geo](https://readyapis.com/apis/geo). If you just need the data dump, you're in the right place — clone, download, ship.

## What's in the CSV

`data/us-zips.csv` — one row per ZIP code (33,100 total), header row, UTF-8, RFC 4180 quoting.

| Column                      | Type    | Example          | Source                |
|----------------------------|---------|------------------|------------------------|
| `zip_code`                 | string  | `30303`          | USPS / SimpleMaps      |
| `city`                     | string  | `Atlanta`        | USPS / SimpleMaps      |
| `state`                    | string  | `GA`             | USPS / SimpleMaps      |
| `county`                   | string  | `Fulton`         | U.S. Census            |
| `county_fips`              | string  | `13121`          | U.S. Census            |
| `latitude`                 | float   | `33.7544`        | U.S. Census Gazetteer  |
| `longitude`                | float   | `-84.3902`       | U.S. Census Gazetteer  |
| `timezone`                 | string  | `America/New_York` | IANA tz database     |
| `metro_area`               | string  | `Atlanta-Sandy Springs-Alpharetta, GA` | OMB CBSA |
| `population`               | integer | `12345`          | U.S. Census ACS 5-year (2022) |
| `median_household_income`  | integer | `78400`          | U.S. Census ACS        |
| `median_age`               | float   | `36.4`           | U.S. Census ACS        |
| `households`               | integer | `5230`           | U.S. Census ACS        |
| `avg_household_size`       | float   | `2.31`           | U.S. Census ACS        |
| `bachelors_or_higher_pct`  | float   | `45.8`           | U.S. Census ACS        |
| `median_home_value`        | integer | `385000`         | U.S. Census ACS        |
| `median_gross_rent`        | integer | `1620`           | U.S. Census ACS        |
| `poverty_rate_pct`         | float   | `12.4`           | U.S. Census ACS        |
| `unemployment_rate_pct`    | float   | `4.1`            | U.S. Census ACS        |
| `primary_language`         | string  | `English`        | U.S. Census ACS        |

ZIPs included:
- All 50 states + DC
- Territories: Puerto Rico (PR), U.S. Virgin Islands (VI), Guam (GU), American Samoa (AS), Northern Mariana Islands (MP)
- Military APO/FPO/DPO ZIPs are excluded (they don't have geographic centroids)

Demographic fields are empty (`""`) for territories where Census ACS doesn't publish the same series.

## Quick examples

### Python — load + filter

```python
import csv

with open("data/us-zips.csv", newline="", encoding="utf-8") as f:
    rows = list(csv.DictReader(f))

# Every ZIP in Georgia
ga = [r for r in rows if r["state"] == "GA"]
print(f"{len(ga)} ZIPs in Georgia")

# Median income above $100k
rich = [r for r in rows if r["median_household_income"] and int(r["median_household_income"]) > 100_000]
print(f"{len(rich)} ZIPs with median income > $100k")
```

### pandas

```python
import pandas as pd

df = pd.read_csv("data/us-zips.csv", dtype={"zip_code": str, "county_fips": str})
print(df.groupby("state")["population"].sum().sort_values(ascending=False).head(10))
```

### Node

```js
import fs from "node:fs";
import { parse } from "csv-parse/sync";

const records = parse(
  fs.readFileSync("data/us-zips.csv"),
  { columns: true, skip_empty_lines: true }
);
console.log(`${records.length} ZIPs loaded`);
```

### SQLite

```bash
sqlite3 zips.db <<EOF
.mode csv
.import data/us-zips.csv zips
SELECT state, COUNT(*) FROM zips GROUP BY state ORDER BY 2 DESC LIMIT 10;
EOF
```

### Postgres

```sql
CREATE TABLE us_zips (
  zip_code text PRIMARY KEY,
  city text NOT NULL,
  state text NOT NULL,
  county text,
  county_fips text,
  latitude double precision,
  longitude double precision,
  timezone text,
  metro_area text,
  population integer,
  median_household_income integer,
  median_age double precision,
  households integer,
  avg_household_size double precision,
  bachelors_or_higher_pct double precision,
  median_home_value integer,
  median_gross_rent integer,
  poverty_rate_pct double precision,
  unemployment_rate_pct double precision,
  primary_language text
);
\COPY us_zips FROM 'data/us-zips.csv' WITH (FORMAT csv, HEADER true);
```

## Why this exists

ZIP datasets are everywhere on the internet, but they're almost always one of:

- **Out of date** (Census Gazetteer downloads that haven't been refreshed in three years)
- **Locked in a paid product** (Smarty, Loqate, Melissa — fantastic data, but you pay per call)
- **Partial** (ZIP → state only, no county, no demographics)
- **Inconsistent shapes** (county_fips sometimes 4 digits because the leading zero got dropped in Excel)

We needed a single clean source for our own API, so we curated this. Since the underlying data is mostly public domain, there's no reason not to redistribute the cleaned version.

If you find yourself reaching for it: that's the point. If you want it as a live API (with batch lookup, fuzzy matching, ZIP+4, deliverability flags), check out [Ready APIs](https://readyapis.com) — same data, plus enrichment, served by HTTP.

## License

The data in `data/us-zips.csv` is published under **[Creative Commons Attribution 4.0 (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)** — use it freely, including commercially, with attribution.

Suggested attribution line:

> ZIP data from [Ready APIs curated-us-zips](https://github.com/ReadyAPIs-com/curated-us-zips), sourced from U.S. Census Bureau, USPS, and SimpleMaps.

### Underlying source licenses

This file combines several upstream sources. Their original licenses still apply to their respective portions:

- **U.S. Census Bureau** (ACS 5-year 2022, Gazetteer, FIPS codes, CBSA delineations): public domain (U.S. federal government work, not subject to copyright per 17 U.S.C. § 105).
- **U.S. Postal Service** (ZIP code assignments, city-state crosswalk): not subject to copyright; redistribution permitted.
- **SimpleMaps US Zip Codes Basic** (some ZIP centroids and city-state pairs): CC BY 4.0. The Basic version is free for commercial use with attribution.
- **IANA tz database**: public domain.

The structure, normalization, and combination of these sources into this single CSV is published by Ready APIs under CC BY 4.0.

## Contributing

Found a bad row? Open an issue with the ZIP code and what you observed. PRs welcome for:

- Corrections (cite a source)
- Schema additions that have a clear, redistributable upstream
- Better examples / language snippets in the README

Out of scope:

- Adding ZIPs that aren't real (vanity ZIPs, future allocations)
- Scraping from paid datasets
- Adding fields without a public-domain or CC-compatible source

## Refresh cadence

This dataset is regenerated whenever we re-ingest the upstreams in production (roughly monthly, sometimes faster when Census publishes new ACS estimates).

The CSV in `main` is always the current production snapshot. Past snapshots are tagged as releases — pin to a specific tag if you need reproducibility.

## See also

- [Ready APIs Geo namespace](https://readyapis.com/apis/geo) — same data, plus enrichment, served as JSON over HTTP. 1,000 free credits/month, no card.
- [readyapis-python](https://github.com/ReadyAPIs-com/readyapis-python) — Python SDK
- [readyapis-node](https://github.com/ReadyAPIs-com/readyapis-node) — Node CLI
- [SimpleMaps US Zip Codes](https://simplemaps.com/data/us-zips) — original upstream for the basic ZIP centroids
- [U.S. Census ACS](https://www.census.gov/programs-surveys/acs) — original upstream for demographic fields
