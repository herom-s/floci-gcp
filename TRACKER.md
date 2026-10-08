# floci-gcp fix tracker (herom-s)

Personal record of my BigQuery fixes for floci-gcp: what went upstream, what is held, and the rules for sending. This branch (`notes/fix-tracker`) only holds this file; it is not meant to be merged anywhere.

Last updated: 2026-10-07 (#335 approved; AGENTS.md checklist). Upstream base for new work: `floci-io/floci-gcp` `main` @ `5dabe8e` (0.10.0).

Most of these were found by running a real dbt-bigquery project (bob-datahub-sync, ~170 models) and its FastAPI dashboards against floci's DuckDB engine.

## Rules for sending

- Commits as `herom-s <45332114+herom-s@users.noreply.github.com>`, no AI co-author trailers, Conventional Commit titles.
- At most 2 open non-draft PRs (repo rule). Anything beyond that goes up as a draft and is marked ready when a slot frees.
- Branch off current `upstream/main` (rebase before sending); each PR gets its own issue with a repro.
- Every behaviour claim is checked against real BigQuery first (sandbox project `herom-bq-sandbox-961172`; run `bq` with `CLOUDSDK_CORE_ACCOUNT=heromapp1@gmail.com`). The scripts and outputs live locally in `~/floci-json-evidence/`.
- Follow floci-gcp's AGENTS.md code style (the maintainer asked for it on #335). Before every push, check the diff for: no `var` (write the explicit type, including for-loops and try-with-resources); no inline fully-qualified names (`java.util.Arrays.asList` -> import `Arrays`); imports alphabetical with `java.*`/`javax.*` last and no wildcard imports in `src/main`; no tabs; no em-dashes anywhere, PR descriptions and commit messages included; never an empty `catch`; JBoss `LOG.debugf(...)`-style logging; switch expressions; test names as camelCase sentences. Allowed commit types: `feat`, `fix`, `perf`, `docs`, `chore`.
- **Held fixes below are pushed here for safekeeping only. No issue or PR until the open drafts (#336, #347, #348, #349) are merged and I decide to send them.**

## Upstream PRs

| PR | Closes | Branch | Title | Status |
|---|---|---|---|---|
| [#307](https://github.com/floci-io/floci-gcp/pull/307) | [#305](https://github.com/floci-io/floci-gcp/issues/305) | `fix/bigquery-ndjson-json-load` | keep NDJSON load values as JSON values in JSON columns | merged 2026-10-05 |
| [#308](https://github.com/floci-io/floci-gcp/pull/308) | [#306](https://github.com/floci-io/floci-gcp/issues/306) | `fix/bigquery-json-type` | return BigQuery type names from JSON_TYPE | merged 2026-10-04 |
| [#317](https://github.com/floci-io/floci-gcp/pull/317) | none (related: #316) | `fix/bigquery-implicit-select-alias` | quote implicit select-list aliases | merged 2026-10-05 |
| [#330](https://github.com/floci-io/floci-gcp/pull/330) | [#316](https://github.com/floci-io/floci-gcp/issues/316) | `fix/bigquery-offset-identifier` | treat offset as an ordinary name | merged 2026-10-06 |
| [#325](https://github.com/floci-io/floci-gcp/pull/325) | none | `fix/bigquery-interval-expression` | accept expressions as interval step sizes | open, ready for review |
| [#335](https://github.com/floci-io/floci-gcp/pull/335) | [#337](https://github.com/floci-io/floci-gcp/issues/337) | `fix/bigquery-null-repeated-as-empty` | return a NULL array as an empty array | **approved** by avison9 (2026-10-07); `var` -> `List<TableCell>` fixed in 6d5e089, draft note removed |
| [#336](https://github.com/floci-io/floci-gcp/pull/336) | [#338](https://github.com/floci-io/floci-gcp/issues/338) | `fix/bigquery-array-agg-ignore-nulls` | support null modifiers and LIMIT in ARRAY_AGG | open, draft |
| [#347](https://github.com/floci-io/floci-gcp/pull/347) | [#344](https://github.com/floci-io/floci-gcp/issues/344) | `fix/bigquery-ctas-parenthesized-query` | run queries wrapped in parentheses and CTEs inside them | open, draft |
| [#348](https://github.com/floci-io/floci-gcp/pull/348) | [#345](https://github.com/floci-io/floci-gcp/issues/345) | `fix/bigquery-ctas-destination-table` | report the DDL target table as the job destination | open, draft |
| [#349](https://github.com/floci-io/floci-gcp/pull/349) | [#346](https://github.com/floci-io/floci-gcp/issues/346) | `fix/bigquery-merge-insert-source-columns` | resolve bare columns in MERGE NOT MATCHED clauses to the source | open, draft |
| [#356](https://github.com/floci-io/floci-gcp/pull/356) | [#261](https://github.com/floci-io/floci-gcp/issues/261) (filed by hectorvent) | `fix/bigquery-duckdb-external-access` | stop queries from reading files and URLs through DuckDB | open, draft; claimed on the issue |

## Held fixes (pushed to this fork, no issue or PR yet)

1 to 10 branch off `5dabe8e` (0.10.0) and need a rebase onto `upstream/main` before sending; 11 to 14 branch off `362809b` (`upstream/main`, 2026-10-08).

| # | Branch | Commit | Title | What it fixes | Tests | Evidence |
|---|---|---|---|---|---|---|
| 1 | `fix/bigquery-array-subscripts` | `c8ce5ac` | translate OFFSET, ORDINAL and SAFE_ array subscripts | `arr[OFFSET(n)]`, `[SAFE_OFFSET(n)]`, `[ORDINAL(n)]`, `[SAFE_ORDINAL(n)]` and bare `[n]`, with BigQuery's 0/1-based and out-of-bounds semantics | unit 61, Duck IT 11, DML IT 13 | `gaps-2026-10-06.txt` |
| 2 | `fix/bigquery-window-frame-current-row` | `7b0e7a2` | keep ROW a keyword in CURRENT ROW window frame bounds | `ROWS BETWEEN ... AND CURRENT ROW` had `ROW` quoted as a column | unit + Duck IT 60 | `gaps-2026-10-06.txt` |
| 3 | `fix/bigquery-csv-skip-leading-rows` | `a396afb` | accept skipLeadingRows sent as a string in load jobs | the Python client sends `skipLeadingRows` as a string; dbt seeds loaded the header as data | Load IT 11 | `gaps-2026-10-06.txt` |
| 4 | `fix/bigquery-generate-date-array` | `f1a758d` | support GENERATE_DATE_ARRAY | shim over `generate_series`; NULL bound gives `[]`, zero step errors like BigQuery | 60 | `gaps-2026-10-06.txt` |
| 5 | `fix/bigquery-alter-table-set-options` | `78033dd` | support ALTER TABLE and ALTER VIEW SET OPTIONS | dbt-bigquery runs `alter table <seed> set OPTIONS()` after every seed; description, friendly_name, labels, expiration_timestamp, NULL clears, IF EXISTS | unit 50, DML IT 13 | `alter-2026-10-06-{a,b,c}.txt` |
| 6 | `fix/bigquery-struct-star-expansion` | `6cbee56` | expand STRUCT values with .* (unnest) | `expr.*` on a STRUCT (dotted path, call, subscript), e.g. `ARRAY_AGG(STRUCT(...))[OFFSET(0)].*` | unit 49, Duck IT 11 | `struct-star-2026-10-06.txt` |
| 7 | `fix/bigquery-initcap` | `709c9d7` | support INITCAP | DuckDB has no INITCAP; character walk with BigQuery's default delimiter set and optional custom delimiters | unit 49, Duck IT 11 | `initcap-2026-10-06.txt` |
| 8 | `fix/bigquery-any-value-having` | `44bc5ea` | support ANY_VALUE with HAVING MAX and HAVING MIN | `ANY_VALUE(x HAVING MAX y)` was a parser error; now `arg_max_null`/`arg_min_null` with BigQuery's NULL and tie rules, and its error for `OVER` | unit 49, Duck IT 11 | `having-max-2026-10-06.txt` |
| 9 | `fix/bigquery-alias-after-null-keyword` | `e3f6110` | treat a name after NULL, TRUE or FALSE as an implicit alias | `SELECT x IS NOT NULL c` (also `NULL n`, `TRUE t`, `x IS NOT FALSE nf`) got `AS f0_` appended after the alias, a DuckDB syntax error; `NULL`/`TRUE`/`FALSE` now end a value like `END` does | unit 49, Duck IT 11, DML IT 12 | `null-alias-2026-10-07.txt` |
| 10 | `fix/bigquery-is-distinct-from` | `f77e84c` | keep IS [NOT] DISTINCT FROM out of FROM clause handling | the `FROM` in `a IS [NOT] DISTINCT FROM b` was read as a FROM clause, so the operand was resolved as a table (`Table name "b" missing dataset`) in the select list, `WHERE`, `JOIN ... ON` and `CASE WHEN` | unit 49, Duck IT 11, DML IT 12 | `is-distinct-from-2026-10-07.txt` |
| 11 | `fix/bigquery-format-date-shorthands` | `5cbee24`, `7eff0c9` | expand %F, %D and %R in FORMAT_* and PARSE_* formats | DuckDB strftime/strptime reject `%F`, `%D`, `%R` ("Unrecognized format for strftime"); literal formats are expanded in Java, parameter formats in SQL with `replace()` (`%%` parked on `chr(57344)`), NULL format stays NULL; `%T` already worked. Crispin One Page uses `FORMAT_DATE('%F', ...)` | unit 51, Duck IT 12 | `format-date-shorthands.json`, `format-param-shorthands.json` |
| 12 | `fix/bigquery-string-param-temporal-coercion` | `7be55a1` | coerce date-shaped STRING parameters like BigQuery | STRING params were inlined as `CAST(... AS VARCHAR)` and DuckDB refuses `DATE BETWEEN VARCHAR`; date-shaped values are now untyped literals (DuckDB casts them implicitly), others keep the cast so `INT64 = STRING` still fails as in BigQuery | unit 52, Duck IT 12 | `string-param-coercion.json`, `string-param-vs-int64.json`, `string-param-invalid-date.json` |
| 13 | `fix/bigquery-alias-after-parameter` | `b4ffd44` | keep an implicit alias that follows a query parameter | `SELECT @p x` / `SELECT ? y` got `AS f0_` appended after the alias (DuckDB syntax error); `isAliasable` now accepts named and positional parameters | unit 52, Duck IT 11 | `param-implicit-alias.json` |
| 14 | `fix/bigquery-string-param-date-arithmetic` | `f238e8b` | coerce STRING operands of date and time functions | `DATE_ADD(@d, INTERVAL 1 DAY)` with a STRING `@d` (or a string literal) failed: DuckDB has no `VARCHAR + INTERVAL`. A string literal or STRING parameter operand of `*_ADD`, `*_SUB`, `*_DIFF`, `*_TRUNC` (DATE, DATETIME, TIMESTAMP, TIME) is cast to the function's type | unit 51, Duck IT 12 | `string-param-date-arithmetic.json`, `string-param-date-arithmetic-it.json` |

Notes for when these go up:

- AGENTS.md cleanup still needed (audit 2026-10-07; all open PRs are clean): inline fully-qualified names in #1 (`java.util.Arrays.asList`, 1), #5 (`java.math.BigDecimal` in `src/main`, `java.util.Map.of` in a test), #6 (1), #8 (3), #9 (2) and #10 (2). Replace with imports when rebasing each branch.
- The subscripts rewrite must run in `run()`/`runDml()`, not `prepare()`: `classify()` re-translates CREATE TABLE/VIEW text, and the rewrite is not idempotent.
- The subscripts pass and #336's ARRAY_AGG shim both touch `ARRAY_AGG(...)[...]`; re-test that combination after rebasing.
- Translator tests from different branches land in the same spots; resolve conflicts by keeping both sides and check with `git diff --name-only --diff-filter=U` before committing a merge.

## `local/all-fixes`

Integration branch: `upstream/main` (merged 2026-10-08 at `362809b`, which brings #294, the Firestore nested update mask fix bob's `update_source` needs) plus every PR branch above and all 14 held fixes, merged. It is what the local `floci-gcp:local` image is built from (`docker build -f docker/Dockerfile -t floci-gcp:local .`). With it, bob-datahub-sync's dbt project builds 166/166 models and all of its `/ads/*` dashboard endpoints answer. Rebuild it from the branches; don't send it upstream.

## Known, not fixed yet

- A FORMAT_*/PARSE_* format read from a column: DuckDB requires a constant format ("strftime format must be a constant"). Literals and parameters work (#11).
