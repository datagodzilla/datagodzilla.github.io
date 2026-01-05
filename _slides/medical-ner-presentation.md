---
marp: true
theme: default
paginate: true
backgroundColor: #fff
header: 'Medical NER Pipeline'
footer: 'Building Production-Ready Clinical Text Processing | 2025'
---

# Building a Production-Ready Medical NER Pipeline

## Hybrid BioBERT + Template Approach for Clinical Text Processing

**ML Engineering Team** | January 2025

---

## The Challenge of Clinical Text Processing

```
"Patient denies chest pain but reports shortness of breath.
No history of diabetes. Father had hypertension."
```

**Key Challenges**:
- ❌ Negation: "denies chest pain" ≠ has chest pain
- ❌ Scope: "denies X but reports Y" - complex scoping
- ❌ Family: "father had" ≠ patient has
- ❌ Historical: "no history of" ≠ current condition

---

## Why Standard NLP Tools Fail

| Generic NER | Our Requirements |
|-------------|------------------|
| ❌ Miss medical entities | ✅ 57K+ medical terms |
| ❌ No clinical context | ✅ 5 context types |
| ❌ Poor negation handling | ✅ 99 negation patterns |
| ❌ 60-70% accuracy | ✅ 96% accuracy needed |

**Example Failure**:
- Input: "No evidence of pneumonia"
- Generic: ✅ "pneumonia" (WRONG!)
- Medical NER: ❌ "pneumonia" [NEGATED]

---

## Project Goals and Outcomes

### Success Metrics Achieved

| Metric | Target | Achieved |
|--------|--------|----------|
| Entity Detection | >95% | **96%** |
| Context Classification | >90% | **93%** |
| Negation Precision | >95% | **99.2%** |

---

## Key Performance Metrics

### Entity Detection Comparison

| Method | Precision | Recall | F1 |
|--------|-----------|--------|-----|
| BioBERT Only | 87% | 72% | 79% |
| **Hybrid System** | **96%** | **96%** | **96%** |

### Context Classification by Type

| Type | Accuracy | Patterns |
|------|----------|----------|
| Confirmed | 97% | 138 |
| Negated | 99% | 99 |
| Uncertain | 91% | 48 |
| Historical | 95% | 82 |
| Family | 88% | 79 |

---

## System Architecture: Hybrid Approach

```
┌─────────────────────────────────────┐
│         Clinical Text Input         │
└────────────────┬────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
   ┌────▼─────┐    ┌─────▼────┐
   │ BioBERT  │    │ Template │
   │ Models   │    │ Matching │
   └────┬─────┘    └─────┬────┘
        │                │
        └────────┬────────┘
                 │
           ┌─────▼──────┐
           │ Confidence │
           │  Scoring   │
           └─────┬──────┘
                 │
           ┌─────▼──────┐
           │   Output   │
           └────────────┘
```

---

## BioBERT Models

### Three Specialized Models

| Model | Purpose | Accuracy |
|-------|---------|----------|
| BC5CDR-disease | Disease detection | 91.2% |
| BC5CDR-chem | Drug/chemical | 89.7% |
| BC5CDR-gene | Gene/protein | 88.3% |

**Why BioBERT?**
- Pre-trained on 18B biomedical words
- Understands medical context
- Handles abbreviations

---

## Template-Based Boosting

### 57,476 Curated Medical Terms

| Category | Terms | Source |
|----------|-------|--------|
| Diseases | 42,000+ | ICD-10, SNOMED CT |
| Chemicals | 5,242 | RxNorm, FDA |
| Genes | 10,234 | HGNC, UniProt |

**Why Templates?**
- Catches entities BioBERT misses
- Exact matching = high precision
- Domain-specific coverage

---

## 5-Stage Processing Pipeline

```
Stage 1: Base NLP (spaCy)
    ↓
Stage 2: Entity Extraction (BioBERT + Templates)
    ↓
Stage 3: Context Classification (446 patterns)
    ↓
Stage 4: Section Detection (20+ sections)
    ↓
Stage 5: Output Generation (43-column Excel)
```

---

## Stage 1-2: NLP and Entity Extraction

### Stage 1: Base NLP (spaCy)
- Tokenization
- Sentence segmentation
- POS tagging
- Dependency parsing

### Stage 2: Entity Extraction
- Run 3 BioBERT models
- Template matching (57,476 terms)
- Word boundary validation
- Confidence scoring

---

## Stage 3: Context Classification

### 5 Context Types

| Type | Patterns | Example |
|------|----------|---------|
| Confirmed | 138 | "has diabetes" |
| Negated | 99 | "denies fever" |
| Uncertain | 48 | "possible MI" |
| Historical | 82 | "history of CAD" |
| Family | 79 | "mother has cancer" |

**Total: 446 patterns**

---

## Stage 4-5: Section Detection & Output

### Clinical Section Detection
- Chief Complaint, HPI, PMH
- Medications, Allergies
- Physical Exam, Labs
- Assessment, Plan

### Output Generation
- 43-column Excel
- Streamlit dashboard
- JSON export
- HTML visualization

---

## Scope Reversal Detection

### 103 Patterns for Complex Sentences

**Problem**: "Patient denies fever but reports cough"

| Standard NER | Our Pipeline |
|--------------|--------------|
| fever: Negated | fever: NEGATED ✓ |
| cough: Negated ❌ | cough: CONFIRMED ✓ |

**Reversal Triggers**: but, however, yet, although, except

---

## Scope Reversal Example

