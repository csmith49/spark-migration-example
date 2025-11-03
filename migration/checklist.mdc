# Spark 2.4 to 3.0 Migration Checklist

This checklist covers the key changes and potential breaking changes when migrating from Apache Spark 2.4 to 3.0. Each item includes the change description and configuration options to restore legacy behavior where applicable.

**📖 For a complete and detailed list of all changes, refer to the official Apache Spark documentation:**  
https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-24-to-30

## 🔧 Configuration and Environment

### Hive Upgrade (Major Change)
- [ ] **Hive upgraded from 1.2 to 2.3**
  - Update `spark.sql.hive.metastore.version` and `spark.sql.hive.metastore.jars` if connecting to older Hive metastore
  - Migrate custom SerDes to Hive 2.3 or build Spark with `hive-1.2` profile
  - Test decimal string representation in TRANSFORM operations (may have trailing zeros)

### Session Configuration
- [ ] **SparkSession cloneSession() behavior changed**
  - Parent SparkSession configs now take precedence over SparkContext configs
  - **Legacy config**: `spark.sql.legacy.sessionInitWithConfigDefaults=true`

## 📊 Dataset/DataFrame APIs

### API Behavior Changes
- [ ] **unionAll() is no longer deprecated** - now alias for union()
- [ ] **groupByKey naming changed**
  - Key attribute now named "key" instead of "value" for non-struct types
  - **Legacy config**: `spark.sql.legacy.dataset.nameNonStructGroupingKeyAsValue=true`

### Column Metadata
- [ ] **Column metadata propagation changed**
  - Metadata always propagated in `Column.name` and `Column.as`
  - Use `as(alias: String, metadata: Metadata)` for explicit metadata control

### Type Casting
- [ ] **Stricter Dataset type casting**
  - String to other types (like Boolean) no longer allowed
  - **Legacy config**: `spark.sql.legacy.doLooseUpcast=true`

## 🏗️ DDL Statements

### Table Operations
- [ ] **ANSI SQL type coercion for table inserts**
  - Stricter type conversions (string to int, double to boolean disallowed)
  - Out-of-range values throw exceptions instead of truncating
  - **Legacy config**: `spark.sql.storeAssignmentPolicy=LEGACY`

### Reserved Properties
- [ ] **New reserved table/database properties**
  - `provider`, `location`, `owner` are now reserved
  - Use specific clauses instead of TBLPROPERTIES/DBPROPERTIES
  - **Legacy config**: `spark.sql.legacy.notReserveProperties=true`

### Command Changes
- [ ] **ADD JAR command** - now returns empty result set instead of single value 0
- [ ] **SET command** - fails on SparkConf keys instead of silently ignoring
  - **Legacy config**: `spark.sql.legacy.setCommandRejectsSparkCoreConfs=false`
- [ ] **ADD FILE command** - now supports directories, not just single files
  - **Legacy config**: `spark.sql.legacy.addSingleFileInAddFile=true`

### Table Schema
- [ ] **CHAR type no longer allowed** in non-Hive-SerDe tables - use STRING instead
- [ ] **SHOW CREATE TABLE** - always returns Spark DDL, use `AS SERDE` for Hive DDL

## 🔢 UDFs and Built-in Functions

### Function Parameter Validation
- [ ] **date_add/date_sub functions**
  - Only accept int/smallint/tinyint as 2nd argument
  - String literals still allowed but must be valid integers
- [ ] **percentile_approx/approx_percentile**
  - 3rd argument must be integral value in [1, 2147483647]

### Function Behavior Changes
- [ ] **array()/map() functions** - return NullType elements when called without parameters
  - **Legacy config**: `spark.sql.legacy.createEmptyCollectionUsingStringType=true`
- [ ] **Map functions with duplicated keys** - now throw RuntimeException
  - **Legacy config**: `spark.sql.mapKeyDedupPolicy=LAST_WIN`
- [ ] **Hash expressions on MapType** - now throw AnalysisException
  - **Legacy config**: `spark.sql.legacy.allowHashOnMapType=true`

### JSON Processing
- [ ] **from_json function** - default mode changed to PERMISSIVE
  - Malformed JSON handling changed (e.g., `{"a" 1}` → `Row(null)` instead of `null`)

### UDF Changes
- [ ] **Scala UDF with DataType parameter** - not allowed by default
  - **Legacy config**: `spark.sql.legacy.allowUntypedScalaUDF=true`
- [ ] **Higher-order function exists()** - follows three-valued logic
  - **Legacy config**: `spark.sql.legacy.followThreeValuedLogicInArrayExists=false`

### Date/Time Functions
- [ ] **add_months function** - no longer adjusts to last day of month
- [ ] **current_timestamp** - may return microsecond resolution instead of millisecond
- [ ] **Math functions** - now use StrictMath for consistency across platforms

### String Casting
- [ ] **CAST function** - case-insensitive handling of special values
  - 'Infinity', 'NaN', 'Inf' variants now properly cast to Double/Float
- [ ] **Interval casting** - no "interval" prefix in string representation
- [ ] **String to numeric/datetime casting** - trims leading/trailing whitespace (≤ ASCII 32)

## 🔍 Query Engine

### SQL Syntax
- [ ] **Invalid FROM syntax** - `FROM <table>` without SELECT no longer supported
- [ ] **Interval literal syntax** - multiple from-to units no longer allowed
- [ ] **Scientific notation** - parsed as Double instead of Decimal
  - **Legacy config**: `spark.sql.legacy.exponentLiteralAsDecimal.enabled=true`

### Data Types
- [ ] **Negative decimal scale** - not allowed by default
  - **Legacy config**: `spark.sql.legacy.allowNegativeScaleOfDecimal=true`
