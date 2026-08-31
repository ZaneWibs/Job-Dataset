# Definisi Entitas

Korpus ini memakai 14 jenis entitas dengan penandaan IOB2, sehingga
menghasilkan 29 penanda (`B-`/`I-` untuk tiap jenis, ditambah `O`). Seluruh
contoh di bawah adalah span nyata yang diambil dari anotasi yang dirilis.

Berkas ini adalah ringkasan. Acuan resminya adalah
`ANNOTATION_GUIDELINE.docx`, yaitu dokumen yang diberikan kepada para
anotator; bila keduanya berbeda, panduan itulah yang berlaku.

| # | Entitas | Definisi | Contoh |
|---|---|---|---|
| 1 | `JOB_TITLE` | Nama posisi, jabatan, atau peran pekerjaan. Bukan kalimat tugas kerja. | *Administrator HR*, *CS Admin*, *Technical Support*, *Desk Collection* |
| 2 | `COMPANY` | Nama perusahaan, organisasi, merek, atau institusi pemberi kerja. | *PT Azandhi Global Partner*, *IMFI*, *Yayasan Akses Generasi Berlian* |
| 3 | `LOCATION` | Kota, provinsi, negara, area kerja, lokasi penempatan, atau lokasi kantor. | *Jakarta Utara*, *Jawa Timur*, *Gempol*, *Depok* |
| 4 | `EDUCATION_LEVEL` | Tingkat pendidikan formal. | *S1*, *D3*, *SMA*, *SMK*, *Paket C*, *Semester akhir* |
| 5 | `DEGREE_FIELD` | Bidang studi, jurusan, atau disiplin pendidikan. | *Psikologi*, *PAUD*, *Manajemen SDM*, *Administrasi Bisnis* |
| 6 | `EXPERIENCE_YEARS` | Durasi pengalaman kerja yang diminta, termasuk penanda tingkat pemula. | *1 tahun*, *3 hingga 5 tahun*, *fresh graduate*, *Lulusan Baru dipersilakan* |
| 7 | `RESPONSIBILITY` | Aktivitas, tugas, atau kewajiban yang harus dilakukan kandidat. Umumnya frasa verbal. | *Mendampingi driver saat pengiriman*, *Membersihkan bagian dalam kendaraan*, *sortir barang di gudang* |
| 8 | `SKILL` | Kemampuan teknis, profesional, atau prosedural yang dapat dipelajari dan relevan dengan pekerjaan. | *rekrutmen*, *interviewing*, *pengembangan karyawan*, *evaluasi pelatihan* |
| 9 | `SOFT_SKILL` | Kemampuan interpersonal, karakter, sikap kerja, atau kualitas personal. | *komunikasi*, *pengelolaan waktu*, *ramah*, *bertanggung jawab* |
| 10 | `TOOL` | Nama alat, perangkat lunak, aplikasi, platform, framework, pustaka, basis data, perangkat keras, atau sistem tertentu. | *Microsoft Excel*, *Google Sheets*, *CRM*, *G-Suite* |
| 11 | `PROGRAMMING_LANGUAGE` | Bahasa pemrograman, bahasa skrip, atau bahasa kueri. Bukan framework atau pustaka. | *Python*, *Golang*, *TypeScript*, *PHP*, *C/C++* |
| 12 | `INDUSTRY` | Bidang industri, sektor bisnis, atau domain pekerjaan yang lebih luas. | *F&B*, *Retail*, *e-commerce*, *klinik kecantikan*, *multifinance* |
| 13 | `EMPLOYMENT_TYPE` | Bentuk hubungan kerja atau pengaturan waktu kerja. | *fulltime*, *Part-time*, *Freelance*, *Full remote*, *Harian* |
| 14 | `SALARY` | Besaran atau rentang imbalan kerja, termasuk satuan waktunya. | *Rp6.000.000 / bulan*, *Rp6.000.000 - Rp10.000.000 /bulan*, *Rp95.000 / hari* |

## Prioritas label

Bila satu span tampak cocok dengan lebih dari satu label, panduan menetapkan
urutan berikut. Label yang posisinya paling atas dan berlaku, itulah yang
dipakai.

`PROGRAMMING_LANGUAGE` → `TOOL` → `EDUCATION_LEVEL` → `DEGREE_FIELD` →
`JOB_TITLE` → `COMPANY` → `LOCATION` → `RESPONSIBILITY` → `SOFT_SKILL` →
`SKILL` → `INDUSTRY`

## Aturan batas span

