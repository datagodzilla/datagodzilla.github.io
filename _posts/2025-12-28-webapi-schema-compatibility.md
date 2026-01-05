---
title: "OHDSI WebAPI Schema Compatibility: A Deep Dive into External Database Integration"
date: 2025-12-28
categories: [ohdsi, atlas, database, healthcare-data]
tags: [ohdsi, webapi, postgresql, omop-cdm, broadsea, schema-compatibility, database-migration, cohort-generation, characterization]
description: "Discover the hidden schema requirements when connecting OHDSI ATLAS to your own PostgreSQL database. Learn from our hands-on troubleshooting of WebAPI 2.x compatibility issues, including the exact SQL fixes for cohort generation and characterization."
author: DataGodzilla
reading_time: 15 min
keywords: [OHDSI WebAPI, schema compatibility, PostgreSQL, OMOP CDM, Broadsea, database integration, cohort generation, characterization]
layout: post
---

# OHDSI WebAPI Schema Compatibility: A Deep Dive into External Database Integration

## The Hidden Requirements Nobody Tells You About

You've built your OMOP CDM database. You've loaded Synthea data. You've set up the standardized vocabularies. Everything looks perfect. Then you connect ATLAS to your external PostgreSQL database and... **nothing works quite right**.

Cohort generation shows "..." instead of patient counts. Characterization fails with cryptic schema errors. The UI displays "COMPLETE" but the data seems invisible.

This is the story of what we discovered connecting ATLAS to our own database—and the exact fixes that made everything work.

---

## The Setup: What We Had Built

After extensive preparation, our PostgreSQL database contained a complete OHDSI environment:

```
Database: ohdsi_learning
├── cdm (OMOP CDM 5.4)
│   ├── person (2,411 synthetic patients)
│   ├── condition_occurrence
│   ├── drug_exposure
│   ├── procedure_occurrence
│   └── measurement
├── vocabulary (~6 million concepts)
│   ├── concept
│   ├── concept_relationship
│   └── concept_ancestor
├── results (Achilles outputs)
└── webapi (Source configuration)
```

We deployed OHDSI Broadsea via Docker Compose, configured our external database as a data source, and launched ATLAS.

**Initial Success**: The Data Sources dashboard showed our 2,411 patients with demographic breakdowns. Achilles results displayed correctly. Vocabulary search worked perfectly.

**Then we tried to use the cohorts...**

---

## Phase 1: The Cohort Generation Mystery

### The Symptom

After creating a Type 2 Diabetes cohort definition and clicking "Generate", the status showed **COMPLETE**—but the People and Records columns displayed "..." instead of actual counts.

![Cohort showing counts after fixes](/assets/images/posts/schema-fix-cohort-generation-success.png)
*After fixes: cohorts now show actual patient counts (219 for T2DM)*

### Direct Database Check

Our first troubleshooting step was to query the database directly:

```sql
SELECT cohort_definition_id, COUNT(DISTINCT subject_id) AS persons
FROM results.cohort
GROUP BY cohort_definition_id;

-- Result: 219 patients in T2DM cohort!
```

**The data was correct**—the problem was in the UI/API layer.

### The First Errors

Checking WebAPI logs revealed the real issues:

```
ERROR: schema "temp" does not exist
ERROR: column "design_hash" of relation "cohort_inclusion" does not exist
ERROR: relation "results.cohort_cache" does not exist
ERROR: column "mode_id" of relation "cohort_inclusion_result" does not exist
```

### Root Cause: WebAPI ≠ OMOP CDM

**WebAPI 2.x has evolved significantly beyond the base OMOP CDM specification.**

The OMOP Common Data Model defines the clinical data tables. But WebAPI requires additional infrastructure:

| Component | OMOP CDM | WebAPI 2.x | Purpose |
|-----------|----------|------------|---------|
| `temp` schema | No | **Required** | Working space for analytics |
| `design_hash` column | No | **Required** | Cache invalidation |
| `mode_id` column | No | **Required** | Generation mode tracking |
| `*_cache` tables | No | **Required** | Performance optimization |

When WebAPI starts with its **built-in database**, Flyway migrations create these automatically. With an **external database**, they don't exist.

---

