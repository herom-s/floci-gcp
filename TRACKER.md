# floci-gcp fix tracker (herom-s)

Personal record of my BigQuery fixes for floci-gcp: what went upstream, what is held, and the rules for sending. This branch (`notes/fix-tracker`) only holds this file; it is not meant to be merged anywhere.

Last updated: 2026-10-14 (#348 merged 2026-10-13; #349 reworked after hectorvent's and avison9's review and Greptile's sixth pass; follow-up issue #410 opened with its fix queued; #371 fix opened as draft #412). Upstream base for new work: `floci-io/floci-gcp` `main` @ `a27384b`.

Most of these were found by running a real dbt-bigquery project (bob-datahub-sync, ~170 models) and its FastAPI dashboards against floci's DuckDB engine.

## Rules for sending

- The sandbox project `herom-bq-sandbox-961172` has billing linked since 2026-10-13 (billing account `019C80-ADF43C-45891A`), so DML (`MERGE`, `INSERT`, `UPDATE`) runs there now; before that, DML failed with "DML queries are not allowed in the free tier".
- Commits as `herom-s <45332114+herom-s@users.noreply.github.com>`, no AI co-author trailers, Conventional Commit titles.
- PR limits (CONTRIBUTING.md): at most 4 open PRs per contributor, drafts included, and at most 2 of them ready for review. A bot closes a PR opened while 4 are already open. Open the next queued PR only when a slot frees; mark a draft ready only when a ready slot frees.
- Branch off current `upstream/main` (rebase before sending); each PR gets its own issue with a repro.
- Every behaviour claim is checked against real BigQuery first (sandbox project `herom-bq-sandbox-961172`; run `bq` with `CLOUDSDK_CORE_ACCOUNT=heromapp1@gmail.com`). The scripts and outputs live locally in `~/floci-json-evidence/`.
- Follow floci-gcp's AGENTS.md code style (the maintainer asked for it on #335). Before every push, check the diff for: no `var` (write the explicit type, including for-loops and try-with-resources); no inline fully-qualified names (`java.util.Arrays.asList` -> import `Arrays`); imports alphabetical with `java.*`/`javax.*` last and no wildcard imports in `src/main`; no tabs; no em-dashes anywhere, PR descriptions and commit messages included; never an empty `catch`; JBoss `LOG.debugf(...)`-style logging; switch expressions; test names as camelCase sentences. Allowed commit types: `feat`, `fix`, `perf`, `docs`, `chore`.
- Open PRs on 2026-10-14: #349 and #356 ready (2 of 2); #394 and #412 drafts (4 of 4 open). The next queued PR waits for a merge.
- Decision 2026-10-13: open an issue for every held fix now, and open their PRs one at a time as slots free, in the queue order below. The 4th slot went to the #380 fix (draft).

## Upstream PRs

| PR | Closes | Branch | Title | Status |
|---|---|---|---|---|
| [#307](https://github.com/floci-io/floci-gcp/pull/307) | [#305](https://github.com/floci-io/floci-gcp/issues/305) | `fix/bigquery-ndjson-json-load` | keep NDJSON load values as JSON values in JSON columns | merged 2026-10-05 |
| [#308](https://github.com/floci-io/floci-gcp/pull/308) | [#306](https://github.com/floci-io/floci-gcp/issues/306) | `fix/bigquery-json-type` | return BigQuery type names from JSON_TYPE | merged 2026-10-04 |
| [#317](https://github.com/floci-io/floci-gcp/pull/317) | none (related: #316) | `fix/bigquery-implicit-select-alias` | quote implicit select-list aliases | merged 2026-10-05 |
| [#330](https://github.com/floci-io/floci-gcp/pull/330) | [#316](https://github.com/floci-io/floci-gcp/issues/316) | `fix/bigquery-offset-identifier` | treat offset as an ordinary name | merged 2026-10-06 |
| [#325](https://github.com/floci-io/floci-gcp/pull/325) | none | `fix/bigquery-interval-expression` | accept expressions as interval step sizes | merged 2026-10-08 (4886f1b) |
| [#335](https://github.com/floci-io/floci-gcp/pull/335) | [#337](https://github.com/floci-io/floci-gcp/issues/337) | `fix/bigquery-null-repeated-as-empty` | return a NULL array as an empty array | merged 2026-10-09 (9f8a65a); #337 closed |
| [#336](https://github.com/floci-io/floci-gcp/pull/336) | [#338](https://github.com/floci-io/floci-gcp/issues/338) | `fix/bigquery-array-agg-ignore-nulls` | support null modifiers and LIMIT in ARRAY_AGG | merged 2026-10-13 (0a76f84); #338 closed; follow-up split out as issue #371 |
| [#347](https://github.com/floci-io/floci-gcp/pull/347) | [#344](https://github.com/floci-io/floci-gcp/issues/344) | `fix/bigquery-ctas-parenthesized-query` | run queries wrapped in parentheses and CTEs inside them | merged 2026-10-11 (c96e936), approved by avison9 and hectorvent; #344 closed; follow-up issue #380 |
| [#348](https://github.com/floci-io/floci-gcp/pull/348) | [#345](https://github.com/floci-io/floci-gcp/issues/345) | `fix/bigquery-ctas-destination-table` | report the DDL target table as the job destination | merged 2026-10-13 (4d7d918); #345 closed |
| [#349](https://github.com/floci-io/floci-gcp/pull/349) | [#346](https://github.com/floci-io/floci-gcp/issues/346) | `fix/bigquery-merge-insert-source-columns` | resolve bare columns in MERGE NOT MATCHED clauses to the source | ready for review (2026-10-13); rebased onto 532679c (head 1eda437, inline `Assertions.assertEquals` replaced with a static import), then 13be8b3 for Greptile's four P1s (typed literals and date parts read as columns, subqueries qualified with the outer source, the right side of `a < b` skipped, a `WHEN` inside `CASE` ending the clause), then 223e27d after the re-review still flagged dates (a date part that is not the last argument, `TIMESTAMP_TRUNC(ts, DAY, 'UTC')`, and the end of `INTERVAL '0-1' YEAR TO MONTH`), then 80e907c for Greptile's third pass (the date-part skip must hold only for the argument position that takes a part, so a column named `day` in `DATE_DIFF(d, day, DAY)` stays qualified), then a96eaf0 for Greptile's fourth pass (commas inside an array literal shifted that argument count); all checked with real `MERGE`s on BigQuery (sandbox now has billing); `bigquery` package 288 tests pass, the new tests fail without each fix; replied on each finding; 2026-10-14: hectorvent and avison9 found that words that are not columns got the source prefix (parenless `CURRENT_DATE`/`CURRENT_TIMESTAMP`, `AT TIME ZONE`, named arguments `name => x`); fixed in f9629a8 (`SESSION_USER` left out: BigQuery needs its parentheses); Greptile's sixth pass found an unaliased `USING UNNEST(...)` source qualified as `"UNNEST"`, fixed in d5649d0 (unaliased UNNEST rendered as `_merge_source`; aliased UNNEST left unqualified as on main); Greptile's seventh pass: the generated name could equal a target named or aliased `_merge_source`, fixed in 686fb95 (first unused of `_merge_source`, `_merge_source_1`, ...); rebased onto a27384b; `bigquery` package 247 tests pass, the changed tests fail without each fix; follow-ups went to issue #410 |
| [#394](https://github.com/floci-io/floci-gcp/pull/394) | [#380](https://github.com/floci-io/floci-gcp/issues/380) | `fix/bigquery-subquery-anonymous-columns` | name anonymous subquery and CTE columns like BigQuery | open, draft (2026-10-13), head e04c3a9; takes the 4th PR slot; translator + Duck IT pass, both new tests fail without the fix |
| [#412](https://github.com/floci-io/floci-gcp/pull/412) | [#371](https://github.com/floci-io/floci-gcp/issues/371) | `fix/bigquery-array-null-element` | fail the query when a result array has a NULL element | open, draft (2026-10-14), head ecfc58c, rebased onto a27384b; `bigquery` package 247 tests pass; its tests fail without the fix |
| [#356](https://github.com/floci-io/floci-gcp/pull/356) | [#261](https://github.com/floci-io/floci-gcp/issues/261) (filed by hectorvent) | `fix/bigquery-duckdb-external-access` | stop queries from reading files and URLs through DuckDB | ready for review (2026-10-14); rebased onto a27384b (head bbbf6ee, test file rebuilt from main plus the PR's additions after a conflict); 246 BigQuery tests pass, `queriesCannotReadFilesOrUrls` fails without the fix |

## Issues I opened

| Issue | Title | State | Fixed by |
|---|---|---|---|
| [#305](https://github.com/floci-io/floci-gcp/issues/305) | NDJSON load job rejects objects/arrays in JSON columns | closed 2026-10-05 | #307 |
| [#306](https://github.com/floci-io/floci-gcp/issues/306) | JSON_TYPE returns DuckDB type names | closed 2026-10-04 | #308 |
| [#316](https://github.com/floci-io/floci-gcp/issues/316) | select-list aliases that are DuckDB keywords fail | closed 2026-10-06 | #317, #330 |
| [#337](https://github.com/floci-io/floci-gcp/issues/337) | a NULL array is returned as null instead of [] | closed 2026-10-09 | #335 |
| [#338](https://github.com/floci-io/floci-gcp/issues/338) | ARRAY_AGG with IGNORE NULLS or RESPECT NULLS fails | closed 2026-10-13 | #336 |
| [#344](https://github.com/floci-io/floci-gcp/issues/344) | a query wrapped in parentheses fails; CTEs inside parentheses or subqueries read as tables | closed 2026-10-11 | #347 |
| [#345](https://github.com/floci-io/floci-gcp/issues/345) | DDL jobs have no configuration.query.destinationTable | closed 2026-10-13 | #348 |
| [#346](https://github.com/floci-io/floci-gcp/issues/346) | bare columns in MERGE WHEN NOT MATCHED fail as ambiguous | open | #349 (ready for review) |
| [#371](https://github.com/floci-io/floci-gcp/issues/371) | arrays with NULL elements are returned instead of failing the query | open, opened 2026-10-09 at hectorvent's request on #336 | #412 (draft) |
| [#380](https://github.com/floci-io/floci-gcp/issues/380) | anonymous columns of a subquery or CTE are named after their expression instead of `f0_` | open, opened 2026-10-13 at avison9's suggestion on #347 | #394 (draft) |
| [#381](https://github.com/floci-io/floci-gcp/issues/381) | OFFSET, ORDINAL and SAFE_ array subscripts are rejected | open, opened 2026-10-13 | `fix/bigquery-array-subscripts` (queued) |
| [#382](https://github.com/floci-io/floci-gcp/issues/382) | window frames bounded by CURRENT ROW fail | open, opened 2026-10-13 | `fix/bigquery-window-frame-current-row` (queued) |
| [#383](https://github.com/floci-io/floci-gcp/issues/383) | load jobs ignore or reject skipLeadingRows sent as a string | open, opened 2026-10-13 | `fix/bigquery-csv-skip-leading-rows` (queued) |
| [#384](https://github.com/floci-io/floci-gcp/issues/384) | GENERATE_DATE_ARRAY is not supported | open, opened 2026-10-13 | `fix/bigquery-generate-date-array` (queued) |
| [#385](https://github.com/floci-io/floci-gcp/issues/385) | .* on a STRUCT value is not expanded into its fields | open, opened 2026-10-13 | `fix/bigquery-struct-star-expansion` (queued) |
| [#386](https://github.com/floci-io/floci-gcp/issues/386) | INITCAP is not supported | open, opened 2026-10-13 | `fix/bigquery-initcap` (queued) |
| [#387](https://github.com/floci-io/floci-gcp/issues/387) | ANY_VALUE(x HAVING MAX y) and HAVING MIN are rejected | open, opened 2026-10-13 | `fix/bigquery-any-value-having` (queued) |
| [#388](https://github.com/floci-io/floci-gcp/issues/388) | an implicit alias after NULL, TRUE or FALSE is not recognized | open, opened 2026-10-13 | `fix/bigquery-alias-after-null-keyword` (queued) |
| [#389](https://github.com/floci-io/floci-gcp/issues/389) | IS [NOT] DISTINCT FROM fails | open, opened 2026-10-13 | `fix/bigquery-is-distinct-from` (queued) |
| [#390](https://github.com/floci-io/floci-gcp/issues/390) | FORMAT_* and PARSE_* reject %F, %D, %R and other format elements, and formats read from a column | open, opened 2026-10-13 | `fix/bigquery-format-date-shorthands` (queued) |
| [#391](https://github.com/floci-io/floci-gcp/issues/391) | a date-shaped STRING parameter can't be compared with DATE or TIMESTAMP | open, opened 2026-10-13 | `fix/bigquery-string-param-temporal-coercion` (queued) |
| [#392](https://github.com/floci-io/floci-gcp/issues/392) | an implicit alias after a query parameter makes the query fail | open, opened 2026-10-13 | `fix/bigquery-alias-after-parameter` (queued) |
| [#393](https://github.com/floci-io/floci-gcp/issues/393) | DATE_ADD and other date functions fail on STRING parameters | open, opened 2026-10-13 | `fix/bigquery-string-param-date-arithmetic` (queued) |
| [#395](https://github.com/floci-io/floci-gcp/issues/395) | ALTER TABLE ... SET OPTIONS is rejected | open, opened 2026-10-13 | `fix/bigquery-alter-table-set-options` (queued) |
| [#410](https://github.com/floci-io/floci-gcp/issues/410) | a correlated subquery in a MERGE WHEN NOT MATCHED clause fails with a DuckDB error instead of BigQuery's | open, opened 2026-10-14 as the follow-up hectorvent and avison9 asked for on #349 | `fix/bigquery-merge-correlated-source-columns` (queued after #349) |

Also: [#261](https://github.com/floci-io/floci-gcp/issues/261) (filed by hectorvent, claimed by me) is fixed by draft #356.

## Fixes waiting for a PR slot (pushed to this fork, issue open)

All rebased onto `upstream/main` @ `532679c` on 2026-10-13 and pushed. For each: the changed test classes pass with the fix, and its new tests fail with `src/main` reverted to `main` (run with `/tmp/held_tests.sh`). Inline fully-qualified names from the 2026-10-07 audit are replaced with imports; a style check of each diff is clean. Open PRs in this order as slots free (see the rules above); each issue already says a fix is ready.

| Queue | Branch | Head | Issue | Title | Tests | Evidence |
|---|---|---|---|---|---|---|
| 1 | `fix/bigquery-array-null-element` | `ecfc58c` | [#371](https://github.com/floci-io/floci-gcp/issues/371) | fail the query when a result array has a NULL element | opened as draft #412 on 2026-10-14 | `array-null-*.json` |
| 2 | `fix/bigquery-array-subscripts` | `b611394` | [#381](https://github.com/floci-io/floci-gcp/issues/381) | translate OFFSET, ORDINAL and SAFE_ array subscripts | 90 pass with the fix; 5 new tests fail on main | `gaps-2026-10-06.txt` |
| 3 | `fix/bigquery-window-frame-current-row` | `743cf06` | [#382](https://github.com/floci-io/floci-gcp/issues/382) | keep ROW a keyword in CURRENT ROW window frame bounds | 74 pass with the fix; 2 new tests fail on main | `gaps-2026-10-06.txt` |
| 4 | `fix/bigquery-csv-skip-leading-rows` | `a0bc82d` | [#383](https://github.com/floci-io/floci-gcp/issues/383) | accept skipLeadingRows sent as a string in load jobs | 11 pass with the fix; 1 new test fails on main | `gaps-2026-10-06.txt` |
| 5 | `fix/bigquery-generate-date-array` | `82c0a81` | [#384](https://github.com/floci-io/floci-gcp/issues/384) | support GENERATE_DATE_ARRAY | 74 pass with the fix; 2 new tests fail on main | `gaps-2026-10-06.txt` |
| 6 | `fix/bigquery-alter-table-set-options` | `f81ce7b` | [#395](https://github.com/floci-io/floci-gcp/issues/395) | support ALTER TABLE and ALTER VIEW SET OPTIONS | 72 pass with the fix; the new DML IT test fails on main (the unit tests need the fix to compile, so they were left out of that run) | `alter-2026-10-06-{a,b,c}.txt` |
| 7 | `fix/bigquery-struct-star-expansion` | `116645e` | [#385](https://github.com/floci-io/floci-gcp/issues/385) | expand STRUCT values with .* (unnest) | 74 pass with the fix; 2 new tests fail on main | `struct-star-2026-10-06.txt` |
| 8 | `fix/bigquery-initcap` | `9f44ada` | [#386](https://github.com/floci-io/floci-gcp/issues/386) | support INITCAP | 74 pass with the fix; 2 new tests fail on main | `initcap-2026-10-06.txt` |
| 9 | `fix/bigquery-any-value-having` | `d3bab16` | [#387](https://github.com/floci-io/floci-gcp/issues/387) | support ANY_VALUE with HAVING MAX and HAVING MIN | 74 pass with the fix; 2 new tests fail on main | `having-max-2026-10-06.txt` |
| 10 | `fix/bigquery-alias-after-null-keyword` | `0d1329a` | [#388](https://github.com/floci-io/floci-gcp/issues/388) | treat a name after NULL, TRUE or FALSE as an implicit alias | 74 pass with the fix; 2 new tests fail on main | `null-alias-2026-10-07.txt` |
| 11 | `fix/bigquery-is-distinct-from` | `1e50d58` | [#389](https://github.com/floci-io/floci-gcp/issues/389) | keep IS [NOT] DISTINCT FROM out of FROM clause handling | 74 pass with the fix; 2 new tests fail on main | `is-distinct-from-2026-10-07.txt` |
| 12 | `fix/bigquery-format-date-shorthands` | `7c8bd24` | [#390](https://github.com/floci-io/floci-gcp/issues/390) | support every format element and formats read from a column | 74 pass with the fix; 3 new tests fail on main | `format-*.json` |
| 13 | `fix/bigquery-string-param-temporal-coercion` | `640a706` | [#391](https://github.com/floci-io/floci-gcp/issues/391) | coerce date-shaped STRING parameters like BigQuery | 74 pass with the fix; 2 new tests fail on main | `string-param-*.json` |
| 14 | `fix/bigquery-alias-after-parameter` | `6d36c30` | [#392](https://github.com/floci-io/floci-gcp/issues/392) | keep an implicit alias that follows a query parameter | 73 pass with the fix; 2 new tests fail on main | `param-implicit-alias.json` |
| 15 | `fix/bigquery-string-param-date-arithmetic` | `5124218` | [#393](https://github.com/floci-io/floci-gcp/issues/393) | coerce STRING operands of date and time functions | 73 pass with the fix; 2 new tests fail on main | `string-param-date-arithmetic*.json` |
| 16 | `fix/bigquery-merge-correlated-source-columns` | `0fa1bae` | [#410](https://github.com/floci-io/floci-gcp/issues/410) | reject correlated subqueries in MERGE NOT MATCHED clauses like BigQuery | builds on #349 (branch is #349 plus one commit); send after #349 merges. Gives the translator the stored table schemas: only target columns get the source prefix (avison9's suggestion), and a subquery that reads the MERGE tables fails with `Correlated Subquery is unsupported in INSERT/WHEN clause.` `bigquery` package plus `DuckSqlEngineServiceTest`, 251 tests pass; the new integration test fails without the fix (on main it gets `Binder Error: Ambiguous reference to column name "k"`). | `corr-*.json` |

Notes for when these go up:

- The subscripts rewrite must run in `run()`/`runDml()`, not `prepare()`: `classify()` re-translates CREATE TABLE/VIEW text, and the rewrite is not idempotent.
- These branches all add tests to the same spots in `SqlDialectTranslatorTest` and `BigQueryDuck*IntegrationTest`, so each one will need a rebase after the one before it merges. On 2026-10-13 a "keep both sides" conflict resolution broke the test files (method bodies spliced into each other); they were rebuilt member by member (`/tmp/member_merge.py`: upstream members plus the branch's added or changed members, in the branch's order). Compile the tests after any rebase.
- The #371 PR body is ready in `~/floci-json-evidence/pr-bodies/371-array-null-element.md` (local). Write the others from the issue text when their turn comes.

## `local/all-fixes`

Integration branch: `upstream/main` (merged 2026-10-08 at `362809b`, which brings #294, the Firestore nested update mask fix bob's `update_source` needs) plus every PR branch above and all 14 held fixes, merged. It is what the local `floci-gcp:local` image is built from (`docker build -f docker/Dockerfile -t floci-gcp:local .`). With it, bob-datahub-sync's dbt project builds 166/166 models and all of its `/ads/*` dashboard endpoints answer. Rebuild it from the branches; don't send it upstream.

Stale since 2026-10-13: the held fixes were rebased and their heads changed (see the queue table), and the #380 and #371 fixes are not in it. Rebuild before using it again. Status 2026-10-09: up to date at `50101b7`. It has `upstream/main` `87b4f26` (so #335 as merged, #358, #361, #363), the current heads of #336 (`0a8619d`), #347 (`1c7515e`), #348, #349 and #356 (merged in for the first time), and all 14 held fixes; the subscripts fix is now the final `c8ce5ac` with its two CTAS regression tests. BigQuery, Firestore and Secret Manager tests: 425 passed. The `floci-gcp:local` image has not been rebuilt from it yet.

## Known, not fixed yet

- A MERGE source written `USING UNNEST(...) AS u` (aliased): BigQuery resolves bare struct fields (`v`, `id`) to it (`merge-unnest-alias.json`); floci renders it as one STRUCT column `"_unnest_u"("u")`, and qualifying `"u".id` hits a DuckDB internal binder error, so #349 leaves it unqualified as on main. Not reported yet.
- Found while fixing #371 and #380 (2026-10-13), not reported yet:
  - `ARRAY_TO_STRING(arr, sep, null_text)` (three arguments) fails with a DuckDB binder error.
  - BigQuery renames a repeated named result column (`SELECT x, * FROM (SELECT 3 AS x)` gives `x`, `x_1`); floci fails with the duplicate column error.
  - BigQuery rejects an anonymous column in `CREATE TABLE ... AS SELECT` / `CREATE VIEW` ("columns must be named"); floci accepts it.
  - BigQuery rejects a NULL array element written by `INSERT ... VALUES` / `INSERT ... SELECT` (`Array cannot have a null element; error in writing field a at [1:62]`); the #371 fix covers query results and CTAS only. Evidence `array-null-insert*.json`.
- PARSE_* with a format read from a column: DuckDB requires a constant strptime format and there is no per-row split for parsing. Literal and parameter formats work.
- PARSE_* with elements DuckDB lacks (`%e`, `%k`, `%Q`, ...): only the composites are expanded for parsing.
- FORMAT_TIMESTAMP's time zone argument is ignored (pre-existing); `%z`/`%Ez` print the UTC offset.