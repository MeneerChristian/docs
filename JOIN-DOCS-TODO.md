# Join index and JOIN docs: what the docs team has to decide or finish

Written next to the pages (docs repo, uncommitted). Sources read: `/Volumes/GoldenGate/ravendb/.agents/proposal/PROPOSAL.md`, `NUMBERS.md`, `.agents/issues/01-09`, the code on the branches `join/query` (3d6f942159), `join/index` (dd1efaa235), `join-fix-smallhw` (f88ee9188b), `join/indirect` (f6f9a8943d, same `JoinConfiguration.cs` as smallhw), `join/studio` (5880ee9948).

Pages written or changed:

| File | State |
|---|---|
| `docs/client-api/session/querying/how-to-join-indexes.mdx` + `content/_how-to-join-indexes-csharp.mdx` | new (RQL JOIN, client API); this pass added the id(main) join, the memory bound, counters, see also |
| `docs/indexes/join-indexes.mdx` + `content/_join-indexes-csharp.mdx` | new, rewritten this pass (storage modes, Auto rule, auto join index, Studio, small machines, settings) |
| `docs/server/configuration/join-configuration.mdx` | new: every `Join.*` key |
| `docs/server/configuration/query-configuration.mdx` | `Query.JoinMaxKeysMemoryInMb` added |
| `docs/querying/rql/what-is-rql.mdx`, `docs/indexes/search-engine/corax.mdx`, `.../_query-timings-csharp.mdx`, `.../_sort-query-results-csharp.mdx`, `.../_using-analyzers-csharp.mdx` | JOIN section, limits, counters, `FieldIndexing.Key` (earlier sessions; memory bound and counters added this pass) |
| `docs/studio/database/indexes/indexes-list-view.mdx`, `docs/indexes/content/_indexing-related-documents-csharp.mdx` | cross-links (this pass) |

## 1. Decisions

- [ ] **Studio help button: hash / page id (human step, exact request).** Please create a ravendb.net short link (the `https://ravendb.net/l/<HASH>` that `useRavenLink({ hash })` opens) that points to the docs page path `indexes/join-indexes` with the anchor `#creating-a-join-index-in-studio` (full page: `docs/indexes/join-indexes.mdx`, section "Creating a join index in Studio"). Then, in the Studio branch `join/studio`, replace the interim hash `XZXJP2` for the key `"databases/indexes/join(/:indexName)"` in `src/Raven.Studio/typescript/components/utils/docsHashes.ts` (line 20; `XZXJP2` at lines 17 and 18 stays for new/edit index) with the new hash. `docsHashes.spec.ts` (line 13) only asserts it is not `MISSING_DOCS`. Detail: the interim hash is in `docsHashes.ts` lines 17, 18, 20 and `viewmodels/database/indexes/EditIndexInfoHub.tsx` line 26.
- [ ] **Name.** NUMBERS.md `PENDING_NAME`: "join index" is the working name, the issue README recommends "join index". The pages use "join index" (file `join-indexes.mdx`, sidebar label "Join Indexes"). The page `how-to-join-indexes.mdx` is titled "Join Indexes and Add the Score of the Joined Query", which clashes with the name "join index". Rename together when the name is decided (also `JoinDefinition`, `AbstractJoinCreationTask`, `Join.*` keys are fixed in code).
- [ ] **Version label.** The pages do not say in which RavenDB version JOIN and the join index appear (other pages say "7.2.6 and later" for similar items). The cluster command version is 72_002 (`src/Raven.Server/ServerWide/ClusterCommandsVersionManager.cs` lines 228 and 229). Decide the release, add "Introduced in" notes, and decide whether `versioned_docs/` gets anything (nothing was written there; `docs/` is 7.2).
- [x] **Sidebar position of the configuration page.** Set to `25`, the only free whole number after Indexing (10); every position 0 to 28 is used except 3 and 25.
- [ ] **How much to say about `Indirect`.** The pages say it is opt-in and not released (`Join.IndirectEnabled` false, description in code says "It is not released"). Decide whether the docs describe an unreleased mode at all, or only the Auto rule after release. If dropped: Storage modes panel, the `StorageMode` row, the Indirect columns in "Measured numbers", `Join.StorageModeDefault`, `Join.IndirectEnabled`, `Join.AutoIndirectMinRowsPerKey`, `Join.ResidentCacheMaxMemoryInMb`, `Join.MaxResidualKeys`, resident statistics.
- [ ] **Auto join index in the first release.** `Join.AutoEnabled` default false (decided 2026-11-05, NUMBERS.md section 8). NUMBERS.md `PENDING_AUTO_JOIN_DELIVERABLE`: the auto join index has no deliverable token. Decide whether the auto join index section stays in the page or is hidden until it is requested.
- [ ] **Studio screenshots.** The Studio section is text only, taken from `src/Raven.Studio/typescript/common/i18n/locales/en/joinIndex.ts` and the components. Take screenshots of the New join index form, the statistics panel and the list view from a running Studio, then add them (pattern: `LazyImage` / `![...](./assets/...)` as in `indexes-list-view.mdx`). No GUI was started for this pass.
- [ ] **Other client languages.** Only C# is documented (`supported_languages: ["csharp"]`). The JOIN has a C# client (`Join`, `JoinAlias`, ...) and RQL. Decide whether the Java, Node.js, Python and PHP pages get an RQL-only version.
- [ ] **License / edition.** In code a join index counts as a static index and an auto join index as an auto index for the license limits (`src/Raven.Server/ServerWide/ClusterStateMachine.License.cs` lines 95 to 103). The pages say so. No feature flag for joins was found; check whether the Community/Professional/Enterprise tables need a row.
- [ ] **Static agent skill.** `static/skills/ravendb/` was not touched. Check whether its RQL and index references should mention JOIN and join indexes.

