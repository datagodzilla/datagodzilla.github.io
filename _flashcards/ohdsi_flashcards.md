# OHDSI Flashcards

## Study Cards for Spaced Repetition Learning

---

### Card 1: What is OHDSI?
**Front**: What does OHDSI stand for and what is its purpose?

**Back**:
OHDSI = **Observational Health Data Sciences and Informatics**

- Founded: 2014
- Central Hub: Columbia University
- Purpose: Global community that transforms real-world healthcare data into reliable clinical evidence
- Community: 4,751+ collaborators across 88 countries

---

### Card 2: OHDSI Mission
**Front**: What is the OHDSI mission?

**Back**:
"Improve health by empowering a community to **collaboratively generate the evidence** that promotes better health decisions and better care."

Key words: Community, Collaborative, Evidence, Better Decisions

---

### Card 3: OMOP vs OHDSI
**Front**: What is the relationship between OMOP and OHDSI?

**Back**:
- **OMOP** (Observational Medical Outcomes Partnership): 5-year US public-private project that created the Common Data Model
- **OHDSI**: Born from OMOP to continue the work with broader scope
- OHDSI maintains and evolves the "OMOP CDM"

Analogy: OMOP was the startup that created the product; OHDSI is the company that continues to develop it.

---

### Card 4: OMOP CDM Purpose
**Front**: Why was the OMOP Common Data Model created?

**Back**:
To solve the **data standardization problem**:
- Different hospitals use different EHR systems (Epic, Cerner)
- Different coding systems (RxNorm, SNOMED, ICD-10)
- Different database structures

OMOP CDM provides a **universal interface** so analytical methods work across all data sources.

Analogy: Like a universal electrical adapter that works in any country.

---

### Card 5: Patient-Centric Principle
**Front**: What does "patient-centric" mean in OMOP CDM design?

**Back**:
Everything in the data model revolves around the **patient**:
- The PERSON table is the central table
- Not designed for billing or insurance
- Specifically for observational research
- "People are important, not transactions"

---

### Card 6: Concept ID vs Concept Code
**Front**: What is the difference between Concept ID and Concept Code?

**Back**:
| Attribute | Concept ID | Concept Code |
|-----------|------------|--------------|
| Source | OHDSI-assigned | External (SNOMED, RxNorm) |
| Uniqueness | Globally unique | Only within vocabulary |
| Format | Integer | String |
| Example | 1797513 | "2551" |

Concept ID ensures uniqueness across the entire OMOP ecosystem.

---

### Card 7: Standardized Vocabularies
**Front**: What are the key statistics about OMOP standardized vocabularies?

**Back**:
- **12+ million** concepts
- **142** vocabularies
- **44** domains
- **80+ million** concept relationships

All provided FREE by OHDSI through Athena.

---

### Card 8: Major Vocabulary Sources
**Front**: Name 5 major vocabulary sources used in OMOP.

**Back**:
1. **SNOMED-CT**: Conditions/diagnoses
2. **RxNorm**: Medications
3. **LOINC**: Lab tests/measurements
4. **ICD-10**: Diagnosis codes
5. **CPT**: Procedure codes

---

### Card 9: Athena
**Front**: What is Athena and what is it used for?

**Back**:
**Athena** (athena.ohdsi.org):
- Web-based vocabulary browser and download tool
- Browse 12M+ concepts
- Search for specific codes
- Download vocabulary CSV files
- Select which vocabularies to include

---

### Card 10: Atlas
**Front**: What is Atlas and what can you do with it?

**Back**:
**Atlas** (atlas.ohdsi.org):
- Web-based study design tool
- Build cohort definitions
- Create concept sets
- Design observational studies
- Visual interface (no coding required)

---

### Card 11: HADES
**Front**: What is HADES in the OHDSI ecosystem?

**Back**:
**HADES** = Health Analytics Data-to-Evidence Suite

Collection of R packages for:
- CohortGenerator - Create cohorts
- CohortDiagnostics - Validate cohorts
- FeatureExtraction - Extract patient features
- CohortMethod - Comparative effectiveness
- PatientLevelPrediction - Risk models

---

### Card 12: Three Study Types
**Front**: What are the three main study types in OHDSI?

**Back**:
1. **Characterization**: "Who are these patients?"
   - Patient profiles, disease natural history

2. **Estimation**: "Does A cause B?"
   - Comparative effectiveness, safety studies

3. **Prediction**: "What will happen to this patient?"
   - Risk models, ML/AI predictions

---

### Card 13: Estimation Study Template
**Front**: What is the template for an estimation study question?

**Back**:
"Does exposure to **[TREATMENT]** have a different risk of experiencing **[OUTCOME]** within **[TIME FRAME]** relative to **[COMPARATOR]**?"

Example: Does fluoroquinolone have different risk of aortic aneurysm within 60 days vs other UTI treatments?

---

### Card 14: Federated Learning
**Front**: What is federated learning and why is it important in OHDSI?

**Back**:
**Federated Learning**: Analysis runs at each data site; only summary statistics are shared.

Benefits:
- No patient data leaves the institution
- Preserves privacy
- Enables multi-site collaboration
- Regulatory compliance

"Bring the code to the data, not data to the code"

---

### Card 15: OMOP Clinical Tables
**Front**: Name the main clinical tables in OMOP CDM.

**Back**:
- **Person** (central table)
- **Visit_Occurrence** (encounters)
- **Condition_Occurrence** (diagnoses)
- **Drug_Exposure** (medications)
- **Procedure_Occurrence** (procedures)
- **Measurement** (lab results)
- **Observation** (other clinical facts)
- **Device_Exposure** (medical devices)

