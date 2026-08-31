# Dataset NER Lowongan Kerja Bahasa Indonesia

Korpus yang dianotasi secara manual berisi **5.121 iklan lowongan kerja
berbahasa Indonesia** dengan **96.077 span entitas** dari **14 jenis
entitas**, dikumpulkan dari Glints Indonesia dan dianotasi oleh empat orang
anotator.

Korpus ini mendukung tugas pengenalan entitas bernama (NER) untuk analisis
pasar tenaga kerja: mengekstraksi jabatan pekerjaan, keahlian, perkakas
(tools), kualifikasi, dan kondisi kerja dari teks iklan lowongan kerja yang
tidak terstruktur.

## Isi

| Berkas | Ukuran | Deskripsi |
|---|---|---|
| `gold_annotations_dataset.json` | 27,3 MB | Anotasi: 5.121 dokumen dengan span emas (gold-standard) dan 31 kolom metadata per lowongan |
| `ENTITY_DEFINITIONS.md` | — | Definisi, contoh, dan konvensi batas untuk masing-masing dari 14 jenis entitas |
| `ANNOTATION_GUIDELINE.docx` | 37 KB | Panduan anotasi yang diberikan kepada keempat anotator |
| `splits.csv` | 526 KB | Partisi train/validation/test, satu baris per dokumen |
| `splits.json` | 39 KB | Partisi yang sama dalam bentuk daftar `doc_id`, beserta parameter yang digunakan |
| `BENCHMARK.md` | — | Implementasi acuan: prosedur, hyperparameter, dan hasil |

## Jenis entitas

| Entitas | Jumlah Span | Dokumen | Cakupan |
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
setidaknya satu kali. Jumlah yang ditampilkan adalah jumlah span mentah hasil
anotasi; sejumlah kecil span dihapus melalui validasi struktural (span
duplikat dan span dengan offset yang tidak sejajar) sebelum data digunakan
untuk melatih model.

## Struktur rekaman (record)

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
    "...": "31 kolom metadata secara total"
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

- **`doc_id`** — pengenal berurutan, 1 hingga 5121
- **`url`** — URL sumber kanonik; unik di seluruh korpus dan digunakan sebagai
  kunci dokumen saat membangun partisi train/validation/test
- **`data`** — 31 kolom metadata yang di-scrape dari iklan sumber
- **`gold_annotations`** — daftar span entitas yang telah diadjudikasi
- **`n_gold_spans`** — jumlah span dalam dokumen tersebut

Kolom span:

- **`start`**, **`end`** — offset karakter ke dalam teks dokumen
- **`text`** — string permukaan (surface string) dari span
- **`label`** — salah satu dari 14 jenis entitas
- **`decision`** — hasil adjudikasi untuk span tersebut
- **`note`** — alasan adjudikasi, termasuk berapa anotator yang sepakat

## Memuat data

```python
import json

with open("gold_annotations_dataset.json", encoding="utf-8") as f:
    docs = json.load(f)

print(f"{len(docs):,} documents")
print(f"{sum(int(d['n_gold_spans']) for d in docs):,} annotated spans")

# Semua span dari satu jenis entitas
skills = [
    span
    for doc in docs
    for span in doc["gold_annotations"]
    if span["label"] == "SKILL"
]
print(f"{len(skills):,} SKILL spans")
```

## Anotasi

Empat anotator melabeli korpus mengikuti panduan tertulis yang mencakup 14
jenis entitas. Sebagian dokumen dianotasi ganda (double-annotated), dan
perbedaan pendapat diselesaikan melalui adjudikasi. Hasil dan alasan untuk
setiap span disimpan pada kolom `decision` dan `note`.

## Partisi

Gunakan partisi yang dirilis agar hasil dapat dibandingkan antar studi.

| Partisi | Dokumen | Persentase |
|---|---:|---:|
| train | 3.699 | 72,2% |
| validation | 653 | 12,8% |
| test | 769 | 15,0% |

```python
import csv

with open("splits.csv", encoding="utf-8") as f:
    split_of = {int(r["doc_id"]): r["split"] for r in csv.DictReader(f)}

train = [d for d in docs if split_of[d["doc_id"]] == "train"]
test  = [d for d in docs if split_of[d["doc_id"]] == "test"]
```

Partisi dibuat pada tingkat **dokumen** menggunakan
`sklearn.model_selection.train_test_split`, seed 42, menyisihkan 15% untuk
test lalu 15% dari sisanya untuk validation. Pemartisian di bawah tingkat
dokumen — per span atau per kalimat — menyebabkan kebocoran konten antar
partisi dan menaikkan skor secara artifisial, karena satu lowongan
menghasilkan banyak span. Partisi yang dirilis tidak memiliki satu pun
dokumen yang muncul di lebih dari satu partisi.

## Benchmark

Implementasi acuan IndoBERT mencapai **0,7264 strict micro F1** pada partisi
test, dibandingkan 0,116 untuk baseline kamus (dictionary baseline). Prosedur
lengkap, hyperparameter, skor per entitas, dan analisis kesalahan ada di
`BENCHMARK.md`.
