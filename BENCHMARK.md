# Benchmark

Implementasi acuan disediakan agar model-model baru dapat dibandingkan
terhadap baseline yang telah diketahui pada partisi yang dirilis. Seluruh
angka di bawah ini berasal dari satu kali eksekusi pada partisi `test` yang
didefinisikan dalam `splits.csv`.

## Prosedur

### Prapemrosesan

Teks dokumen disusun dari metadata lowongan, kemudian ditokenisasi
berdasarkan spasi (whitespace-tokenised) dan dikonversi menjadi tag IOB2
menggunakan offset karakter dari `gold_annotations`. Dokumen yang lebih
panjang dari 180 kata dipecah menjadi potongan (chunk) yang saling tumpang
tindih (overlapping) sehingga tidak ada entitas yang terpotong di tengah.
Proses chunking menghasilkan 4.919 chunk training, 879 chunk validation, dan
1.006 chunk test.

Pemartisian dilakukan pada tingkat **dokumen** menggunakan pengenal dalam
`splits.csv`. Pemartisian di bawah tingkat dokumen menyebabkan kebocoran
konten antar partisi dan menaikkan skor secara artifisial.

### Model

| Pengaturan | Nilai |
|---|---|
| Checkpoint pralatih | `indolem/indobert-base-uncased` |
| Task head | Token classification, 29 label IOB2 |
| Panjang sekuens maksimum | 256 subword token |
| Ukuran batch | 16 |
| Learning rate | 3 × 10⁻⁵, linear decay |
| Warm-up | 10% dari total langkah |
| Optimizer | AdamW (β₁ = 0,9, β₂ = 0,999, ε = 1 × 10⁻⁸) |
| Weight decay | 0,01 |
| Dropout | 0,1 (default checkpoint) |
| Epoch maksimum | 40, early stopping berdasarkan F1 validation dengan patience 9 |
| Class weights | Tidak digunakan — ablasi menunjukkan hal ini menurunkan F1 |
| Decoding | Viterbi dengan batasan transisi IOB2 |
| Random seed | 42 |
| Perangkat keras | NVIDIA Tesla T4 |

### Decoding

Decoding argmax biasa menghasilkan sekuens IOB2 yang secara struktural tidak
valid — tag `I-X` yang mengikuti tag dari jenis berbeda. Decoding Viterbi
dengan batasan (constrained) melarang transisi semacam ini.

| Decoding | Micro F1 | Sekuens tidak valid |
|---|---:|---:|
| Argmax | 0,7102 | 754 token |
| Constrained Viterbi | **0,7264** | 0 |

Constrained decoding digunakan untuk seluruh hasil di bawah ini.

### Evaluasi

Precision, recall, dan F1 tingkat entitas dihitung dengan `seqeval` di bawah
strict matching: sebuah prediksi dihitung benar hanya jika batas (boundary)
dan labelnya sama persis dengan gold span. Partial matching, yang memberi
kredit untuk tumpang tindih apa pun dengan label yang benar, dilaporkan
sebagai pembanding acuan atas (upper reference).

## Hasil

### Keseluruhan

| Metrik | Baseline kamus | IndoBERT |
|---|---:|---:|
| Akurasi token | 0,3826 | **0,8951** |
| Micro F1 (strict) | 0,116 | **0,7264** |
| Micro F1 (partial) | — | **0,8588** |
| Macro F1 | 0,146 | **0,724** |

Ada dua batas bawah yang perlu dicatat. Tagger trivial yang memprediksi `O`
untuk setiap token mencapai akurasi token 0,4892, karena sebagian besar token
berada di luar entitas mana pun; akurasi token saja karena itu bukan ukuran
kualitas yang bermakna. Baseline kamus — pencarian gazetteer atas surface
form dari training set — mencapai 0,116 strict micro F1.

### Per jenis entitas

Skor tingkat entitas secara strict pada partisi test. Support adalah jumlah
gold span.