---

### Card 16: ETL Flow
**Front**: Describe the basic ETL data flow in OMOP.

**Back**:
```
Source Data (EHR, Claims)
        ↓
Standardized Vocabularies (Mapping)
        ↓
Standard Clinical Tables
```

Source codes are mapped to standard concept IDs using the vocabulary tables.

---

### Card 17: Source Preservation
**Front**: Why does OMOP preserve source data along with standard codes?

**Back**:
**Data Provenance**: Original source codes are kept so you can:
- Trace back to the source system
- Verify mapping accuracy
- Maintain granularity
- Debug ETL issues
- Ensure no information is lost

---

### Card 18: Fluoroquinolones
**Front**: What are fluoroquinolones and why were they studied?

**Back**:
**Fluoroquinolones**: Common antibiotics (Ciprofloxacin, Levofloxacin)
- Used for UTIs, respiratory infections
- Well-tolerated but with rare severe side effects

**Study Question**: Do they increase risk of aortic aneurysm/dissection?

---

### Card 19: Aortic Aneurysm vs Dissection
**Front**: What is the difference between aortic aneurysm and dissection?

**Back**:
- **Aneurysm**: Bulging of aortic wall (balloon-like)
- **Dissection**: Tear in aortic wall where blood splits the layers

Both are rare but potentially **fatal**. The fluoroquinolone study investigated if these antibiotics increase this risk.

---

### Card 20: Study Building Blocks
**Front**: What are the 5 building blocks of an OHDSI study?

**Back**:
1. **Databases**: OMOP-converted data sources
2. **Phenotypes**: Patient cohort definitions
3. **Study Design**: Methodology (estimation, prediction)
4. **Methods**: Statistical approaches
5. **Tools**: Atlas, HADES, Strategus

---

### Card 21: Working Groups
**Front**: How can beginners participate in OHDSI working groups?

**Back**:
Key message: **"You don't have to be an expert!"**

- Join to listen and learn
- Ask "naive" questions (welcomed!)
- No pressure to contribute immediately
- Options: CDM, Vocabularies, HADES, Phenotypes, Clinical Trials

---

### Card 22: Data Quality Dashboard
**Front**: What is the Data Quality Dashboard used for?

**Back**:
**DQD** (Data Quality Dashboard):
- Validates ETL quality
- Identifies data issues before analysis
- Checks completeness, conformance, plausibility
- Maintained by Katie Sidowski

Run AFTER ETL to ensure data is ready for research.

---

### Card 23: Strategus
**Front**: What is Strategus and what problem does it solve?

**Back**:
**Strategus**: Pipeline orchestration tool

- Runs complete studies end-to-end
- Ensures reproducibility
- Automates multi-step analyses
- Coordinates HADES packages

Solves: "How do I run a complex study reproducibly?"

---

### Card 24: Community Size
**Front**: What is the current size of the OHDSI community?

**Back**:
- **4,751+** collaborators
- **88** countries
- **21** time zones
- **6** continents
- **9,000** LinkedIn followers

"We are all one community"

---

### Card 25: Phenotype vs Cohort
**Front**: What is the relationship between phenotype and cohort?

**Back**:
Often used interchangeably:

- **Phenotype**: Algorithm/logic to identify patients
- **Cohort**: The actual group of patients found

Example: "Type 2 Diabetes phenotype" = algorithm that finds diabetic patients
The cohort = the 50,000 patients who match

---

### Card 26: Key OHDSI Resources
**Front**: What are the main OHDSI learning resources?

**Back**:
1. **The Book of OHDSI**: ohdsi.github.io/TheBookOfOhdsi
2. **Athena**: athena.ohdsi.org
3. **Forums**: forums.ohdsi.org
4. **YouTube**: youtube.com/@OHDSI
5. **GitHub**: github.com/OHDSI

All FREE and open-source!

---

### Card 27: Database Independence
**Front**: What does "database independent" mean for OMOP CDM?

**Back**:
OMOP CDM works with ANY relational database:
- PostgreSQL
- SQL Server
- Oracle
- SQLite
- Redshift
- BigQuery

The schema is the same; only the underlying database differs.

---

### Card 28: 142 Vocabularies
**Front**: Why are there 142 vocabularies in Athena?

**Back**:
Healthcare uses many coding systems:
- Different countries (US vs international)
- Different domains (drugs vs diagnoses)
- Different purposes (billing vs clinical)
- Legacy systems

OMOP maps them ALL to standard concepts, enabling global collaboration.

---

### Card 29: Concept Relationships
**Front**: What are concept relationships in OMOP?

**Back**:
**80+ million** relationships linking concepts:
- Is-a (hierarchical)
- Maps-to (vocabulary mapping)
- Has-ingredient (drugs)
- Is-member-of (groupings)

Enables navigating from specific to general concepts (and vice versa).

---

### Card 30: The "Secret Agenda"
**Front**: What is the "secret agenda" of OHDSI tutorials?

**Back**:
From the video: The faculty's secret mission is to:
1. **Expose** you to many topics
2. Get you **excited** about something
3. Hope you **take action to join** the community

"Whatever you're excited about, take the action to join!"

---

## Study Tips

1. Review these cards using spaced repetition (Anki, etc.)
2. Start with basic definitions, then move to relationships
3. Practice explaining concepts in your own words (Feynman technique)
4. Visit the resources to see the tools in action
5. Join a working group to reinforce learning

---

*Total Cards: 30 | Difficulty: Mixed (Beginner to Intermediate)*
