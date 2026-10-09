# floci-gcp fix tracker (herom-s)

Personal record of my BigQuery fixes for floci-gcp: what went upstream, what is held, and the rules for sending. This branch (`notes/fix-tracker`) only holds this file; it is not meant to be merged anywhere.

Last updated: 2026-10-09 (#325 and #335 merged; #336 approved; #347 ready for review; issue #371 opened). Upstream base for new work: `floci-io/floci-gcp` `main` @ `87b4f26`.

Most of these were found by running a real dbt-bigquery project (bob-datahub-sync, ~170 models) and its FastAPI dashboards against floci's DuckDB engine.

## Rules for sending

- Commits as `herom-s <45332114+herom-s@users.noreply.github.com>`, no AI co-author trailers, Conventional Commit titles.
- At most 2 open non-draft PRs (repo rule). Anything beyond that goes up as a draft and is marked ready when a slot frees.
- Branch off current `upstream/main` (rebase before sending); each PR gets its own issue with a repro.
- Every behaviour claim is checked against real BigQuery first (sandbox project `herom-bq-sandbox-961172`; run `bq` with `CLOUDSDK_CORE_ACCOUNT=heromapp1@gmail.com`). The scripts and outputs live locally in `~/floci-json-evidence/`.
- Follow floci-gcp's AGENTS.md code style (the maintainer asked for it on #335). Before every push, check the diff for: no `var` (write the explicit type, including for-loops and try-with-resources); no inline fully-qualified names (`java.util.Arrays.asList` -> import `Arrays`); imports alphabetical with `java.*`/`javax.*` last and no wildcard imports in `src/main`; no tabs; no em-dashes anywhere, PR descriptions and commit messages included; never an empty `catch`; JBoss `LOG.debugf(...)`-style logging; switch expressions; test names as camelCase sentences. Allowed commit types: `feat`, `fix`, `perf`, `docs`, `chore`.
- Non-draft slots are full right now: #336 and #347. #348, #349 and #356 stay drafts until one of those merges.
- **Held fixes below are pushed here for safekeeping only. No issue or PR until the open PRs (#336, #347, #348, #349, #356) are merged and I decide to send them.**

## Upstream PRs

| PR | Closes | Branch | Title | Status |
|---|---|---|---|---|
| [#307](https://github.com/floci-io/floci-gcp/pull/307) | [#305](https://github.com/floci-io/floci-gcp/issues/305) | `fix/bigquery-ndjson-json-load` | keep NDJSON load values as JSON values in JSON columns | merged 2026-10-05 |
| [#308](https://github.com/floci-io/floci-gcp/pull/308) | [#306](https://github.com/floci-io/floci-gcp/issues/306) | `fix/bigquery-json-type` | return BigQuery type names from JSON_TYPE | merged 2026-10-04 |
| [#317](https://github.com/floci-io/floci-gcp/pull/317) | none (related: #316) | `fix/bigquery-implicit-select-alias` | quote implicit select-list aliases | merged 2026-10-05 |
| [#330](https://github.com/floci-io/floci-gcp/pull/330) | [#316](https://github.com/floci-io/floci-gcp/issues/316) | `fix/bigquery-offset-identifier` | treat offset as an ordinary name | merged 2026-10-06 |
| [#325](https://github.com/floci-io/floci-gcp/pull/325) | none | `fix/bigquery-interval-expression` | accept expressions as interval step sizes | merged 2026-10-08 (4886f1b) |
| [#335](https://github.com/floci-io/floci-gcp/pull/335) | [#337](https://github.com/floci-io/floci-gcp/issues/337) | `fix/bigquery-null-repeated-as-empty` | return a NULL array as an empty array | merged 2026-10-09 (9f8a65a); #337 closed |
| [#336](https://github.com/floci-io/floci-gcp/pull/336) | [#338](https://github.com/floci-io/floci-gcp/issues/338) | `fix/bigquery-array-agg-ignore-nulls` | support null modifiers and LIMIT in ARRAY_AGG | **approved** by hectorvent (2026-10-09); rebased after #335 (keep both tests), head 0a8619d (Greptile P1: keywords after a dot such as `t.limit` read as clauses, fixed with tests); follow-up split out as issue #371 |
| [#347](https://github.com/floci-io/floci-gcp/pull/347) | [#344](https://github.com/floci-io/floci-gcp/issues/344) | `fix/bigquery-ctas-parenthesized-query` | run queries wrapped in parentheses and CTEs inside them | ready for review (2026-10-09); rebased onto main, head 1c7515e; full suite 1983 tests, only EmbeddedDnsServerTest errors locally (WSL UDP, untouched) |
| [#348](https://github.com/floci-io/floci-gcp/pull/348) | [#345](https://github.com/floci-io/floci-gcp/issues/345) | `fix/bigquery-ctas-destination-table` | report the DDL target table as the job destination | open, draft |
| [#349](https://github.com/floci-io/floci-gcp/pull/349) | [#346](https://github.com/floci-io/floci-gcp/issues/346) | `fix/bigquery-merge-insert-source-columns` | resolve bare columns in MERGE NOT MATCHED clauses to the source | open, draft |
| [#356](https://github.com/floci-io/floci-gcp/pull/356) | [#261](https://github.com/floci-io/floci-gcp/issues/261) (filed by hectorvent) | `fix/bigquery-duckdb-external-access` | stop queries from reading files and URLs through DuckDB | open, draft; claimed on the issue |

## Issues I opened

| Issue | Title | State | Fixed by |
|---|---|---|---|
| [#305](https://github.com/floci-io/floci-gcp/issues/305) | NDJSON load job rejects objects/arrays in JSON columns | closed 2026-10-05 | #307 |
| [#306](https://github.com/floci-io/floci-gcp/issues/306) | JSON_TYPE returns DuckDB type names | closed 2026-10-04 | #308 |
| [#316](https://github.com/floci-io/floci-gcp/issues/316) | select-list aliases that are DuckDB keywords fail | closed 2026-10-06 | #317, #330 |
| [#337](https://github.com/floci-io/floci-gcp/issues/337) | a NULL array is returned as null instead of [] | closed 2026-10-09 | #335 |
| [#338](https://github.com/floci-io/floci-gcp/issues/338) | ARRAY_AGG with IGNORE NULLS or RESPECT NULLS fails | open | #336 (approved) |
| [#344](https://github.com/floci-io/floci-gcp/issues/344) | a query wrapped in parentheses fails; CTEs inside parentheses or subqueries read as tables | open | #347 (ready for review) |
| [#345](https://github.com/floci-io/floci-gcp/issues/345) | DDL jobs have no configuration.query.destinationTable | open | #348 (draft) |
| [#346](https://github.com/floci-io/floci-gcp/issues/346) | bare columns in MERGE WHEN NOT MATCHED fail as ambiguous | open | #349 (draft) |
| [#371](https://github.com/floci-io/floci-gcp/issues/371) | arrays with NULL elements are returned instead of failing the query | open, opened 2026-10-09 at hectorvent's request on #336 | none yet |

Also: [#261](https://github.com/floci-io/floci-gcp/issues/261) (filed by hectorvent, claimed by me) is fixed by draft #356.

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
| 11 | `fix/bigquery-format-date-shorthands` | `5cbee24`, `7eff0c9`, `276a55b` | support every BigQuery format element in FORMAT_* and PARSE_*, including formats read from a column | DuckDB lacks or renders differently 22 of the 47 elements (`%C %c %e %g %k %l %n %P %Q %r %s %t %x %z %Ez %E4Y %E<n>S %E*S` and the `%F %D %R` composites) and only takes a constant strftime format. Constant formats (literal or parameter) are split in Java: shared elements in one strftime, composites spelled out, the rest emulated and concatenated; formats from a column are split per row (`regexp_extract_all` + a constant strftime per element). Unknown elements and a trailing `%` print as written, like BigQuery. PARSE_* expands composites of constant formats. Crispin One Page uses `FORMAT_DATE('%F', ...)` | unit 51, Duck IT 13 (47 elements x 2 timestamps, constant and per row) | `format-date-shorthands.json`, `format-param-shorthands.json`, `format-elements-bigquery.json`, `format-from-column.json`, `format-element-edges.json` |
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

Status 2026-10-09: up to date at `50101b7`. It has `upstream/main` `87b4f26` (so #335 as merged, #358, #361, #363), the current heads of #336 (`0a8619d`), #347 (`1c7515e`), #348, #349 and #356 (merged in for the first time), and all 14 held fixes; the subscripts fix is now the final `c8ce5ac` with its two CTAS regression tests. BigQuery, Firestore and Secret Manager tests: 425 passed. The `floci-gcp:local` image has not been rebuilt from it yet.

## Known, not fixed yet

- [#371](https://github.com/floci-io/floci-gcp/issues/371) (opened 2026-10-09, asked for by hectorvent on #336): a result row with an array that has a NULL element must fail the query (`Array cannot have a null element; error in writing field a`); floci returns the NULL element. Applies to any array in the output, not only `ARRAY_AGG`. Evidence `array-agg-null-element-*.json`, `array-literal-null-element-output.json`, `array-agg-struct-null-field-output.json`.

- PARSE_* with a format read from a column: DuckDB requires a constant strptime format and there is no per-row split for parsing. Literal and parameter formats work.
- PARSE_* with elements DuckDB lacks (`%e`, `%k`, `%Q`, ...): only the composites are expanded for parsing.
- FORMAT_TIMESTAMP's time zone argument is ignored (pre-existing); `%z`/`%Ez` print the UTC offset.