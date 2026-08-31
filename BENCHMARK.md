# Benchmark

Implementasi acuan disertakan agar model lain dapat dibandingkan pada
pembagian data yang sama. Seluruh angka di bawah dihitung pada bagian `test`
sebagaimana tercantum di `splits.csv`.

## Prosedur

### Prapemrosesan

Teks dokumen disusun dari kolom metadata lowongan, ditokenisasi berdasarkan
spasi, lalu dikonversi ke penandaan IOB2 memakai posisi karakter pada
`gold_annotations`. Dokumen yang lebih panjang dari 180 kata dipecah menjadi
potongan bertumpang tindih agar tidak ada entitas yang terpotong di tengah.

Pembagian data dilakukan pada tingkat **dokumen** memakai pengenal di
`splits.csv`. Membagi di bawah tingkat dokumen menyebabkan kebocoran antar
bagian dan membuat skor tampak lebih tinggi daripada yang sebenarnya.

### Model

| Pengaturan | Nilai |
|---|---|
| Checkpoint pralatih | `indolem/indobert-base-uncased` |
| Kepala tugas | Klasifikasi token, 29 label IOB2 |
| Panjang urutan maksimum | 256 token subkata |
| Ukuran batch | 16 |
| Laju pembelajaran | 3 × 10⁻⁵, peluruhan linear |
| Pemanasan | 10% dari total langkah |
| Pengoptimal | AdamW (β₁ = 0,9; β₂ = 0,999; ε = 1 × 10⁻⁸) |
| Peluruhan bobot | 0,01 |
| Dropout | 0,1 (bawaan checkpoint) |
| Epoch maksimum | 40, penghentian dini pada F1 validasi dengan kesabaran 9 |
| Bobot kelas | Tidak dipakai — ablasi menunjukkan F1 justru menurun |
| Dekode | Viterbi dengan kendala transisi IOB2 |
| Seed acak | 42 |
| Perangkat keras | NVIDIA Tesla T4 |

### Dekode

Dekode argmax biasa menghasilkan urutan IOB2 yang tidak sah secara struktural,
yaitu penanda `I-X` yang mengikuti penanda berjenis lain. Dekode Viterbi
berkendala melarang transisi semacam itu, dan menaikkan F1 mikro sebesar
0,0161 sekaligus menghapus seluruh urutan tidak sah. Dekode berkendala dipakai
untuk semua hasil di bawah ini.

### Evaluasi

Presisi, recall, dan F1 pada tingkat entitas dihitung dengan `seqeval` memakai
pencocokan ketat (*strict*): sebuah prediksi dianggap benar hanya bila batas
span **dan** labelnya sama persis dengan span acuan.

## Hasil

### Keseluruhan

| Metrik | Baseline kamus | IndoBERT |
|---|---:|---:|
| Akurasi token | 0,3801 | **0,8965** |
| Presisi mikro | 0,071 | **0,713** |
| Recall mikro | 0,282 | **0,734** |
| F1 mikro | 0,113 | **0,723** |
| F1 makro | 0,146 | **0,721** |

Baseline kamus bekerja dengan pencarian gazetteer atas bentuk permukaan yang
muncul di bagian train.

Perlu dicatat bahwa akurasi token bukan ukuran mutu yang bermakna pada tugas
ini, karena sebagian besar token berada di luar entitas mana pun. Perhatikan
tabel berikut: `PROGRAMMING_LANGUAGE` mencapai akurasi token 0,9999 tetapi F1
hanya 0,636. Gunakan F1 tingkat entitas.

### Per jenis entitas

