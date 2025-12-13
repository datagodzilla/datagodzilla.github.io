# OHDSI Introduction Quiz

## Assessment for Understanding OHDSI Fundamentals

**Instructions**: Answer all questions. Check your answers against the answer key at the end.

---

## Section 1: Multiple Choice (15 Questions)

### Question 1
What does OHDSI stand for?

A) Observational Healthcare Data Science Initiative
B) Observational Health Data Sciences and Informatics
C) Open Health Data Sharing Infrastructure
D) Organized Healthcare Data Systems Integration

---

### Question 2
When was OHDSI founded?

A) 2010
B) 2012
C) 2014
D) 2016

---

### Question 3
What was OMOP (the predecessor to OHDSI)?

A) An open-source software company
B) A public-private partnership focused on medical product effects
C) A government health database
D) A European healthcare initiative

---

### Question 4
How many collaborators are currently in the OHDSI community?

A) About 1,000
B) About 2,500
C) About 4,750
D) About 10,000

---

### Question 5
What is the primary purpose of the OMOP Common Data Model?

A) To process insurance claims
B) To standardize healthcare data for reproducible research
C) To manage hospital billing systems
D) To store patient images

---

### Question 6
Which tool would you use to download OMOP vocabularies?

A) Atlas
B) HADES
C) Athena
D) Strategus

---

### Question 7
What is the difference between a Concept ID and a Concept Code?

A) There is no difference
B) Concept ID is OHDSI-assigned and globally unique; Concept Code is from external sources
C) Concept Code is always numeric; Concept ID is always text
D) Concept ID is for drugs only; Concept Code is for conditions

---

### Question 8
How many concepts are in the OMOP standardized vocabularies?

A) About 1 million
B) About 5 million
C) About 12 million
D) About 50 million

---

### Question 9
Which of these is NOT one of the three main study types in OHDSI?

A) Characterization
B) Estimation
C) Prediction
D) Randomization

---

### Question 10
What does the PERSON table represent in OMOP CDM?

A) Healthcare providers
B) The central table around which all patient data revolves
C) Administrative staff
D) Insurance companies

---

### Question 11
Which HADES package is used for comparative effectiveness studies?

A) CohortGenerator
B) CohortMethod
C) PatientLevelPrediction
D) FeatureExtraction

---

### Question 12
What is federated learning in the OHDSI context?

A) A method to share all patient data centrally
B) A way to run analyses locally and only share summary statistics
C) A type of machine learning algorithm
D) A voting system for study design

---

### Question 13
In the fluoroquinolone study example, what was the outcome being studied?

A) UTI recurrence
B) Liver toxicity
C) Aortic aneurysm or dissection
D) Kidney failure

---

### Question 14
What principle ensures that original source codes are preserved in OMOP?

A) Patient-centric design
B) Data provenance preservation
C) Database independence
D) Standard vocabulary

---

### Question 15
Which vocabulary is primarily used for medications in OMOP?

A) SNOMED-CT
B) ICD-10
C) RxNorm
D) LOINC

---

## Section 2: True/False (10 Questions)

### Question 16
True or False: OMOP CDM is designed primarily for hospital billing purposes.

---

### Question 17
True or False: You must be an expert to join an OHDSI working group.

---

### Question 18
True or False: OMOP CDM only works with PostgreSQL databases.

---

### Question 19
True or False: Athena provides vocabulary data for free.

---

### Question 20
True or False: A phenotype and a cohort are essentially the same thing.

---

### Question 21
True or False: The OHDSI community spans all 6 inhabited continents.

---

### Question 22
True or False: Atlas requires programming skills to build cohort definitions.

---

### Question 23
True or False: Fluoroquinolones are antibiotics commonly used to treat UTIs.

---

### Question 24
True or False: In federated learning, patient-level data must be shared between sites.

---

### Question 25
True or False: The Data Quality Dashboard should be run after ETL to validate data quality.

---

## Section 3: Short Answer (5 Questions)

### Question 26
Explain the "electrical outlet" analogy for understanding OMOP's purpose. (2-3 sentences)

---