## Phase 1 Fix: Core Schema Patches

Here's what we applied to enable cohort generation:

```sql
-- 1. Create temp schema
CREATE SCHEMA IF NOT EXISTS temp;
GRANT ALL ON SCHEMA temp TO PUBLIC;

-- 2. Add design_hash column
ALTER TABLE results.cohort_inclusion
  ADD COLUMN IF NOT EXISTS design_hash INT;

-- 3. Add mode_id columns
ALTER TABLE results.cohort_inclusion_result
  ADD COLUMN IF NOT EXISTS mode_id INT DEFAULT 0;
ALTER TABLE results.cohort_inclusion_stats
  ADD COLUMN IF NOT EXISTS mode_id INT DEFAULT 0;
ALTER TABLE results.cohort_summary_stats
  ADD COLUMN IF NOT EXISTS mode_id INT DEFAULT 0;
ALTER TABLE results.cohort_censor_stats
  ADD COLUMN IF NOT EXISTS mode_id INT DEFAULT 0;

-- 4. Create cache tables
CREATE TABLE IF NOT EXISTS results.cohort_cache (
    design_hash INT NOT NULL,
    subject_id BIGINT NOT NULL,
    cohort_start_date DATE NOT NULL,
    cohort_end_date DATE NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_cohort_cache_hash
  ON results.cohort_cache(design_hash);

CREATE TABLE IF NOT EXISTS results.cohort_censor_stats_cache (
    design_hash INT NOT NULL,
    lost_count BIGINT NOT NULL
);

CREATE TABLE IF NOT EXISTS results.cohort_inclusion_result_cache (
    design_hash INT NOT NULL,
    mode_id INT NOT NULL,
    inclusion_rule_mask BIGINT NOT NULL,
    person_count BIGINT NOT NULL
);

CREATE TABLE IF NOT EXISTS results.cohort_inclusion_stats_cache (
    design_hash INT NOT NULL,
    rule_sequence INT NOT NULL,
    mode_id INT NOT NULL,
    person_count BIGINT NOT NULL,
    gain_count BIGINT NOT NULL,
    person_total BIGINT NOT NULL
);

CREATE TABLE IF NOT EXISTS results.cohort_summary_stats_cache (
    design_hash INT NOT NULL,
    mode_id INT NOT NULL,
    base_count BIGINT NOT NULL,
    final_count BIGINT NOT NULL
);
```

**After restart**: Cohort generation worked! All 5 cohorts showed correct patient counts:

| Cohort | Patients |
|--------|----------|
| Type 2 Diabetes | 219 |
| Multiple Chronic Conditions | 486 |
| Metformin New Users | 0 |
| New Statin Users | 0 |
| Elevated HbA1c | 69 |

Inclusion reports now displayed match rates and population visualizations correctly.

---

## Phase 2: The Characterization Challenge

With cohort generation working, we tried **Cohort Characterization**—comparing demographics, conditions, and medications across cohorts.

### New Errors Appeared

```
ERROR: column "type" of relation "cc_results" does not exist
ERROR: column "concept_id" of relation "cc_results" does not exist
```

Characterization uses different result tables than cohort generation.

### Discovery: The WebAPI DDL Endpoint

WebAPI provides an endpoint that returns the expected schema:

```bash
curl http://localhost:8080/WebAPI/ddl/results
```

This revealed the complete `cc_results` table structure including columns our database was missing:

- `type` - Classification of the result
- `concept_id` - Linked vocabulary concept
- `aggregate_id` - Aggregation identifier
- `aggregate_name` - Aggregation label
- `missing_means_zero` - Zero handling flag

---

## Phase 2 Fix: Characterization Schema