## 2. Things in the pages that could not be verified

- [x] **Auto join index removal when idle** (closed by reading the code). An auto join index is stored in `AutoIndexes` with the `Auto/` prefix, and `ClusterObserver.GetUnusedAutoIndexes` (`src/Raven.Server/ServerWide/Maintenance/ClusterObserver.cs` lines 582 to 669, branch `join/indirect`) marks it idle after `Indexing.TimeToWaitBeforeMarkingAutoIndexAsIdleInMin` (30 min, `IndexingConfiguration.cs` line 194) and deletes it after `Indexing.TimeToWaitBeforeDeletingAutoIndexMarkedAsIdleInHrs` (72 h, line 207) without a query; a query that reads it calls `MarkUsedByQuery` (`Index.cs` line 3987), which promotes it from idle. The Studio string is therefore correct; the page now states the rule.
- [x] **`Join.MaxIdsPerCommit` and partial generations** (closed). `JoinServing.Compare` (`src/Raven.Server/Documents/Indexes/Joins/Serving/JoinServing.cs` lines 333 to 352) returns `Behind` for a partial generation (any partial one for Expanded, only an inner cut for Indirect), so a partial generation does not serve reads: the query waits, then falls back (`BehindTimeout`). Issue 06 wording ("so a reset ... does not block reads") means the reads fall back to the server join. The page says so.
- [x] **Studio list view** (closed). The list (`IndexPanel.tsx`, branch `join/studio`) shows type and the usual state, an Edit link to the join index form, and hides Query and Entries for a join index (lines 186, 340, 555). Pending reason / "Ready" is in the database statistics view (`IndexesDatabaseStats.tsx` line 113); `RefusedReason` and `PendingReason` are in `JoinStatsPanel.tsx` lines 38 to 47. Issue 07 scope (state, mode, refusal reason in the list) is not implemented; the page is corrected.
- [x] **Query-time memory of a served join** (closed). A served join holds no sub-query keys, except the check of `and`/`not` over rows with several documents: `JoinServing.cs` lines 122 to 135 call `request.ScanKeys` (`CoraxIndexReadOperation.ScanTermScoresAmong`, line 904), which holds the keys of those rows in native memory (`new JoinTermScores`, not `Bounded`), limited by `Join.MaxResidualRows` / `Join.MaxResidualKeys`, not by `Query.JoinMaxKeysMemoryInMb`. Stated in the page and in the Query configuration entry.
- [x] **C# snippets compiled** (not run). 30 snippets of the two pages (24 of the JOIN page without the signature listing, 6 of the join index page) compile against `Raven.Client.dll` of the worktree `/Volumes/GoldenGate/ravendb-p9-join-indirect` (0ff4d92bca, Release, net10.0) in a scratch project outside both repos. One failure fixed: inside an index class the `SearchEngineType` property hides the enum, so `SearchEngineType = SearchEngineType.Corax` does not compile in the join index page (3 index classes); they now use `Raven.Client.Documents.Indexes.SearchEngineType.Corax`, as the JOIN page does. Not run against a server (no embedded server started).
- [x] **MDX** (partly). `npm ci --offline` worked from the cache; all 13 changed MDX files compile with `@mdx-js/mdx`. The full `npm run build` of the site was not run.
- [ ] **`JoinStorageMode` description in code.** `Join.StorageModeDefault` says Auto chooses Indirect "for a join index on a field", and the `JoinStorageMode.Indirect` comment says "Only for a join on a field of the inner index". `JoinOptions.IndirectPossible` (`src/Raven.Server/Documents/Indexes/Joins/Refresh/JoinOptions.cs` line 92) is `JoinsOnOuterId == false`, so Auto also applies to a join on the id of the inner document. The page follows the code. Fix the code comments or the page.