### Question 27
What are the five building blocks needed for an OHDSI study?

---

### Question 28
Write an estimation study question using the template: "Does exposure to [TREATMENT] have a different risk of [OUTCOME] within [TIME] relative to [COMPARATOR]?"

---

### Question 29
Name three major vocabulary sources used in OMOP and what clinical domain each covers.

---

### Question 30
Describe two ways a newcomer can participate in the OHDSI community.

---

## Answer Key

### Section 1: Multiple Choice

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | B | Observational Health Data Sciences and Informatics |
| 2 | C | OHDSI was founded in 2014 |
| 3 | B | OMOP was a US public-private partnership that created the CDM |
| 4 | C | Currently 4,751+ collaborators |
| 5 | B | OMOP CDM standardizes data for reproducible observational research |
| 6 | C | Athena (athena.ohdsi.org) is the vocabulary download tool |
| 7 | B | Concept ID is OHDSI-assigned unique identifier; Code is from source vocabularies |
| 8 | C | Over 12 million concepts across 142 vocabularies |
| 9 | D | The three types are Characterization, Estimation, and Prediction |
| 10 | B | PERSON is the central patient-centric table |
| 11 | B | CohortMethod is used for comparative effectiveness/safety studies |
| 12 | B | Analysis runs locally; only summary statistics are shared |
| 13 | C | The study examined risk of aortic aneurysm or dissection |
| 14 | B | Data provenance preservation keeps original source codes |
| 15 | C | RxNorm is the primary drug vocabulary |

### Section 2: True/False

| Q | Answer | Explanation |
|---|--------|-------------|
| 16 | False | OMOP CDM is for observational research, NOT billing |
| 17 | False | Working groups welcome newcomers to listen and learn |
| 18 | False | OMOP CDM is database-independent (PostgreSQL, SQL Server, Oracle, etc.) |
| 19 | True | Athena provides vocabulary downloads for free |
| 20 | True | Terms are often used interchangeably (phenotype = algorithm, cohort = patients found) |
| 21 | True | OHDSI spans 88 countries across 6 continents |
| 22 | False | Atlas provides a visual interface requiring no programming |
| 23 | True | Fluoroquinolones (Ciprofloxacin, Levofloxacin) commonly treat UTIs |
| 24 | False | In federated learning, only summary statistics are shared |
| 25 | True | DQD validates data quality after ETL process |

### Section 3: Short Answer (Sample Answers)

**Question 26**:
Just as electronic appliances work the same everywhere but need different electrical outlet adapters in different countries, analytical methods work the same way but need different interfaces for different healthcare data systems. OMOP CDM acts as a universal adapter, providing the same interface for all data sources so tools work everywhere.

**Question 27**:
1. Databases (OMOP-converted data sources)
2. Phenotypes (cohort definitions)
3. Study Design (estimation, characterization, prediction)
4. Methods (statistical approaches)
5. Tools (Atlas, HADES, Strategus)

**Question 28**:
Example: "Does exposure to ACE inhibitors have a different risk of experiencing stroke within 1 year relative to ARBs in patients with hypertension?"

**Question 29**:
- **SNOMED-CT**: Conditions/diagnoses
- **RxNorm**: Medications/drugs
- **LOINC**: Laboratory tests/measurements
(Also acceptable: ICD-10 for diagnoses, CPT for procedures)

**Question 30**:
1. Join a working group as an observer to listen and learn (CDM, Vocabularies, HADES, etc.)
2. Use OHDSI tools like Athena to browse vocabularies or Atlas to explore concepts
(Also acceptable: Attend the annual symposium, read The Book of OHDSI, participate in forums, convert your data to OMOP)

---

## Scoring Guide

| Score | Level | Recommendation |
|-------|-------|----------------|
| 27-30 | Excellent | Ready for advanced OHDSI topics |
| 22-26 | Good | Review missed concepts, then proceed |
| 17-21 | Fair | Review the comprehensive report |
| <17 | Needs Work | Re-watch the tutorial video |

---

*Total Questions: 30 | Estimated Time: 20-30 minutes*
