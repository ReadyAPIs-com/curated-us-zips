# Regenerating `data/us-zips.csv`

This file documents how Ready APIs regenerates the canonical CSV from
production. It's not a runnable script in this repo — production
ingestion happens server-side against the live Postgres database that
backs the API — but the SQL query below is the exact projection.

## Source of truth

The CSV is exported from the `geo_zip_profiles` Postgres table in
production. That table is reconciled from:

1. **SimpleMaps US Zip Codes Basic** for the core ZIP-to-city-state
   crosswalk and centroid coordinates.
2. **U.S. Census Gazetteer** for county and county FIPS where SimpleMaps
   omits them, and for cross-checking centroids.
3. **U.S. Census ACS 5-year** estimates for demographic enrichment.
4. **IANA tz database** for the timezone string (computed from centroid
   coordinates).
5. **OMB CBSA delineations** for the `metro_area` string.

The pipeline lives in `app/services/geoip_service.py` and
`scripts/ingest_geo_zip_profiles.py` in the main Ready APIs repo. It's
not open source today, but the data it produces is — that's this repo.

## Export query

```sql
COPY (
  SELECT
    zip_code,
    city,
    state,
    county,
    county_fips,
    latitude,
    longitude,
    timezone,
    metro_area,
    population,
    median_household_income,
    median_age,
    households,
    avg_household_size,
    bachelors_or_higher_pct,
    median_home_value,
    median_gross_rent,
    poverty_rate_pct,
    unemployment_rate_pct,
    primary_language
  FROM geo_zip_profiles
  ORDER BY zip_code
) TO STDOUT WITH CSV HEADER;
```

That's the entire projection. Same column order, same field semantics,
same RFC 4180 quoting that Postgres' `COPY ... TO STDOUT WITH CSV`
produces by default.

## Refresh cadence

We regenerate this dataset whenever any of these changes:

- Census releases new ACS 5-year estimates (annually, December).
- SimpleMaps publishes a new Basic dataset (a few times a year).
- We notice a row needs correction (rare — opens a GitHub issue first).

On each refresh, we tag the commit with `vYYYY.MM` so consumers can
pin to a specific snapshot. `main` always carries the latest.

## Validation we run before publishing

Before committing a new snapshot, our internal CI runs these checks
(also informative if you fork and regenerate your own version):

- Row count is within 1% of the previous release (catch accidental
  truncations).
- Every row has a non-empty `zip_code`, `city`, and `state`.
- Every `zip_code` matches `^[0-9]{5}$`.
- Every `state` is in the canonical 50 states + DC + 5 territories set.
- No duplicate `zip_code` values.
- Coordinates, when present, are in valid US-territory ranges.
- `county_fips` is exactly 5 digits when present.
- File size is between 2 MB and 5 MB (catch corrupted exports).

If you find a row that fails any of these, please open an issue — it
means our CI missed something.
