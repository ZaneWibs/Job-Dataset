# Indonesian Job Advertisement NER Dataset

A manually annotated corpus of **5,121 Indonesian job advertisements** containing
**96,077 entity spans** across **14 entity types**, collected from Glints
Indonesia and annotated by four annotators.

The corpus supports named entity recognition for labour-market analysis:
extracting job titles, skills, tools, qualifications, and employment conditions
from unstructured Indonesian job postings.

## Contents

| File | Size | Description |
|---|---|---|
| `gold_annotations_dataset.json` | 27.3 MB | Full dataset: 5,121 documents with gold-standard annotations and 31 metadata fields per posting |

## Entity types

| Entity | Spans | Documents | Coverage |
|---|---:|---:|---:|
| RESPONSIBILITY | 27,999 | 4,331 | 84.6% |
| SOFT_SKILL | 19,326 | 4,065 | 79.4% |
| SKILL | 16,057 | 3,909 | 76.3% |
| TOOL | 8,914 | 2,574 | 50.3% |
| EDUCATION_LEVEL | 4,668 | 3,408 | 66.5% |
| JOB_TITLE | 3,914 | 2,385 | 46.6% |
| DEGREE_FIELD | 3,503 | 1,498 | 29.3% |
| LOCATION | 3,305 | 1,944 | 38.0% |
| EXPERIENCE_YEARS | 2,974 | 2,644 | 51.6% |
| INDUSTRY | 2,910 | 1,689 | 33.0% |
| COMPANY | 1,801 | 1,210 | 23.6% |
| EMPLOYMENT_TYPE | 336 | 268 | 5.2% |
| PROGRAMMING_LANGUAGE | 191 | 82 | 1.6% |
| SALARY | 179 | 143 | 2.8% |

Coverage is the share of the 5,121 documents in which the entity appears at
least once. Counts are of raw annotated spans; a small number are removed by
structural validation (duplicate and misaligned spans) before model training.

## Record structure

```json
{
  "doc_id": 1,
  "url": "https://glints.com/id/opportunities/jobs/...",
  "data": {
    "title": "Helper Driver Pengiriman",
    "company": "...",
    "location": "...",
    "province": "...",
    "industry": "...",
    "description": "...",
    "requirements": "...",
    "benefits": "...",
    "salary_min": 3000000,
    "salary_max": 4000000,
    "job_type": "Full-Time",
    "work_mode": "Onsite",
    "education_level": "...",
    "experience_min_years": 1,
    "posted_date": "2025-11-04",
    "...": "31 metadata fields in total"
  },
  "gold_annotations": [
    {
      "start": 3,
      "end": 38,
      "text": "Membantu proses loading & unloading",
      "label": "RESPONSIBILITY",
      "decision": "ACCEPT",
      "note": "2 anotator setuju label RESPONSIBILITY"
    }
  ],
  "n_gold_spans": "18"
}
```

Top-level fields:

- **`doc_id`** — sequential identifier, 1 to 5121
- **`url`** — canonical source URL; unique across the corpus and used as the
  document key when building train/validation/test splits
- **`data`** — 31 metadata fields scraped from the source posting
- **`gold_annotations`** — list of adjudicated entity spans
- **`n_gold_spans`** — number of spans in that document

Span fields:

- **`start`**, **`end`** — character offsets into the document text
- **`text`** — the surface string of the span
- **`label`** — one of the 14 entity types
- **`decision`** — adjudication outcome for the span
- **`note`** — adjudication rationale, including how many annotators agreed

## Loading the data

```python
import json

with open("gold_annotations_dataset.json", encoding="utf-8") as f:
    docs = json.load(f)

print(f"{len(docs):,} documents")
print(f"{sum(int(d['n_gold_spans']) for d in docs):,} annotated spans")

# All spans of one entity type
skills = [
    span
    for doc in docs
    for span in doc["gold_annotations"]
    if span["label"] == "SKILL"
]
print(f"{len(skills):,} SKILL spans")
```

## Annotation

Four annotators labelled the corpus following a written guideline covering the
14 entity types. A subset of documents was double-annotated, and disagreements
resolved by adjudication. The outcome and rationale for each span are kept in
the `decision` and `note` fields.

## Splitting the data

Split at the **document** level, not the span or sentence level. Documents are
long and a single posting yields many spans, so splitting below the document
level leaks content between training and test sets and inflates evaluation
scores. Use `url` or `doc_id` as the split key.

```python
from sklearn.model_selection import train_test_split

ids = [d["doc_id"] for d in docs]
train_ids, temp_ids = train_test_split(ids, test_size=0.28, random_state=42)
val_ids, test_ids = train_test_split(temp_ids, test_size=0.54, random_state=42)
```