- Ambil frasa yang lengkap, bukan potongan kata.
- `SKILL`, `TOOL`, dan `SOFT_SKILL` harus berupa frasa yang ringkas.
- `RESPONSIBILITY` boleh lebih panjang, tetapi harus mencakup **satu**
  aktivitas kerja. Beberapa tugas dalam satu kalimat dipecah menjadi span
  terpisah.
- Kata benda tunggal seperti *laporan*, *konten*, atau *marketing* tidak
  dilabeli sebagai `RESPONSIBILITY`.
- Entitas yang berada di dalam sebuah responsibility tetap boleh dianotasi
  terpisah. Pada *Melakukan analisis data menggunakan Google Analytics*,
  seluruh frasa adalah `RESPONSIBILITY` dan *Google Analytics* juga `TOOL`.
- Entitas yang disebut berderet dianotasi satu per satu. *Next.js, TypeScript,
  dan Docker* menghasilkan tiga span, bukan satu.
- *S1 Arsitektur* dipisah menjadi `EDUCATION_LEVEL` (*S1*) dan `DEGREE_FIELD`
  (*Arsitektur*).

## Kasus ambigu yang sudah diputuskan

Panduan menetapkan keputusan berikut agar korpus tetap konsisten.

| Bentuk permukaan | Label | Alasan |
|---|---|---|
| *negosiasi* (berdiri sendiri) | `SOFT_SKILL` | Dibaca sebagai sifat interpersonal |
| *negosiasi kontrak*, *teknik negosiasi* | `SKILL` | Terikat domain dan dapat dipelajari |
| *presentasi* | `SKILL` | — |
| *komunikasi*, *komunikasi efektif* | `SOFT_SKILL` | — |
| *complaint handling* | `SKILL` | Kecuali jelas ditulis sebagai sifat interpersonal |
| *customer focus* | `SOFT_SKILL` | — |

Kata benda telanjang tidak dianotasi sendirian: gunakan *kerja tim*, bukan
*tim*; gunakan *berorientasi pada target*, bukan *target*.

## Pembedaan yang mudah tertukar

- **`SKILL` dan `TOOL`.** Nama perangkat lunak, platform, framework, pustaka,
  atau basis data adalah `TOOL`. Kemampuan yang dapat dipelajari adalah
  `SKILL`.
- **`TOOL` dan `PROGRAMMING_LANGUAGE`.** Framework, pustaka, dan platform
  masuk `TOOL`; hanya bahasa pemrograman, bahasa skrip, dan bahasa kueri yang
  masuk `PROGRAMMING_LANGUAGE`.
- **`SKILL` dan `SOFT_SKILL`.** Sifat interpersonal, sikap kerja, dan kualitas
  personal masuk `SOFT_SKILL`; kemampuan teknis, profesional, dan prosedural
  masuk `SKILL`.
- **`SKILL` dan `RESPONSIBILITY`.** Skill adalah sesuatu yang dimiliki
  kandidat; responsibility adalah tugas yang dituntut oleh posisinya.
- **`SKILL` dan `INDUSTRY`.** Sektor bisnis yang luas adalah `INDUSTRY`, bukan
  kemampuan.

## Adjudikasi

Bila para anotator berbeda pendapat, span diputuskan sebagai berikut.

- **Dua anotator setuju, satu mengosongkan.** Diterima bila span sesuai
  panduan, tidak terlalu umum, batasnya tepat, dan labelnya sesuai definisi.
- **Hanya satu anotator menandai.** Tidak diterima otomatis. Diterima hanya
  bila span jelas merupakan entitas, labelnya sesuai, dan informasinya penting;
  ditolak bila span terlalu pendek dan umum, tidak bermakna tanpa konteks, atau
  labelnya tampak dipaksakan.
- **Anotator berbeda label.** Diselesaikan memakai definisi di atas dan urutan
  prioritas label.

Hasil dan alasan adjudikasi untuk setiap span tersimpan pada kolom `decision`
dan `note` di berkas anotasi.

## Tag set

Penandaan IOB2 dipakai. `B-X` menandai token pertama dari entitas berjenis
`X`, `I-X` menandai kelanjutannya, dan `O` menandai token di luar entitas mana
pun. Dengan 14 jenis entitas, seluruhnya menjadi 29 penanda.

Penanda `I-X` hanya boleh mengikuti `B-X` atau `I-X` yang berjenis sama.
Urutan yang melanggar ketentuan ini dianggap tidak sah; anotasi yang dirilis
tidak memuat satu pun pelanggaran.