```
Input: "Patient denies chest pain but reports dyspnea"

Step 1: Detect entities
  - "chest pain", "dyspnea"

Step 2: Find reversal trigger
  - "but" detected between entities

Step 3: Split into scopes
  - Scope 1: "denies chest pain" → NEGATED
  - Scope 2: "reports dyspnea" → CONFIRMED

Output:
  - chest pain [NEGATED, 0.98]
  - dyspnea [CONFIRMED, 0.95]
```

---

## Template-Priority Mode

### When to Trust Templates Over BioBERT

```python
def merge_entities(biobert, templates):
    for template_entity in templates:
        if template_entity.confidence > 0.9:
            merged.append(template_entity)
            remove_overlapping(biobert, template_entity)
    merged.extend(biobert)
    return merged
```

- Templates: 100% precision on exact matches
- BioBERT: Better recall for variations

---

## Confidence Scoring Algorithm

### Multi-Factor Calculation

```
Score = Strength (40) + Proximity (40) + Structure (20)

Strength Points:
  - Strong pattern (has, denies): 40
  - Moderate pattern: 25
  - Weak pattern: 10

Proximity Points:
  - ≤5 chars: 40
  - ≤10 chars: 35
  - ≤20 chars: 25
  - ≤35 chars: 15
```

---

## Word Boundary Validation

### Preventing False Positives

**Problem**: "pain" matching in "spain"

```python
pattern = r'\b' + re.escape(entity_text) + r'\b'

# Examples:
has_word_boundaries("spain", "pain")      # False ✓
has_word_boundaries("chest pain", "pain") # True ✓
```

**Impact**: 84% reduction in false positives

---

## Context Priority Resolution

### Handling Multiple Context Signals

```python
PRIORITY = {
    'NEGATED': 5,      # Highest
    'FAMILY': 4,
    'HISTORICAL': 3,
    'UNCERTAIN': 2,
    'CONFIRMED': 1     # Default
}
```

**Example**: "no family history of diabetes"
- Family pattern detected
- Negation pattern detected
- Result: NEGATED (higher priority)

---

## False Positive Suppression

### Filtering Low-Quality Matches

- Too short (<3 chars)
- Generic terms without context
- Low confidence (<0.3)
- Exclusion list (pronouns, common words)

**Result**: 92% noise reduction

---

## Overlapping Entity Resolution

### Choosing Best Entity

```python
# When multiple entities overlap:
# Keep: longest, highest confidence

"chest pain" vs "pain"
  → Keep "chest pain" (longer, more specific)
```

---

## Technology Stack

| Category | Technologies |
|----------|--------------|
| NLP | spaCy 3.7+, scispaCy, negspacy |
| ML | BioBERT, Hugging Face, PyTorch |
| Data | pandas, openpyxl |
| Web | Streamlit 1.28+ |
| Templates | 57,476 curated terms |

---

## Template File Structure

```
templates/
├── diseases/
│   ├── cardiovascular.txt (3,245)
│   ├── respiratory.txt (2,890)
│   └── neurological.txt (4,123)
├── medications/
│   ├── antibiotics.txt (1,234)
│   └── antihypertensives.txt (890)
└── genes/
    └── oncogenes.txt (1,500)
```

---

## Excel Output Format

### 43 Columns Including:

| Column | Description |
|--------|-------------|
| entity_text | Extracted entity |
| entity_type | Disease/Chemical/Gene |
| context | Confirmed/Negated/etc. |
| confidence | 0.0-1.0 score |
| predictor | Pattern that triggered |
| section | Clinical section |

---

## Streamlit App Features

- File upload (Excel, CSV, TXT)
- Manual text input
- Real-time processing
- Color-coded entity visualization
- Export to Excel/JSON
- Configurable detection options

---

## Performance Benchmarks

| Note Length | Processing Time |
|-------------|-----------------|
| Short (100 words) | 68ms |
| Medium (500 words) | 156ms |
| Long (2000 words) | 445ms |

**Batch**: 1000 notes in 2 min 14 sec (447/min)

---

## Demo: Sample Input

```
Chief Complaint: Chest pain

History: 55-year-old male with acute chest pain.
Denies shortness of breath but reports nausea.
No history of MI. Father had CAD.

Assessment: Possible acute coronary syndrome
```

---

## Demo: Sample Output

| Entity | Type | Context | Conf |
|--------|------|---------|------|
| chest pain | Disease | Confirmed | 0.94 |
| shortness of breath | Symptom | Negated | 0.97 |
| nausea | Symptom | Confirmed | 0.86 |
| MI | Disease | Historical+Neg | 0.92 |
| CAD | Disease | Family | 0.91 |
| acute coronary syndrome | Disease | Uncertain | 0.88 |

---

## Lessons Learned

1. **Hybrid > Pure ML**: +23% recall with templates
2. **Scope is Hard**: 103 patterns needed
3. **Context Matters**: Same entity, different meanings
4. **Word Boundaries Critical**: 84% false positive reduction
5. **Iterative Curation**: 5K → 57K templates

---

## Future Roadmap

**Q1 2025**
- FHIR integration
- Multi-language support

**Q2-Q4 2025**
- Institution-specific fine-tuning
- Temporal relationship extraction
- ICD-10 auto-coding
- UMLS entity linking

---

## Resources

**GitHub**: github.com/your-username/medical-ner-pipeline

**Key Metrics**:
- 96% entity detection
- 93% context classification
- 57,476 curated terms
- 103 scope reversal patterns

**Contact**: your-email@domain.com

---

# Thank You!

## Questions?

**Key Achievements**:
- 96% entity detection accuracy
- 93% context classification accuracy
- Production-ready Streamlit app

⭐ Star the repo if you found this useful!