```sql
-- 1. Add missing cc_results columns
ALTER TABLE results.cc_results
  ADD COLUMN IF NOT EXISTS type VARCHAR(255);
ALTER TABLE results.cc_results
  ADD COLUMN IF NOT EXISTS concept_id INTEGER DEFAULT 0;
ALTER TABLE results.cc_results
  ADD COLUMN IF NOT EXISTS aggregate_id INTEGER;
ALTER TABLE results.cc_results
  ADD COLUMN IF NOT EXISTS aggregate_name VARCHAR(1000);
ALTER TABLE results.cc_results
  ADD COLUMN IF NOT EXISTS missing_means_zero INTEGER;

-- 2. Create temporal results tables
CREATE TABLE IF NOT EXISTS results.cc_temporal_results (
  type VARCHAR(255),
  fa_type VARCHAR(255),
  cc_generation_id BIGINT,
  analysis_id INTEGER,
  analysis_name VARCHAR(1000),
  covariate_id BIGINT,
  covariate_name VARCHAR(1000),
  strata_id BIGINT,
  strata_name VARCHAR(1000),
  concept_id INTEGER,
  count_value BIGINT,
  avg_value DOUBLE PRECISION,
  cohort_definition_id BIGINT,
  time_id INTEGER,
  start_day INTEGER,
  end_day INTEGER
);

CREATE TABLE IF NOT EXISTS results.cc_temporal_annual_results (
  type VARCHAR(255),
  fa_type VARCHAR(255),
  cc_generation_id BIGINT,
  analysis_id INTEGER,
  analysis_name VARCHAR(1000),
  covariate_id BIGINT,
  covariate_name VARCHAR(1000),
  strata_id BIGINT,
  strata_name VARCHAR(1000),
  concept_id INTEGER,
  count_value BIGINT,
  avg_value DOUBLE PRECISION,
  cohort_definition_id BIGINT,
  event_year INTEGER
);

-- 3. Grant permissions
GRANT ALL ON ALL TABLES IN SCHEMA results TO PUBLIC;
GRANT ALL ON ALL TABLES IN SCHEMA temp TO PUBLIC;
```

**After restart**: Characterization completed successfully!

- 10 reports generated
- 866 total records
- Demographics analysis with age group distributions

---

## Complete Test Results

### Functionality Verification

| Feature | Before Fix | After Fix | Status |
|---------|------------|-----------|--------|
| Cohort Generation | Shows "..." | Shows counts | ✅ |
| Inclusion Reports | Fails | Works with statistics | ✅ |
| Population Visualization | Fails | Displays correctly | ✅ |
| Characterization | ERROR | 866 records displayed | ✅ |
| Demographics Analysis | Fails | Age groups with counts | ✅ |

### Jobs Execution Summary

| Job | Name | Status | Duration |
|-----|------|--------|----------|
| 7 | Type 2 Diabetes cohort | COMPLETED | 1 min |
| 8 | Multiple Chronic Conditions | COMPLETED | < 1 min |
| 9-11 | Other cohorts | COMPLETED | < 1 min each |
| 14 | Characterization | COMPLETED | 1 min |

---

## Architecture Understanding

### What Broadsea Looks Like

```
┌─────────────────────────────────────────────────────────────┐
│                    OHDSI Broadsea Stack                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Browser → Traefik → ATLAS (nginx) → WebAPI (Tomcat)      │
│                           ↓                                 │
│              PostgreSQL (ohdsi_learning)                    │
│     ┌─────────┬───────────┬─────────┬──────────┐          │
│     │   cdm   │ vocabulary│ results │  webapi  │          │
│     │ (data)  │ (concepts)│ (output)│ (config) │          │
│     └─────────┴───────────┴─────────┴──────────┘          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Where Schema Gaps Occur

```
┌──────────────────────────────────────────────────────┐
│              OMOP CDM (What You Built)               │
│  ✓ person, condition_occurrence, drug_exposure...   │
│  ✓ vocabulary.concept, concept_relationship...      │
│  ✓ results.cohort, cohort_definition                │
└──────────────────────────────────────────────────────┘
                          ↓
