# Definisi Entitas

Korpus ini menggunakan 14 jenis entitas yang dikodekan dalam format IOB2,
menghasilkan 29 tag (`B-`/`I-` per jenis, ditambah `O`). Semua contoh di
bawah ini adalah span nyata yang diambil dari anotasi yang dirilis.

Berkas ini adalah ringkasan. Sumber otoritatif adalah
`ANNOTATION_GUIDELINE.docx`, dokumen yang diberikan kepada para anotator;
jika keduanya berbeda, panduan (guideline) yang berlaku.

| # | Entitas | Definisi | Contoh |
|---|---|---|---|
| 1 | `JOB_TITLE` | Jabatan pekerjaan dari posisi yang diiklankan, atau jabatan lain yang disebutkan dalam iklan. Tidak termasuk nama perusahaan dan nama departemen. | *Administrator HR*, *CS Admin*, *Technical Support*, *Desk Collection* |
| 2 | `RESPONSIBILITY` | Tugas atau kewajiban yang diharapkan dilakukan oleh pemegang jabatan. Biasanya berupa frasa verba; umumnya merupakan span terpanjang dalam korpus. | *Mendampingi driver saat pengiriman*, *Membersihkan bagian dalam kendaraan*, *sortir barang di gudang* |
| 3 | `SKILL` | Kompetensi teknis atau spesifik-pekerjaan yang dapat dilatih atau disertifikasi. Dibedakan dari `SOFT_SKILL` karena terikat pada suatu domain, bukan bersifat disposisional. | *rekrutmen*, *interviewing*, *pengembangan karyawan*, *evaluasi pelatihan* |
| 4 | `SOFT_SKILL` | Sifat interpersonal, disposisi, atau kualitas perilaku yang dapat dipindahtangankan (transferable). Tidak terikat pada domain tertentu. | *komunikasi*, *pengelolaan waktu*, *ramah*, *bertanggung jawab* |
| 5 | `TOOL` | Produk perangkat lunak, platform, mesin, atau instrumen bernama yang digunakan untuk melakukan pekerjaan. Produk bernama, bukan kompetensi umum. | *Microsoft Excel*, *Google Sheets*, *CRM*, *G-Suite* |
| 6 | `PROGRAMMING_LANGUAGE` | Bahasa pemrograman atau bahasa markup yang disebutkan secara eksplisit. Dipisahkan dari `TOOL` karena bahasa bukan produk. | *Python*, *Golang*, *TypeScript*, *PHP*, *C/C++* |
| 7 | `EDUCATION_LEVEL` | Tingkat pendidikan formal minimum atau yang diutamakan, dinyatakan sebagai jenjang kualifikasi Indonesia. | *S1*, *D3*, *SMA*, *SMK*, *Paket C*, *Semester akhir* |
| 8 | `DEGREE_FIELD` | Bidang atau jurusan studi yang disyaratkan atau diutamakan, terlepas dari jenjangnya. | *Psikologi*, *PAUD*, *Manajemen SDM*, *Administrasi Bisnis* |
| 9 | `EXPERIENCE_YEARS` | Pernyataan mengenai pengalaman kerja yang disyaratkan, baik berupa durasi maupun penanda eksplisit untuk posisi entry-level. | *1 tahun*, *3 hingga 5 tahun*, *fresh graduate*, *Lulusan Baru dipersilakan* |
| 10 | `EMPLOYMENT_TYPE` | Bentuk pengaturan kontrak atau jadwal kerja dari posisi tersebut. | *fulltime*, *Part-time*, *Freelance*, *Full remote*, *Harian* |
| 11 | `SALARY` | Angka atau rentang kompensasi yang dinyatakan, termasuk periodenya jika disebutkan. | *Rp6.000.000 / bulan*, *Rp6.000.000 - Rp10.000.000 /bulan*, *Rp95.000 / hari* |
| 12 | `LOCATION` | Lokasi geografis tempat kerja pada tingkat administratif apa pun. | *Jakarta Utara*, *Jawa Timur*, *Gempol*, *Depok* |
| 13 | `COMPANY` | Nama organisasi yang membuka lowongan atau organisasi lain yang disebutkan dalam iklan. | *PT Azandhi Global Partner*, *IMFI*, *Yayasan Akses Generasi Berlian* |
| 14 | `INDUSTRY` | Sektor ekonomi atau lini bisnis dari pemberi kerja. | *F&B*, *Retail*, *e-commerce*, *klinik kecantikan*, *multifinance* |

## Prioritas label

Ketika sebuah span secara masuk akal dapat menerima lebih dari satu label,
panduan menetapkan urutan berikut. Label yang tercantum paling atas dan dapat
diterapkan yang menang.

`PROGRAMMING_LANGUAGE` → `TOOL` → `EDUCATION_LEVEL` → `DEGREE_FIELD` →
`JOB_TITLE` → `COMPANY` → `LOCATION` → `RESPONSIBILITY` → `SOFT_SKILL` →
`SKILL` → `INDUSTRY`

