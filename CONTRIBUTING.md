# Contributing to `curated-us-zips`

This is a **data distribution**, not a software project — the value here is in `data/us-zips.csv`, generated from public sources (USPS, U.S. Census ACS, NWS timezones, etc.) and normalized to a consistent schema. PR shape and review priorities differ from a typical OSS repo.

## What we accept

### Data corrections

If you spot a wrong ZIP / city / county / coord, file an issue with:

1. The ZIP code
2. The wrong value as it appears in the CSV
3. The correct value
4. A **public source** we can cite — USPS ZIP Code Lookup, Census Gazetteer, official municipal source, etc. (We can't accept "I live there" — corrections feed back into a generation pipeline and need verifiable provenance.)

We'll fix it in the next data refresh (typically within a week) and credit you in the changelog if you'd like.

### Schema additions

New columns are case-by-case. Open an issue describing:

1. What field you want added
2. The public dataset it would come from + that dataset's license
3. How it would be normalized (e.g. timezone always as IANA name, not Olson offset)

We'd rather keep the schema lean than chase every adjacent field — for the long tail, the live [Location Enrichment API](https://readyapis.com/apis/location-enrichment) is the better tool.

### Documentation / README fixes

Just send a PR. Typos, broken links, clarification — all welcome.

## What we don't accept

- Pull requests that modify `data/us-zips.csv` directly. The CSV is regenerated from upstream sources by a pipeline; hand-edits would be overwritten on the next refresh. File an issue with the source instead.
- Schema changes that break backward compatibility (column renames, type changes). Additive changes only.

## Filing an issue

For data: see above. For repo bugs (broken release tarball, missing column header, etc.): just describe what you ran and what happened.

## License

The data is CC BY 4.0 — attribution required, commercial use allowed. The README and scripts are MIT. By contributing data corrections you affirm the underlying public source permits redistribution under CC BY 4.0.