┌──────────────────────────────────────────────────────┐
│              WebAPI 2.x Requirements (Gap)           │
│  ✗ temp schema                                       │
│  ✗ design_hash, mode_id columns                     │
│  ✗ cohort_cache, cohort_*_cache tables              │
│  ✗ cc_results additional columns                    │
│  ✗ cc_temporal_results, cc_temporal_annual_results  │
└──────────────────────────────────────────────────────┘
```

---

## Three Paths Forward

### Option 1: Manual Patching (What We Did)

Apply SQL fixes reactively as errors appear.

**Pros:**
- Learn exactly what WebAPI needs
- Good for understanding the system
- Works for core features

**Cons:**
- May miss edge cases
- Tedious
- May break with WebAPI updates

### Option 2: Run Flyway Migrations

Apply official DDL scripts from WebAPI repository:

```bash
git clone https://github.com/OHDSI/WebAPI.git
cd WebAPI/src/main/resources/db/migration/postgresql
# Apply migrations manually or configure Flyway
```

**Pros:**
- Complete and version-matched
- Official approach

**Cons:**
- Complex setup
- Requires Flyway knowledge

### Option 3: Use Broadsea's Built-in Database (Recommended)

Migrate your data into Broadsea's pre-configured database:

```bash
# Export from external database
pg_dump -h localhost -U postgres -d ohdsi_learning \
  --schema=cdm --schema=vocabulary --data-only -f export.sql

# Import into Broadsea's database
docker exec -i broadsea-atlasdb psql -U postgres < export.sql
```

**Pros:**
- Zero schema errors
- All features work immediately
- Easiest long-term maintenance

**Cons:**
- Requires data migration
- Additional container

---

## Key Lessons Learned

### 1. WebAPI ≠ OMOP CDM

The OMOP Common Data Model specification and WebAPI's operational requirements have diverged. WebAPI needs additional infrastructure beyond the CDM.

### 2. The DDL Endpoint is Your Friend

When troubleshooting, fetch the expected schema:
```
GET /WebAPI/ddl/results
```

This tells you exactly what WebAPI expects to find.

### 3. Check the Database First

When the UI shows unexpected results, query the database directly. The data may be correct even when the display is broken.

### 4. Flyway Manages WebAPI Schema

WebAPI uses Flyway for database migrations. These run automatically on built-in databases but not on external ones.

### 5. Cache Tables Enable UI Features

The `*_cache` tables aren't just for performance—they're required for inclusion reports and statistics to display in the UI.

### 6. Restart After Schema Changes

After modifying the database schema, restart the WebAPI container for changes to take effect.

---

## Quick Reference: Complete Fix Script

For anyone facing similar issues, here's the complete script:

```sql
-- ============================================
-- WebAPI 2.x Schema Compatibility Fixes
-- For external PostgreSQL databases
-- ============================================

-- PHASE 1: Core Infrastructure
CREATE SCHEMA IF NOT EXISTS temp;
GRANT ALL ON SCHEMA temp TO PUBLIC;

-- PHASE 2: Cohort Generation Support
ALTER TABLE results.cohort_inclusion
  ADD COLUMN IF NOT EXISTS design_hash INT;

ALTER TABLE results.cohort_inclusion_result
  ADD COLUMN IF NOT EXISTS mode_id INT DEFAULT 0;
ALTER TABLE results.cohort_inclusion_stats
  ADD COLUMN IF NOT EXISTS mode_id INT DEFAULT 0;
ALTER TABLE results.cohort_summary_stats
  ADD COLUMN IF NOT EXISTS mode_id INT DEFAULT 0;
ALTER TABLE results.cohort_censor_stats
  ADD COLUMN IF NOT EXISTS mode_id INT DEFAULT 0;

