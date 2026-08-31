# Dataset NER Lowongan Kerja Indonesia

Korpus teranotasi manual berisi **5.121 lowongan kerja berbahasa Indonesia**
dengan **96.077 span entitas** dalam **14 jenis entitas**, dikumpulkan dari
Glints Indonesia dan dianotasi oleh empat anotator.

Korpus ini ditujukan untuk pengenalan entitas bernama (*named entity
recognition*) dalam analisis pasar kerja: mengekstraksi judul pekerjaan,
keterampilan, perkakas, kualifikasi, dan kondisi kerja dari teks lowongan yang
tidak terstruktur.

## Isi

| Berkas | Ukuran | Keterangan |
|---|---|---|
| `gold_annotations_dataset.json` | 27,3 MB | Anotasi: 5.121 dokumen berisi span acuan dan 31 kolom metadata per lowongan |
| `ENTITY_DEFINITIONS.md` | — | Definisi, contoh, dan aturan batas span untuk 14 jenis entitas |
| `ANNOTATION_GUIDELINE.docx` | 37 KB | Panduan anotasi yang diberikan kepada keempat anotator |
| `splits.csv` | 526 KB | Pembagian train/validation/test, satu baris satu dokumen |
| `splits.json` | 39 KB | Pembagian yang sama dalam bentuk daftar `doc_id`, beserta parameternya |
| `BENCHMARK.md` | — | Implementasi acuan: prosedur, hiperparameter, dan hasilnya |

## Jenis entitas

| Entitas | Span | Dokumen | Cakupan |
|---|---:|---:|---:|
| RESPONSIBILITY | 27.999 | 4.331 | 84,6% |
| SOFT_SKILL | 19.326 | 4.065 | 79,4% |
| SKILL | 16.057 | 3.909 | 76,3% |
| TOOL | 8.914 | 2.574 | 50,3% |
| EDUCATION_LEVEL | 4.668 | 3.408 | 66,5% |
| JOB_TITLE | 3.914 | 2.385 | 46,6% |
| DEGREE_FIELD | 3.503 | 1.498 | 29,3% |
| LOCATION | 3.305 | 1.944 | 38,0% |
| EXPERIENCE_YEARS | 2.974 | 2.644 | 51,6% |
| INDUSTRY | 2.910 | 1.689 | 33,0% |
| COMPANY | 1.801 | 1.210 | 23,6% |
| EMPLOYMENT_TYPE | 336 | 268 | 5,2% |
| PROGRAMMING_LANGUAGE | 191 | 82 | 1,6% |
| SALARY | 179 | 143 | 2,8% |

Cakupan adalah persentase dari 5.121 dokumen yang memuat entitas tersebut
minimal satu kali. Angka span di atas dihitung dari anotasi mentah; sebagian
kecil dibuang pada tahap validasi struktural (span ganda dan span yang tidak
selaras dengan tokenisasi) sebelum pelatihan model.

## Struktur rekaman

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
    "...": "31 kolom metadata seluruhnya"
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

Kolom tingkat atas:

- **`doc_id`** — pengenal berurutan, 1 sampai 5121
- **`url`** — tautan sumber; unik di seluruh korpus dan dipakai sebagai kunci
  dokumen saat membagi data
- **`data`** — 31 kolom metadata hasil pengambilan dari sumber
- **`gold_annotations`** — daftar span entitas hasil adjudikasi
- **`n_gold_spans`** — jumlah span pada dokumen tersebut

Kolom pada tiap span:

- **`start`**, **`end`** — posisi karakter dalam teks dokumen
- **`text`** — teks span apa adanya
- **`label`** — salah satu dari 14 jenis entitas
- **`decision`** — hasil adjudikasi untuk span tersebut
- **`note`** — alasan adjudikasi, termasuk berapa anotator yang menyetujuinya

## Memuat data

```python
import json

with open("gold_annotations_dataset.json", encoding="utf-8") as f:
    docs = json.load(f)

print(f"{len(docs):,} dokumen")
print(f"{sum(int(d['n_gold_spans']) for d in docs):,} span teranotasi")

# Seluruh span dari satu jenis entitas
skills = [
    span
    for doc in docs
    for span in doc["gold_annotations"]
    if span["label"] == "SKILL"
]
print(f"{len(skills):,} span SKILL")
```

## Pembagian data

Gunakan pembagian yang disertakan agar hasilnya dapat dibandingkan antar
penelitian.

| Bagian | Dokumen | Persentase |
|---|---:|---:|
| train | 3.699 | 72,2% |
| validation | 653 | 12,8% |
| test | 769 | 15,0% |

```python
import csv

with open("splits.csv", encoding="utf-8") as f:
    bagian = {int(r["doc_id"]): r["split"] for r in csv.DictReader(f)}

train = [d for d in docs if bagian[d["doc_id"]] == "train"]
test  = [d for d in docs if bagian[d["doc_id"]] == "test"]
```

Pembagian dilakukan pada tingkat **dokumen** memakai
`sklearn.model_selection.train_test_split` dengan seed 42, menyisihkan 15%
untuk test lalu 15% dari sisanya untuk validation.

Membagi data di bawah tingkat dokumen — misalnya per span atau per kalimat —
menyebabkan kebocoran antar bagian dan membuat skor evaluasi tampak lebih baik
daripada yang sebenarnya, karena satu lowongan menghasilkan banyak span.
Pembagian yang disertakan di sini tidak memuat satu pun dokumen yang berada di
lebih dari satu bagian.

## Benchmark

Implementasi acuan dengan IndoBERT mencapai **F1 mikro strict 0,723** pada
bagian test, dibandingkan 0,113 untuk baseline berbasis kamus. Prosedur
lengkap, hiperparameter, skor per entitas, dan analisis kesalahan ada di
`BENCHMARK.md`.