## 3. Statements in NUMBERS.md and in the issue texts that the code or the pages contradict

- [ ] NUMBERS.md `F3_FLOOR_1GB_MB` says the gate floor is `min(1024 MB, total/4)` = 256 MB at 1 GB. The code at `join-fix-smallhw` is `min(2560 MB, total/8)` (`JoinConfiguration.cs` line 27), 128 MB at 1 GB. The pages use the code. NUMBERS.md and PR-JOIN-INDEX.md should be updated.
- [ ] Issue 05 "Known limits" says there is no projection of fields of the joined document and that a `where` comparing a main field with a joined field is refused. The code and the tests on `join/query` support both (`JoinProjectionTests`, `JoinCompareFieldsTests`), and the JOIN page documents them. Fix the issue text.
- [ ] Issue 09 says Auto never builds an id join as Indirect and that the rule is not implemented. The code has the rule (`JoinOptions.ResolveBuildMode`, lines 104 to 148). NUMBERS.md `PENDING_AUTO_RULE_IMPLEMENTED` still reads pending.
- [ ] Issue 06 and the old join index page said a join on `id(o) = i.B` is not served. It is served (`JoinOnId`, `JoinDefinition.JoinsOnOuterId`, tests `JoinIndexIdOfOuterQueryTests`), unless the query projects the joined alias. The page now says so.
- [ ] The old join index page listed `Join.MaxQueuedListChanges` as "never shorter than 1048576" and `Join.MaxQueuedListChangesInBuild` default `1048576`. Both follow the memory now (see the table in the configuration page). `Join.MinQueuedListChanges`, `Join.MaxResidualKeys`, `Join.StorageModeDefault`, `Join.IndirectEnabled`, `Join.AutoIndirectMinRowsPerKey` and `Join.ResidentCacheMaxMemoryInMb` were missing.

## 4. Numbers that were removed because they are not in NUMBERS.md

Only numbers from NUMBERS.md are in the pages (measured or derived, with the setup sentence). These figures from earlier sessions were removed. Restore them only with a source:

- Join index page, "Measured cost" table: 405 and 840 queries/s (9.9 and 4.8 ms CPU), 1,072 and 1,366 queries/s (left join), 148 and 706 queries/s (array of ids). They are in issue 06's evidence table (busy host, load 4 to 25, 3 runs of 20 s) but not in NUMBERS.md.
- JOIN page, "Compare fields": 200,000 placements and candidates, about 190 ms without the comparison, 500 to 590 ms for 100,000 matching rows, about 340 ms with `p.Status = 'open'`.
- JOIN page, "Left join and boost": 200,000 documents per index, 12 ms without a join, about 80 ms scored left join (+0.9 MB), about 35 ms with `boost 0`.
- JOIN page, "Performance": about 543 ms (and 21 ms for the outer query alone) for an id join on 1,000,000 users and games with 100,000 matching outer rows; about 1.5 times for `or` against `and` at 200,000 documents; 10 ms against 160 to 280 ms (and about 300 ms) for one or two fields of a join on several fields; the 1 CPU / 512 MB container remark.
- The qualitative statements that stayed in "Performance" (cost does not grow with the page size, a selective sub-query is the main lever, a field join on a low-cardinality field is cheaper than an id join, the ordered walk conditions) have no number, and no source in NUMBERS.md. The constants of the ordered walk (`skip + take` at most 10,000, at least 8,192 matched documents) were verified in `src/Raven.Server/Documents/Queries/JoinOrderedWalkSettings.cs` lines 7 and 10. Verify the rest by a benchmark or drop them.
- NUMBERS.md `PENDING_*` items still open that touch the pages: the benchmarks were not repeated on the final head (`PENDING_BENCH_FINAL_HEAD`), small-hardware suites and 1M under `small` (`PENDING_SUITES_SMALL`), why the resident cache ended empty under `small` (`PENDING_B2_EMPTY_CACHE_SMALL`), the fingerprint for one-key rows (`PENDING_B1_ONEKEY_FINGERPRINT`). The pages say "not repeated on the final version" and "cause is not isolated".