-- Cache tables
CREATE TABLE IF NOT EXISTS results.cohort_cache (
    design_hash INT NOT NULL,
    subject_id BIGINT NOT NULL,
    cohort_start_date DATE NOT NULL,
    cohort_end_date DATE NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_cohort_cache_hash
  ON results.cohort_cache(design_hash);

CREATE TABLE IF NOT EXISTS results.cohort_censor_stats_cache (
    design_hash INT NOT NULL,
    lost_count BIGINT NOT NULL
);

CREATE TABLE IF NOT EXISTS results.cohort_inclusion_result_cache (
    design_hash INT NOT NULL,
    mode_id INT NOT NULL,
    inclusion_rule_mask BIGINT NOT NULL,
    person_count BIGINT NOT NULL
);

CREATE TABLE IF NOT EXISTS results.cohort_inclusion_stats_cache (
    design_hash INT NOT NULL,
    rule_sequence INT NOT NULL,
    mode_id INT NOT NULL,
    person_count BIGINT NOT NULL,
    gain_count BIGINT NOT NULL,
    person_total BIGINT NOT NULL
);

CREATE TABLE IF NOT EXISTS results.cohort_summary_stats_cache (
    design_hash INT NOT NULL,
    mode_id INT NOT NULL,
    base_count BIGINT NOT NULL,
    final_count BIGINT NOT NULL
);

-- PHASE 3: Characterization Support
ALTER TABLE results.cc_results
  ADD COLUMN IF NOT EXISTS type VARCHAR(255);
ALTER TABLE results.cc_results
  ADD COLUMN IF NOT EXISTS concept_id INTEGER DEFAULT 0;
ALTER TABLE results.cc_results
  ADD COLUMN IF NOT EXISTS aggregate_id INTEGER;
ALTER TABLE results.cc_results
  ADD COLUMN IF NOT EXISTS aggregate_name VARCHAR(1000);
ALTER TABLE results.cc_results
  ADD COLUMN IF NOT EXISTS missing_means_zero INTEGER;

CREATE TABLE IF NOT EXISTS results.cc_temporal_results (
  type VARCHAR(255),
  fa_type VARCHAR(255),
  cc_generation_id BIGINT,
  analysis_id INTEGER,
  analysis_name VARCHAR(1000),
  covariate_id BIGINT,
  covariate_name VARCHAR(1000),
  strata_id BIGINT,
  strata_name VARCHAR(1000),
  concept_id INTEGER,
  count_value BIGINT,
  avg_value DOUBLE PRECISION,
  cohort_definition_id BIGINT,
  time_id INTEGER,
  start_day INTEGER,
  end_day INTEGER
);

CREATE TABLE IF NOT EXISTS results.cc_temporal_annual_results (
  type VARCHAR(255),
  fa_type VARCHAR(255),
  cc_generation_id BIGINT,
  analysis_id INTEGER,
  analysis_name VARCHAR(1000),
  covariate_id BIGINT,
  covariate_name VARCHAR(1000),
  strata_id BIGINT,
  strata_name VARCHAR(1000),
  concept_id INTEGER,
  count_value BIGINT,
  avg_value DOUBLE PRECISION,
  cohort_definition_id BIGINT,
  event_year INTEGER
);

-- PHASE 4: Permissions
GRANT ALL ON ALL TABLES IN SCHEMA results TO PUBLIC;
GRANT ALL ON ALL TABLES IN SCHEMA temp TO PUBLIC;
```

---

## End-to-End Case Study: Dr. Sarah Chen's Research Workflow

With all schema fixes in place, let's walk through a complete clinical research workflow—the kind of study that ATLAS enables across the global OHDSI network.

### The Research Question

**Dr. Sarah Chen**, an endocrinologist at an academic medical center, wants to characterize patients with Type 2 Diabetes in her institution's data to understand:

1. What is the demographic profile of T2DM patients?
2. How do different age groups compare?
3. What comorbidities are most common?

### Step 1: Define the Cohort

Dr. Chen creates a cohort definition in ATLAS:

**Entry Event**: First diagnosis of Type 2 Diabetes Mellitus (SNOMED 201826)
- Requires 365 days prior observation
- Limited to first occurrence

**Inclusion Criteria**: Age ≥ 18 at index date

**Exit**: End of continuous observation

### Step 2: Generate the Cohort

After clicking "Generate", ATLAS executes the cohort SQL against our Synthea database:

![Cohort Generation Results](/assets/images/posts/cohort-generation-t2dm-219.png)
*Type 2 Diabetes cohort: 219 patients identified, generation completed in 1 second*

**Results**:
- **219 patients** meet all criteria
- **99.55% match rate** (219 of 220 initial events passed inclusion rules)
- Generation duration: 00:00:01

### Step 3: Review Inclusion Report

The inclusion report shows how patients flow through each criterion:

![Inclusion Report](/assets/images/posts/inclusion-report-t2dm.png)
*Inclusion report showing 99.55% match rate with population visualization*

| Criterion | N | % Satisfied |
|-----------|---|-------------|
| Initial Events (T2DM diagnosis) | 220 | 100% |
| Age ≥ 18 at index | 219 | 99.55% |
| **Final Cohort** | **219** | **99.55%** |

The population visualization (green area) shows the overlap between criteria—a useful tool for understanding attrition.

### Step 4: Run Characterization

Dr. Chen creates a characterization analysis to understand patient demographics. She selects:
- **Cohorts**: Type 2 Diabetes Cohort
- **Features**: Demographics (Age Group, Gender, Race, Ethnicity)
- **Domain**: Demographics

After clicking "Generate", the characterization job completes successfully:

![Characterization Results](/assets/images/posts/characterization-results-verification.png)
*Characterization results: 10 reports with 866 records showing age group distribution*

**Demographics Analysis Results**:

| Age Group | Count | Percentage |
|-----------|-------|------------|
| 40-44 | 43 | 19.63% |
| 55-59 | 31 | 14.16% |
| 30-34 | 23 | 10.50% |
| 50-54 | 23 | 10.50% |
| 45-49 | 22 | 10.05% |
| 60-64 | 22 | 10.05% |

### Step 5: Verify Job Completion

The Jobs page shows the complete execution history:

![Jobs History](/assets/images/posts/jobs-complete-history.png)
*Job 14 (Characterization) COMPLETED - all cohort generation and characterization jobs successful*

| Job | Type | Status | Duration |
|-----|------|--------|----------|
| 7-11 | Cohort Generation | COMPLETED | < 1 min each |
| 14 | Characterization | COMPLETED | 1 min |

### What Dr. Chen Learned

From this analysis, Dr. Chen discovers:
1. Her T2DM population skews toward middle age (40-59)
2. Nearly all patients (99.55%) meet the adult age criterion
3. The cohort is reproducible—running the same definition anywhere in the OHDSI network would apply identical logic

### The Power of Reproducibility

The JSON cohort definition Dr. Chen created can be:
- **Exported** and shared with collaborators worldwide
- **Imported** into any ATLAS instance connected to an OMOP CDM database
- **Executed** against 450+ databases in the OHDSI network
- **Versioned** for regulatory submissions and publications

This is the promise of OHDSI: **write once, run anywhere**.

---

## Verified Test Results Summary

Our Playwright-based validation confirmed all features working after schema fixes:

| Feature | Status | Evidence |
|---------|--------|----------|
| Cohort Generation | ✅ Working | 219 patients for T2DM |
| Inclusion Reports | ✅ Working | 99.55% match rate displayed |
| Population Visualization | ✅ Working | Green area renders correctly |
| Characterization | ✅ Working | 866 records, 10 reports |
| Demographics Analysis | ✅ Working | Age groups with counts |
| Jobs Tracking | ✅ Working | All jobs show COMPLETED |

---

## What's Next

Having resolved the schema compatibility issues and validated the complete workflow, our next step is migrating to Broadsea's built-in database for long-term stability. This will ensure full feature compatibility as WebAPI evolves.

We're also ready to explore **population-level estimation**—using our validated cohorts to ask comparative effectiveness questions like:

> *"Among T2DM patients initiating metformin, does adding an SGLT2 inhibitor reduce cardiovascular events compared to adding a sulfonylurea?"*

That's the power of reproducible cohort definitions—once you've solved "which patients?", you can ask any clinical question.

---

## Resources

### OHDSI Documentation
- [WebAPI GitHub Repository](https://github.com/OHDSI/WebAPI) - Source code and migrations
- [Broadsea Documentation](https://github.com/OHDSI/Broadsea) - Docker deployment
- [OHDSI Forums](https://forums.ohdsi.org/) - Community support
- [The Book of OHDSI](https://ohdsi.github.io/TheBookOfOhdsi/) - Comprehensive guide

### Related Posts
- [Mastering ATLAS Cohort Definitions](/ohdsi/atlas/cohort-studies/healthcare-data/2025/12/28/mastering-atlas-cohort-definitions.html) - Complete guide to building cohorts

---

*Have you encountered WebAPI schema issues? What solutions worked for your deployment? Share your experience on the [OHDSI Forums](https://forums.ohdsi.org/).*

---

**Tags**: OHDSI, WebAPI, PostgreSQL, Schema Compatibility, OMOP CDM, Broadsea, Database Integration, Cohort Generation, Characterization, Troubleshooting
