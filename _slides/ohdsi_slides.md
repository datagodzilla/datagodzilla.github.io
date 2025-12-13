---
marp: true
theme: default
paginate: true
backgroundColor: #1a1a2e
color: #eaeaea
style: |
  section {
    font-family: 'Segoe UI', sans-serif;
  }
  h1 {
    color: #e94560;
  }
  h2 {
    color: #0f3460;
    background: linear-gradient(90deg, #e94560, #0f3460);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }
  code {
    background: #16213e;
  }
  table {
    font-size: 0.8em;
  }
---

# OHDSI Introduction
## From Real-World Data to Reliable Evidence

**Observational Health Data Sciences and Informatics**

---

# What We'll Cover Today

1. **What is OHDSI?** - The global community
2. **The Data Problem** - Why standardization matters
3. **OMOP CDM** - The Common Data Model
4. **Vocabularies** - The heart of OMOP
5. **Study Types** - Characterization, Estimation, Prediction
6. **Tools** - Athena, Atlas, HADES
7. **Getting Started** - How to join

---

# The Real-World Data Challenge

Healthcare data is **messy**:

- Different EHR systems (Epic, Cerner, Meditech)
- Different coding systems (ICD-10, SNOMED, RxNorm)
- Different countries and languages
- Different database structures

**Problem**: How do we analyze data across institutions?

---

# The Electrical Outlet Analogy

Your laptop (analytical method) works the same everywhere...

But every country has **different outlets** (data formats)!

| USA 🇺🇸 | UK 🇬🇧 | Europe 🇪🇺 | Australia 🇦🇺 |
|---------|---------|------------|---------------|
| Type A/B | Type G | Type C/F | Type I |

**OMOP CDM = Universal Adapter**

---

# What is OHDSI?

**O**bservational **H**ealth **D**ata **S**ciences and **I**nformatics

| Metric | Value |
|--------|-------|
| Founded | 2014 |
| Collaborators | 4,751+ |
| Countries | 88 |
| Time Zones | 21 |
| Continents | 6 |

*Central Hub: Columbia University*

---

# OHDSI Mission

> "Improve health by empowering a community to **collaboratively generate the evidence** that promotes better health decisions and better care."

**Key Words:**
- Community
- Collaborative
- Evidence
- Better Decisions

---

# OMOP vs OHDSI: What's the Difference?

| OMOP | OHDSI |
|------|-------|
| 5-year US project | Global ongoing community |
| Created the CDM | Maintains and evolves CDM |
| Narrow focus (drug safety) | Broad scope (all research) |
| Project ended | Living ecosystem |

**Analogy**: OMOP created the iPhone; OHDSI develops iOS

---

# OMOP CDM Principles

1. **Patient-Centric** - People, not transactions
2. **Standard Vocabulary** - Common terminology
3. **Domain-Oriented** - Organized by clinical area
4. **Source Preservation** - Keep original codes
5. **Extensible** - Can evolve and grow
6. **Database Independent** - Works anywhere

---

# OMOP CDM Table Groups

```
┌─────────────────────────────────────┐
│ 1. STANDARDIZED VOCABULARIES        │
│    (Pre-filled by OHDSI)            │
│    • 12M concepts • 142 vocabularies│
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ 2. STANDARD CLINICAL TABLES         │
│    (You fill via ETL)               │
│    • Person • Visit • Condition     │
│    • Drug • Procedure • Measurement │
└─────────────────────────────────────┘
```

---

# The PERSON Table is Central

```
           ┌──────────┐
           │  PERSON  │
           └────┬─────┘
    ┌──────────┬┴──────────┐
    ↓          ↓           ↓
┌───────┐ ┌─────────┐ ┌────────┐
│ Visit │ │Condition│ │  Drug  │
└───────┘ └─────────┘ └────────┘
    ↓          ↓           ↓
┌───────┐ ┌─────────┐ ┌────────┐
│ Meas- │ │Observ-  │ │Proced- │
│ ure   │ │ation    │ │ure     │
└───────┘ └─────────┘ └────────┘
```

---

# Standardized Vocabularies

The **heart** of OMOP:

| Metric | Count |
|--------|-------|
| Concepts | 12+ million |
| Vocabularies | 142 |
| Domains | 44 |
| Relationships | 80+ million |

All provided **FREE** by OHDSI!

---

# Major Vocabulary Sources

| Vocabulary | Domain | Example |
|------------|--------|---------|
| SNOMED-CT | Conditions | Type 2 Diabetes |
| RxNorm | Drugs | Ciprofloxacin |
| LOINC | Labs | Hemoglobin A1c |
| ICD-10 | Diagnoses | E11.9 |
| CPT | Procedures | 99213 |

---

# Concept ID vs Concept Code

| Attribute | Concept ID | Concept Code |
|-----------|------------|--------------|
| Source | OHDSI-assigned | External |
| Unique | Globally | Within vocab |
| Format | Integer | String |
| Example | 1797513 | "2551" |

**Concept ID ensures uniqueness across OMOP**

---

# Vocabulary Mapping Example

**Hospital A** (RxNorm): Cipro code = `2551`
**Hospital B** (HMOG): Cipro code = `104`

Both map to:
**OMOP Standard Concept ID: `1797513`**

✅ Now they can share and compare data!

---

# Athena: Vocabulary Portal

**athena.ohdsi.org**

- Browse 12M+ concepts
- Search for codes
- Download vocabulary CSVs
- Select which vocabs to include

*Your first stop for OMOP vocabulary work*

---

# Three Study Types

| Type | Question | Example |
|------|----------|---------|
| **Characterization** | "Who are these patients?" | Demographics of diabetics |
| **Estimation** | "Does A cause B?" | Drug safety studies |
| **Prediction** | "What will happen?" | Risk models |

---

# Estimation Study Template

> "Does exposure to **[TREATMENT]** have different risk of **[OUTCOME]** within **[TIME]** relative to **[COMPARATOR]**?"

**Example:**
Does **fluoroquinolone** exposure have different risk of **aortic aneurysm** within **60 days** vs **other UTI treatments**?

---

# Federated Learning

**Traditional**: Send all data to central location 😟

**Federated**: Run analysis locally, share only summaries 😊

```
┌─────┐ ┌─────┐ ┌─────┐
│ DB1 │ │ DB2 │ │ DB3 │
│ 📊  │ │ 📊  │ │ 📊  │  ← Analysis runs here
└──┬──┘ └──┬──┘ └──┬──┘
   └───────┼───────┘
           ↓
   Summary Statistics Only
```

---

# OHDSI Tool Ecosystem

| Tool | Purpose |
|------|---------|
| **Athena** | Vocabulary browser & download |
| **Atlas** | Web-based study design |
| **HADES** | R packages for analysis |
| **Strategus** | Pipeline orchestration |
| **DQD** | Data quality validation |

---

# Atlas: Visual Study Design

**atlas.ohdsi.org**

- Build cohort definitions (no coding!)
- Create concept sets
- Design observational studies
- Export to HADES

*The visual interface for OHDSI*

---

# HADES: R Package Suite

**H**ealth **A**nalytics **D**ata-to-**E**vidence **S**uite

- CohortGenerator
- CohortDiagnostics
- FeatureExtraction
- CohortMethod
- PatientLevelPrediction

*The analytical engine of OHDSI*

---

# Study Building Blocks

1. **Databases** - OMOP-converted sites
2. **Phenotypes** - Cohort definitions
3. **Study Design** - Methodology
4. **Methods** - Statistical approaches
5. **Tools** - Atlas, HADES, Strategus

*Assemble blocks → Generate evidence*

---

# Working Groups

Join to **listen and learn** (no expertise required!):

- CDM Working Group
- Vocabularies Working Group
- HADES Working Group
- Phenotype Working Group
- Clinical Trials Working Group

*"You can ask naive questions!"*

---

# Getting Started

1. **Learn**: Read The Book of OHDSI
2. **Explore**: Use Athena & Atlas
3. **Join**: Working groups (just listen!)
4. **Attend**: Annual Symposium
5. **Contribute**: When you're ready

---

# Key Resources

| Resource | URL |
|----------|-----|
| Main Website | ohdsi.org |
| Vocabularies | athena.ohdsi.org |
| Study Design | atlas.ohdsi.org |
| Book | ohdsi.github.io/TheBookOfOhdsi |
| Forums | forums.ohdsi.org |
| GitHub | github.com/OHDSI |

---

# Key Takeaways

1. ✅ OHDSI is a **global community** (4,750+ members)
2. ✅ OMOP CDM **standardizes** real-world data
3. ✅ **12M+ concepts** across 142 vocabularies
4. ✅ Three study types: **Characterization, Estimation, Prediction**
5. ✅ Tools are **free and open-source**
6. ✅ Working groups **welcome newcomers**

---

# The "Secret Agenda"

The faculty wants you to:

1. **Get exposed** to many topics
2. **Get excited** about something
3. **Take action** to join the community

> "Whatever you're excited about, take the action to potentially join!"

---

# Questions?

**Learn More:**
- ohdsi.org
- forums.ohdsi.org
- youtube.com/@OHDSI

**Get Started:**
- athena.ohdsi.org
- atlas.ohdsi.org

---

# Thank You!

**OHDSI**: Improving health through collaborative evidence generation

*One community, one mission, real-world evidence*

🌐 ohdsi.org
📚 ohdsi.github.io/TheBookOfOhdsi
💬 forums.ohdsi.org