- [ ] **Unary plus operator (+)** - stricter type checking and coercion

### Join Behavior
- [ ] **Self-join ambiguous columns** - now fails instead of returning empty results
  - **Legacy config**: `spark.sql.analyzer.failAmbiguousSelfJoin=false`
- [ ] **Cross joins** - `spark.sql.crossJoin.enabled` is now true by default

### CTE (Common Table Expressions)
- [ ] **CTE precedence policy** - inner definitions take precedence
  - **Legacy config**: `spark.sql.legacy.ctePrecedencePolicy=LEGACY`

### Numeric Handling
- [ ] **Float/double -0.0 vs 0.0** - now treated as same value in grouping/joins
- [ ] **Invalid time zones** - now throw DateTimeException instead of defaulting to GMT

### Calendar System
- [ ] **Proleptic Gregorian calendar** - major change affecting dates before Oct 15, 1582
  - Affects parsing, formatting, date arithmetic, and timestamp operations
  - Impacts CSV/JSON datasources and datetime functions
  - **Legacy configs available for specific operations**

### EXTRACT Function
- [ ] **EXTRACT second field** - returns DecimalType(8,6) instead of IntegerType

### Date Pattern
- [ ] **Pattern letter 'F'** - now "aligned day of week in month" instead of "week of month"

## 📁 Data Sources

### Schema Inference
- [ ] **Hive SerDe tables** - Spark no longer infers and updates schema in metastore
  - **Legacy config**: `spark.sql.hive.caseSensitiveInferenceMode=INFER_AND_SAVE`

### Partition Validation
- [ ] **Partition column validation** - now validates against user schema instead of converting to null
  - **Legacy config**: `spark.sql.sources.validatePartitionColumns=false`

### File Listing
- [ ] **Missing files during listing** - now fails unless configured otherwise
  - **Config**: `spark.sql.files.ignoreMissingFiles=true` (default false)

### JSON Data Source
- [ ] **Empty string handling** - now throws exception for non-string/binary types
  - **Legacy config**: `spark.sql.legacy.json.allowEmptyString.enabled=true`
- [ ] **Bad JSON records** - may contain non-null fields in PERMISSIVE mode
- [ ] **Timestamp inference** - infers TimestampType from strings matching timestampFormat
  - **Config**: Set JSON option `inferTimestamp=false` to disable

### CSV Data Source
- [ ] **Malformed CSV handling** - may contain non-null fields in PERMISSIVE mode
- [ ] **Encoding detection** - no longer automatic, uses UTF-8 by default
  - **Config**: Set CSV option `encoding=null` for auto-detection

### Avro Data Source
- [ ] **Schema matching** - fields matched by name instead of position
- [ ] **Non-nullable schema** - can write nullable catalyst data but throws NPE on null records

## 🔧 Built-in Data Source Writers

### Hive Integration
- [ ] **CTAS (CREATE TABLE AS SELECT)** - uses built-in writers instead of Hive SerDe
  - **Legacy config**: `spark.sql.hive.convertMetastoreCtas=false`
- [ ] **Partitioned table inserts** - uses built-in writers for ORC/Parquet
  - **Legacy config**: `spark.sql.hive.convertInsertingPartitionedTable=false`

## 🎯 Testing Strategy

### High Priority Testing Areas
- [ ] **Date/time operations** - especially for dates before 1582
- [ ] **Decimal operations** - check precision and scale handling
- [ ] **JSON/CSV parsing** - verify error handling and type inference
- [ ] **Self-joins** - check for ambiguous column references
- [ ] **UDF behavior** - especially with null handling and type coercion
- [ ] **Partition operations** - verify validation and error handling

### Performance Testing
- [ ] **Cross joins** - now enabled by default
- [ ] **Built-in data source writers** - may have different performance characteristics

### Data Validation
- [ ] **Compare results** between Spark 2.4 and 3.0 for critical queries
- [ ] **Schema inference** - verify inferred schemas match expectations
- [ ] **Type casting** - check for unexpected null values or exceptions

## 📋 Configuration Summary

Quick reference for key legacy configurations:

```properties
# Dataset/DataFrame APIs
spark.sql.legacy.dataset.nameNonStructGroupingKeyAsValue=true
spark.sql.legacy.doLooseUpcast=true

# DDL and SQL
spark.sql.storeAssignmentPolicy=LEGACY
spark.sql.legacy.notReserveProperties=true
spark.sql.legacy.setCommandRejectsSparkCoreConfs=false

# Functions and UDFs
spark.sql.legacy.createEmptyCollectionUsingStringType=true
spark.sql.legacy.allowHashOnMapType=true
spark.sql.legacy.allowUntypedScalaUDF=true
spark.sql.legacy.followThreeValuedLogicInArrayExists=false

# Query Engine
spark.sql.legacy.exponentLiteralAsDecimal.enabled=true
spark.sql.legacy.allowNegativeScaleOfDecimal=true
spark.sql.analyzer.failAmbiguousSelfJoin=false
spark.sql.legacy.ctePrecedencePolicy=LEGACY

# Data Sources
spark.sql.hive.caseSensitiveInferenceMode=INFER_AND_SAVE
spark.sql.sources.validatePartitionColumns=false
spark.sql.files.ignoreMissingFiles=true
spark.sql.legacy.json.allowEmptyString.enabled=true

# Hive Integration
spark.sql.hive.convertMetastoreCtas=false
spark.sql.hive.convertInsertingPartitionedTable=false

# Session
spark.sql.legacy.sessionInitWithConfigDefaults=true
```

---

**Note**: This checklist covers the major changes. Always test thoroughly in a development environment before applying to production workloads.