| Entitas | Precision | Recall | F1 | Support | Akurasi token |
|---|---:|---:|---:|---:|---:|
| EXPERIENCE_YEARS | 0,949 | 0,966 | **0,957** | 440 | 0,9994 |
| DEGREE_FIELD | 0,878 | 0,927 | **0,902** | 535 | 0,9989 |
| EDUCATION_LEVEL | 0,835 | 0,829 | **0,832** | 708 | 0,9983 |
| SOFT_SKILL | 0,769 | 0,813 | **0,791** | 2.804 | 0,9841 |
| TOOL | 0,790 | 0,757 | **0,773** | 1.072 | 0,9936 |
| LOCATION | 0,736 | 0,804 | **0,768** | 460 | 0,9974 |
| COMPANY | 0,745 | 0,754 | **0,750** | 252 | 0,9977 |
| JOB_TITLE | 0,689 | 0,698 | **0,694** | 592 | 0,9950 |
| RESPONSIBILITY | 0,676 | 0,697 | **0,686** | 4.028 | 0,9492 |
| EMPLOYMENT_TYPE | 0,649 | 0,673 | **0,661** | 55 | 0,9995 |
| PROGRAMMING_LANGUAGE | 0,536 | 0,750 | **0,625** | 20 | 0,9999 |
| INDUSTRY | 0,645 | 0,598 | **0,621** | 376 | 0,9963 |
| SKILL | 0,547 | 0,618 | **0,581** | 1.933 | 0,9588 |
| SALARY | 0,483 | 0,519 | **0,500** | 27 | 0,9995 |
| **Rata-rata micro** | **0,712** | **0,742** | **0,726** | **13.302** | **0,8951** |
| Rata-rata macro | 0,709 | 0,743 | 0,724 | 13.302 | — |

Perhatikan celah antara akurasi token dan F1. `PROGRAMMING_LANGUAGE` mencapai
akurasi token 0,9999 namun F1 hanya 0,625: entitas ini hanya muncul pada 20
span test, sehingga model dapat memperoleh akurasi hampir sempurna sementara
salah pada sebagian besar span tersebut. F1 tingkat entitas adalah metrik
yang lebih bermakna.

### Khusus entitas keahlian (skill)

Untuk aplikasi analisis pasar tenaga kerja hilir, keempat jenis entitas yang
bersifat "keahlian" menjadi perhatian utama.

| Metrik | Nilai |
|---|---:|
| Micro F1 (strict) | 0,7151 |
| Micro F1 (partial) | 0,8216 |
| Akurasi token | 0,9547 |
| Gold spans | 5.829 |

## Analisis kesalahan

Dari 14.476 span yang diprediksi dan gold pada partisi test:

| Jenis kesalahan | Jumlah | % dari kesalahan | % dari seluruh span |
|---|---:|---:|---:|
| Kesalahan batas (boundary) | 2.424 | 52,6% | 16,7% |
| False positive | 1.175 | 25,5% | 8,1% |
| Entitas terlewat (missed) | 675 | 14,6% | 4,7% |
| Kesalahan label | 337 | 7,3% | 2,3% |
| **Total** | **4.611** | **100%** | **31,9%** |

Span yang cocok dengan benar: 9.865.

Kesalahan batas mendominasi. Model biasanya menemukan entitasnya, tetapi
tidak sepakat mengenai di mana entitas itu mulai atau berakhir — konsekuensi
dari frasa nomina Bahasa Indonesia yang tidak memiliki penanda batas
eksplisit.

### Kebingungan label yang paling sering terjadi

| Gold | Prediksi | Jumlah |
|---|---|---:|
| TOOL | SKILL | 58 |
| SKILL | SOFT_SKILL | 35 |
| JOB_TITLE | SKILL | 32 |
| SKILL | JOB_TITLE | 31 |
| INDUSTRY | SKILL | 28 |
| SKILL | RESPONSIBILITY | 26 |
| SKILL | TOOL | 25 |

Kebingungan antara `SKILL`, `TOOL`, dan `PROGRAMMING_LANGUAGE` mencakup 88
kasus. Hal ini mencerminkan batas definisi yang memang genuine, bukan
kesalahan acak: *software editing* menandakan sebuah instrumen melalui kata
inti (head noun)-nya, tetapi sebuah kompetensi melalui pewatasnya
(modifier). Lihat `ENTITY_DEFINITIONS.md` untuk konvensi yang digunakan.

## Mereproduksi angka-angka ini

1. Muat `gold_annotations_dataset.json` dan tetapkan setiap dokumen ke
   sebuah partisi menggunakan `splits.csv`.
2. Konversi offset karakter menjadi tag IOB2 dan lakukan chunking dokumen
   pada 180 kata.
3. Fine-tune `indolem/indobert-base-uncased` dengan pengaturan di atas.
4. Lakukan decoding dengan batasan transisi IOB2.
5. Hitung skor dengan `seqeval` di bawah strict matching.

Reproduksi persis hingga desimal terakhir tidak dijamin: pelatihan GPU tidak
bersifat bit-deterministic antar eksekusi meskipun seed sudah ditetapkan. Dua
eksekusi pipeline ini berbeda sekitar 0,008 pada micro F1. Laporkan seed dan
eksekusi yang Anda gunakan.
