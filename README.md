# Brazilian stock market news research data

Research data provided by **Philippe Santa Maria Nizer**, author of the 2011 dissertation and coauthor with J. C. Nievola of *Predicting published news effect in the Brazilian stock market*, **Expert Systems with Applications** 39 (2012), 10674–10680, DOI: [10.1016/j.eswa.2012.02.162](https://doi.org/10.1016/j.eswa.2012.02.162).

This repository contains Portuguese financial news, Brazilian stock prices, and text-processing resources in UTF-8 JSON. The research used news text and stock-price volatility to study the effect of published news on the Brazilian stock market.

## Contents

Dataset paths in the indexes are relative to the repository root.

| Location | Contents | Use in the research |
| --- | --- | --- |
| [data/companies.json](data/companies.json) | 32 study companies, historical tickers, file paths, and record counts | Company mapping; article Tables 1–2 and dissertation Tables 3–4 |
| [data/news/study/](data/news/study/) | 17,538 company-associated news records in 32 files | Portuguese news corpus; 2,552 records pass the study's date/time filter |
| [data/news/additional/](data/news/additional/) | 2,490 records for eight other company searches | Additional collected news outside the published 32-company corpus |
| [data/news/index.json](data/news/index.json) | News file inventory and counts | Locate news by company |
| [data/stocks/study/](data/stocks/study/) | 330,749 quotes for the 32 study tickers | Inputs to log returns and GARCH(3,3), used to define news labels |
| [data/stocks/additional/](data/stocks/additional/) | 49,436 quotes for BRFS3, FIBR3, TCSL3, TNLP4, and TRPL4 | Additional stock series outside the study ticker mapping |
| [data/stocks/historical/PETR4.json](data/stocks/historical/PETR4.json) | Historical Petrobras dataset with 10,831 quotes | Same observations as the primary Petrobras series; counted separately |
| [data/stocks/index.json](data/stocks/index.json) | Stock file inventory, counts, and date ranges | Locate prices by ticker and distinguish versions |
| [data/resources/](data/resources/) | Two stop-word lists, news search notes, and a company selection list | News collection and text preprocessing |
| [data/provenance.json](data/provenance.json) | Research attribution, source identifiers, SHA-256 hashes, sizes, and counts | Identify data sources and verify files |
| [data/validation.json](data/validation.json) | Record counts and integrity checks | Check dataset coverage and consistency |

The news corpus has 20,028 records across 40 company searches and 128 collections. The 37 primary stock series contain 380,185 quotes. The `study` and `additional` directories distinguish the published study's companies from the broader collection.

## News format

Each company file contains:

- `company_prefix`: the company search identifier, normalized to Unicode NFC.
- `study_company`: whether the company belongs to the 32-company study.
- `collections`: collection metadata, including `id`, `company`, `searchExpression`, `startDate`, `finishDate`, and the provenance identifier `original_filename`.
- `records`: news records grouped by collection, in collection order rather than publication-time order.

Each news record has the following fields:

| Field | Meaning |
| --- | --- |
| `id` | Record identifier, such as `bradesco-01:1` |
| `collection_id` | Reference to an entry in `collections` |
| `source_record` | Stable one-based collection ordinal |
| `link` | Article URL |
| `source` | News publisher or outlet |
| `dateTime` | Publication timestamp in `YYYY-MM-DD HH:mm` format |
| `headline` | Article headline |
| `content` | Article body as an HTML string |
| `legacy_filter` | Whether the timestamp passes the study's date/time filter |

A record identifies an article associated with a company. URLs can repeat, and an article mentioning multiple companies can appear in several company files.

The `legacy_filter` accepts timestamps from September 1 through December 31, 2009, with an hour of at least 10, excluding times at or after 17:00 only when the minute is greater than zero. This predicate admits weekends, holidays, and exact-hour times after 17:00. Across the 32 study companies it selects the published 2,552 records. The flag describes date/time eligibility, not an `interesting`/`not-interesting` class label.

All news publication timestamps fall in 2009.

## Stock format

Each stock file contains `ticker`, `study_company`, `historical_variant`, `original_filename`, `source_header`, `timezone`, `price_field`, `interval_minutes`, and a `records` array. `original_filename` and `source_header` are provenance metadata.

Each quote has this shape:

```json
{"timestamp": "2009-08-14T10:05", "price": 13.42}
```

`timestamp` uses `YYYY-MM-DDTHH:mm`. `price` is a JSON number representing Bloomberg `LAST_PRICE`. These are individual price observations at five-minute intervals. Series can contain gaps. Use a decimal-aware JSON reader when exact decimal arithmetic is required. The price adjustment policy is undocumented.

The study series run from August 14, 2009 through February 26, 2010. August history is partial. BRFS3 and FIBR3 start later; the stock index gives each series' exact first and last timestamps.

The historical Petrobras dataset contains the same 10,831 observations as `data/stocks/study/PETR4.json`. Treat these as duplicate observations when combining datasets.

News and quote timestamps are timezone-naive. In stock files, `timezone: null` means the timezone is unspecified.

## Supporting resources

- `stopwords.json`: 258 Portuguese stop-word entries.
- `stopwords-program.json`: the 259-entry stop-word list used by the recovered text preprocessor.
- `news-search-notes.json`: company search expressions and manual search instructions, stored as a list of text lines.
- `company-selection.json`: Bradesco, Gerdau, Itaú, Tam, and Tim, the five names in the selection experiment's list.

The two stop-word lists differ and contain repeated entries. Their word order and accents are significant parts of the stored data.

## Provenance and validation

`data/provenance.json` identifies the research author and publication, and records each dataset's path, SHA-256 checksum, byte size, and record count. Source filenames and checksums identify the underlying research inputs. Dataset paths resolve within this repository; source filenames serve as provenance identifiers.

`data/validation.json` records news and stock totals, checks of article fields and exact decimal prices, and comparison of the 32 filtered company counts with the published counts. It also records file checksum verification and checks for unresolved JSON references and broken documentation links.

News publishers and Bloomberg are the third-party data sources. This repository assigns no new license to their data.
