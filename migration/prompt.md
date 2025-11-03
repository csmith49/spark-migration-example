Migrate the following Spark programs from Spark 2.4 to Spark 3:

<path/to/spark/job/1>
<path/to/spark/job/2>
<path/to/spark/job/3>

Follow the steps given below.

### Phase 1: Discovery
1. Examine structured checklist of breaking changes (migration/checklist.md)
2. Scan codebase for:
   - Spark imports and API usage patterns
   - Configuration files (spark-defaults.conf, etc.)
   - Build files
3. Generate initial migration assessment explaining necessary changes

### Phase 2: Preparation
#### If tests exist:
- Run full test suite on Spark 2.4, record results
- Identify flaky tests (may hide real migration issues)

#### If no tests exist:
- Identify representative code paths
- Create synthetic inputs covering:
  * Basic happy path
  * Null/empty data
  * Type edge cases (dates pre-1582, mixed types)
  * Operations affected by migration (see checklist for examples)
- Capture output snapshots (Parquet format for determinism)
- Document any non-deterministic operations

### Phase 3: Automated Fixes
1. Identify and apply applicable ast-grep rules (migration/rules)
2. Write ast-grep rules for remaining syntactic changes:
   - Start with high-confidence patterns (deprecated → new API)
   - Test each rule on isolated examples before applying
   - Document patterns that cannot be automated (in migration/exceptions.md)
3. Apply rules, commit changes and new rules

### Phase 4: Manual Fixes
- Address patterns not covered by ast-grep:
  * Semantic behavior changes (calendar system)
  * Configuration-driven changes
- Update dependencies (Spark 2.4.x → 3.0.x)
- Update configs for removed/changed spark.sql.* properties

### Phase 5: Verification
- Run tests (original or snapshot)
- For snapshot diffs: verify they're expected migration effects
- Static analysis (ensure code builds/lints/typechecks)
- Document remaining manual review items

### Phase 6: Report
Generate markdown report including:
- Migration checklist with status (✅ automated, ⚠️ manual, ❌ blocked)
- Code change summary (files modified, LOC changed)
- Test results (pass/fail/skip counts, notable diffs)
- Risk items requiring human review
- Estimated confidence level (High/Medium/Low)