## Aturan batas span

- Ambil frasa lengkap, bukan potongan kata.
- `SKILL`, `TOOL`, dan `SOFT_SKILL` harus berupa frasa yang ringkas (concise).
- `RESPONSIBILITY` boleh lebih panjang tetapi harus mencakup **satu**
  aktivitas. Beberapa tugas dalam satu kalimat dipisah menjadi span yang
  berbeda.
- Jangan melabeli kata benda telanjang (bare noun) seperti *laporan*,
  *konten*, atau *marketing* sebagai `RESPONSIBILITY`.
- Entitas yang bersarang (nested) di dalam sebuah responsibility tetap dapat
  dianotasi secara terpisah. Pada *Melakukan analisis data menggunakan Google
  Analytics*, keseluruhan frasa adalah `RESPONSIBILITY` dan *Google
  Analytics* juga merupakan `TOOL`.
- Entitas yang dikoordinasikan (coordinated) dianotasi secara terpisah.
  *Next.js, TypeScript, dan Docker* menghasilkan tiga span, bukan satu.
- `S1 Arsitektur` dipisah menjadi `EDUCATION_LEVEL` (*S1*) dan `DEGREE_FIELD`
  (*Arsitektur*).

## Kasus ambigu yang telah diadjudikasi

Panduan menetapkan keputusan-keputusan spesifik berikut agar korpus tetap
konsisten secara internal.

| Bentuk permukaan | Label | Alasan |
|---|---|---|
| *negosiasi* (berdiri sendiri) | `SOFT_SKILL` | Dibaca sebagai disposisi interpersonal |
| *negosiasi kontrak*, *teknik negosiasi* | `SKILL` | Terikat domain dan dapat dilatih |
| *presentasi* | `SKILL` | — |
| *komunikasi*, *komunikasi efektif* | `SOFT_SKILL` | — |
| *complaint handling* | `SKILL` | Kecuali jelas-jelas ditulis sebagai sebuah sifat |
| *customer focus* | `SOFT_SKILL` | — |

Kata benda telanjang tidak dianotasi sendirian: gunakan *kerja tim*, bukan
*tim*, dan *berorientasi pada target*, bukan *target*.

## Perbedaan yang mudah tertukar

- **`SKILL` versus `TOOL`.** Produk perangkat lunak, platform, framework,
  library, atau database bernama adalah `TOOL`. Kompetensi yang dapat
  dipelajari adalah `SKILL`.
- **`TOOL` versus `PROGRAMMING_LANGUAGE`.** Framework, library, dan platform
  adalah `TOOL`; hanya bahasa pemrograman, bahasa scripting, dan bahasa query
  yang merupakan `PROGRAMMING_LANGUAGE`.
- **`SKILL` versus `SOFT_SKILL`.** Sifat interpersonal, sikap kerja, dan
  kualitas pribadi adalah `SOFT_SKILL`; kompetensi teknis, profesional, dan
  prosedural adalah `SKILL`.
- **`SKILL` versus `RESPONSIBILITY`.** Sebuah keahlian adalah sesuatu yang
  dimiliki kandidat; sebuah responsibility adalah tugas yang disyaratkan oleh
  peran tersebut.
- **`SKILL` versus `INDUSTRY`.** Sektor bisnis yang luas adalah `INDUSTRY`,
  bukan sebuah kompetensi.

## Adjudikasi

Ketika anotator berbeda pendapat, span diselesaikan sebagai berikut:

- **Dua anotator sepakat, satu tidak memberi label** — diterima jika span
  tersebut mengikuti panduan, tidak terlalu generik, memiliki batas yang
  benar, dan membawa label yang tepat.
- **Hanya satu anotator yang menandai** — tidak diterima secara otomatis.
  Diterima hanya jika span tersebut jelas merupakan entitas, berlabel benar,
  dan informatif; ditolak jika pendek dan generik, tidak bermakna tanpa
  konteks, atau labelnya terkesan dipaksakan.
- **Anotator berbeda pendapat mengenai label** — diselesaikan menggunakan
  definisi dan urutan prioritas di atas.

Hasil dan alasan untuk setiap span disimpan pada kolom `decision` dan `note`
dalam anotasi yang dirilis.

## Tag set

Pengkodean IOB2 digunakan. `B-X` menandai token pertama dari sebuah entitas
jenis `X`, `I-X` menandai kelanjutannya, dan `O` menandai token di luar
entitas mana pun. Dengan 14 jenis, ini menghasilkan 29 tag.

Sebuah tag `I-X` hanya boleh mengikuti tag `B-X` atau `I-X` dari jenis yang
sama. Sekuens yang melanggar batasan ini tidak valid; anotasi yang dirilis
tidak mengandung sekuens semacam itu.