## 5. Verification record

### 5.1 Configuration keys and defaults (code, line numbers of the file as of the named commit)

`src/Raven.Server/Config/Categories/JoinConfiguration.cs` at `join-fix-smallhw` f88ee9188b (same file content at `join/indirect` f6f9a8943d). `join/index` dd1efaa235 has the 13 keys that exist there with the same defaults, scopes and minimums (compared by script); the Auto keys come with `join/auto`, the Indirect keys with `join-fix-smallhw`/`join/indirect`.

| Key | Default in the page | Code line(s) | Scope |
|---|---|---|---|
| `Join.MaxIdsPerCommit` | 50000, min 1 | 43 to 47 (44 default, 45 min) | server-wide or per database |
| `Join.MaxUnindexedIdsPerCommit` | 1000000, min 1 | 49 to 54 (51) | same |
| `Join.MaxInnerListChangesPerOuterEntry` | 30, min 0 | 56 to 61 (58) | same |
| `Join.MaxQueuedListChanges` | 4194304, min 1024 | 63 to 67 (64) | same |
| `Join.MinQueuedListChanges` | by memory: max(1024, min(128 MiB, total/128)/128 B) | 69 to 74 (71), constructor 39 to 40 | same |
| `Join.MaxQueuedListChangesInBuild` | by memory: clamp(total/16/128 B, 1024, 1048576) | 76 to 80 (77), constructor 31 | same |
| `Join.MinAtomEntriesForDirectWrite` | 1, min 0 | 82 to 87 (84) | same |
| `Join.MaxEmptyTermRatio` | 0.5 | 89 to 92 (90) | same |
| `Join.MinTermsForEmptyRatio` | 100000 | 94 to 97 (95) | same |
| `Join.RefreshWindowInMs` | 0 | 99 to 105 (102) | same |
| `Join.MaxResidualRows` | 100000 | 107 to 111 (108) | same |
| `Join.MaxResidualKeys` | 20000 | 113 to 117 (115) | same |
| `Join.QueryWaitInMs` | 100 | 119 to 123 (121) | same |
| `Join.QueryWaitForNonStaleInMs` | 1000 | 125 to 129 (127) | same |
| `Join.AutoEnabled` | false | 131 to 136 (134) | same |
| `Join.AutoMinimumAvailableMemoryInMb` | by memory: min(2560 MB, total/8) | 138 to 143 (140), constructor 27 | same |
| `Join.AutoMaxBuildMemoryInMb` | by memory: min(512 MB, total/2) | 145 to 149 (146), constructor 37 | same |
| `Join.AutoBuildMemoryBaseInMb` | 192 | 151 to 155 (152) | same |
| `Join.AutoBuildMemoryPerEntryInBytes` | 8 | 157 to 160 (158) | same |
| `Join.AutoBuildWaitForNonStaleInMs` | 15000 | 162 to 166 (163) | same |
| `Join.StorageModeDefault` | Auto | 168 to 173 (171) | same |
| `Join.ResidentCacheMaxMemoryInMb` | by memory: min(256 MB, total/16) | 175 to 180 (177), constructor 34 | **server-wide only** (179) |
| `Join.AutoIndirectMinRowsPerKey` | 30 | 182 to 185 (183) | same as the first rows |
| `Join.IndirectEnabled` | false | 187 to 190 (188) | same as the first rows |
| `Query.JoinMaxKeysMemoryInMb` | by memory: min(512 MB, max(1, total/8)), min 1 | `src/Raven.Server/Config/Categories/QueryConfiguration.cs` at `join/query` 3d6f942159: attributes 37 to 44 (min 41, key 43), constructor 20 to 24 (23) | **server-wide only** (43) |

Other code facts used (branch `join-fix-smallhw` unless stated):