| Entitas | Presisi | Recall | F1 | Akurasi token |
|---|---:|---:|---:|---:|
| EXPERIENCE_YEARS | 0,951 | 0,966 | **0,958** | 0,9995 |
| DEGREE_FIELD | 0,876 | 0,921 | **0,898** | 0,9989 |
| EDUCATION_LEVEL | 0,833 | 0,815 | **0,824** | 0,9982 |
| SOFT_SKILL | 0,764 | 0,810 | **0,786** | 0,9840 |
| TOOL | 0,811 | 0,745 | **0,777** | 0,9937 |
| LOCATION | 0,729 | 0,796 | **0,761** | 0,9973 |
| COMPANY | 0,733 | 0,675 | **0,703** | 0,9976 |
| RESPONSIBILITY | 0,683 | 0,704 | **0,693** | 0,9507 |
| JOB_TITLE | 0,682 | 0,674 | **0,678** | 0,9948 |
| EMPLOYMENT_TYPE | 0,673 | 0,673 | **0,673** | 0,9995 |
| PROGRAMMING_LANGUAGE | 0,583 | 0,700 | **0,636** | 0,9999 |
| INDUSTRY | 0,632 | 0,566 | **0,597** | 0,9961 |
| SKILL | 0,544 | 0,598 | **0,570** | 0,9593 |
| SALARY | 0,560 | 0,519 | **0,539** | 0,9995 |
| **Rata-rata mikro** | **0,713** | **0,734** | **0,723** | **0,8965** |
| Rata-rata makro | 0,718 | 0,726 | 0,721 | 0,9906 |

Entitas dengan bentuk permukaan yang teratur — durasi pengalaman, jenjang
pendidikan, bidang studi — memperoleh skor tertinggi. Entitas dengan batas
definisi yang kabur, terutama `SKILL`, memperoleh skor terendah.

## Analisis kesalahan

Dari 14.427 span prediksi dan span acuan pada bagian test:

| Jenis kesalahan | Jumlah | % dari kesalahan | % dari seluruh span |
|---|---:|---:|---:|
| Kesalahan batas span | 2.432 | 52,2% | 16,9% |
| Entitas palsu | 1.126 | 24,2% | 7,8% |
| Entitas terlewat | 759 | 16,3% | 5,3% |
| Kesalahan label | 341 | 7,3% | 2,4% |
| **Total** | **4.658** | **100%** | **32,3%** |

Span yang cocok tepat: 9.769.

Kesalahan batas span mendominasi. Model umumnya menemukan entitasnya, tetapi
berbeda pendapat tentang di mana span itu mulai dan berakhir — konsekuensi dari
frasa nomina bahasa Indonesia yang tidak memiliki penanda batas eksplisit.

### Kebingungan label paling sering

| Acuan | Prediksi | Jumlah | % dari kesalahan label |
|---|---|---:|---:|
| TOOL | SKILL | 57 | 16,7% |
| SKILL | SOFT_SKILL | 39 | 11,4% |
| SKILL | JOB_TITLE | 32 | 9,4% |
| JOB_TITLE | SKILL | 32 | 9,4% |
| INDUSTRY | SKILL | 27 | 7,9% |
| SKILL | TOOL | 24 | 7,0% |
| **Enam pasangan di atas** | | **211** | **61,9%** |

Kebingungan antara `SKILL`, `TOOL`, dan `PROGRAMMING_LANGUAGE` mencerminkan
batas definisi yang memang kabur, bukan kesalahan acak. Panduan anotasi
menetapkan urutan prioritas label untuk kasus semacam ini; lihat
`ENTITY_DEFINITIONS.md`.

## Mereproduksi angka ini

1. Muat `gold_annotations_dataset.json`, lalu tetapkan bagian tiap dokumen
   memakai `splits.csv`.
2. Konversi posisi karakter menjadi penandaan IOB2 dan pecah dokumen pada
   180 kata.
3. Latih `indolem/indobert-base-uncased` dengan pengaturan di atas.
4. Dekode dengan kendala transisi IOB2.
5. Hitung skor dengan `seqeval` memakai pencocokan ketat.

Reproduksi sampai desimal terakhir tidak dijamin. Pelatihan pada GPU tidak
identik bit-per-bit antar-eksekusi meskipun seed-nya sama; dua eksekusi
pipeline ini berselisih sekitar 0,008 pada F1 mikro. Cantumkan seed dan
eksekusi yang Anda pakai saat melaporkan hasil.