- Memory defaults use `MemoryInformation.TotalPhysicalMemory` (`src/Sparrow.Server/LowMemory/MemoryInformation.cs` line 66), which honours the cgroup limit (lines 390 to 397): the "container limit" statements.
- The queue floor and the quarter-of-available-memory rule: `Join/Refresh/JoinIncrementalRefresh.cs` lines 552 to 564; the build uses the same limit: `Join/JoinRefreshWork.cs` line 221.
- Auto rule: `JoinOptions.cs` lines 95 to 148 (`ChoosesModeAtBuild`, `ResolveBuildMode`); cache estimate constants 69 B/key and 4.5 B/row: lines 82 to 84, 90.
- Auto join index: `Join/Auto/AutoJoinManager.cs` (gate lines 230 to 259, retry after 30 s line 38, cluster-version check line 114, wait budget line 76); eligibility `AutoJoinShape.cs` lines 28 to 44, prefix `Auto/` line 20.
- Definition limits: `src/Raven.Client/Documents/Indexes/JoinLimits.cs` (8 fields, 4 groupings, name 206, grouping value 128, 32 terms per entry, 256 keys per entry); validator `JoinIndexValidator.cs` lines 34 to 56 (Indirect needs the setting, sharded refused, collections refused, same index refused); rules `JoinIndexRules.cs`.
- Fallback reasons: `Join/Serving/JoinFallbacks.cs`; statistics: `src/Raven.Client/Documents/Indexes/IndexStats.cs` lines 212 to 310.
- Memory bound error text: `src/Raven.Server/Documents/Queries/JoinSources.cs` lines 177 to 182 at `join/query`; the test `test/FastTests/Corax/JoinKeysMemoryLimitTests.cs` shows the bound counts the key bytes plus 8 bytes per key.

### 5.2 RQL examples

All 23 RQL join examples in the three pages were parsed with the real `QueryParser` and AST sources taken from `join/query` (3d6f942159): `Raven.Server.Documents.Queries.Parser.QueryParser` compiled in a scratch project (outside both repos) against `Sparrow` and `Raven.Client` of the same commit; 23 parsed, 0 failed. Four negative checks fail as the pages say (`left outer join`, `limit` in the sub-query, `>` in `on`, `boost` without a value). The parser proves the syntax only. For behaviour, each example has a counterpart in the tests of `join/query` (the tests use Games/Users/Holders/Items indexes, so no query is identical word for word):

| Example | Test counterpart (`test/FastTests/...`) |
|---|---|
| id join, `search()` sub-query, `where` on main, `order by score()`, `limit` | `Corax/JoinLeftTests.cs` (line 127, 139), `Corax/JoinCombinedTests.cs` (lines 29 to 37) |
| join by field value (`o.F = c.F`) | `Corax/JoinCombinedTests.cs` line 36 (`g.Postcode = u.Postcode`), `Corax/JoinFieldScanTests.cs`, `Server/Documents/Queries/FieldJoinPlainJoinTests.cs` |
| join on several fields | `Corax/JoinCompositeKeyTests.cs` |
| two joins, chained join | `Corax/JoinLeftChainTests.cs`, `Corax/JoinChainMatchesTests.cs` (`matches(f)`, `PlacementsListIndex`/`FileInfoTextIndex` text) |
| `order by c.Country, o.Freight as long` | `Corax/JoinInnerSortTests.cs`, `Corax/JoinInnerSortScoreTests.cs` |
| `where` on an alias with `or`, `matches(c) or matches(e)` | `Corax/JoinOrTests.cs`, `Corax/JoinCombinedTests.cs`, `Corax/JoinChainMatchesTests.cs` |
| `where p.Salary > c.MaxSalary` and with `or` | `Corax/JoinCompareFieldsTests.cs`, `Corax/JoinServerHardeningTests.cs` (`g.Rank > u.Limit`) |
| `left join`, `boost 0`, `boost $weight` | `Corax/JoinLeftTests.cs`, `Corax/JoinClientLeftBoostTests.cs`, `Corax/JoinClientLanguageTests.cs` lines 401 to 454 |
| `select ... c.Name as CompanyName` | `Corax/JoinProjectionTests.cs`, `Corax/JoinKeyCaseTests.cs` |
| `select facet(...)` over a join | `Corax/JoinFacetTests.cs`, `Corax/JoinFacetVariantTests.cs` |
| `on id(c) = o.Company` (id of the main document) | `Server/Documents/Queries/FieldJoinIdOfOuterTests.cs` (`id(h) = i.Owner`, projection of the array, rejections, `Exact` message "keeps the case of its values", chain message, "JOIN is supported only for queries on an index") |
| memory bound example | `Corax/JoinKeysMemoryLimitTests.cs` (the same query text) |
| join index page: field join and id join with a declared field | `FastTests/Server/Documents/Indexing/Joins/Query/*` (branch `join-fix-smallhw`), `JoinIndexIdOfOuterQueryTests.cs` |

Not run by this pass: any test (nothing in `/Volumes/GoldenGate/ravendb` was built or changed; the parser probe was built from a `git archive` copy in the scratchpad).
