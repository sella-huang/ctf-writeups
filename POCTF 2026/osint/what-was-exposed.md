# What Was Exposed

| | |
|---|---|
| **Bagian** | wave 6 |
| **Kategori** | osint |
| **Jumlah Solve** | 119 |

## Ikhtisar

Prompt memisahkan subjek dari target. Subjeknya adalah Janet Jackson — "penyanyi, penulis lagu, aktris, penari. Kambing hitam Super Bowl" — tetapi tugasnya dibingkai sebagai menemukan "seseorang yang mencari subjeknya". Seseorang tersebut adalah akun Reddit `Legal-Yoghurt2423`, yang memposting di `r/janetjackson`, mengklaim $17.000 terutang "untuk pelanggaran Robert Shepherd", dan memublikasikan alamat rumah kerabatnya "untuk mendiskusikan masalah tersebut". Alamatnya adalah apa yang dipaparkan dalam poster, dan itulah porosnya: pertanyaan sebenarnya adalah pertanyaan gambaran sejarah tentang pagar halaman belakang properti itu.

Pekerjaan ini dibagi menjadi tiga bagian: memulihkan aktivitas Reddit pembuat poster, menyelesaikan alamat ke paket yang tepat, dan membaca posisi pagar dari rekaman sejarah Google Earth.

## Penyelesaian

Menelusuri `r/janetjackson` untuk frasa khas akun tersebut menghasilkan dua komentar dari `u/Legal-Yoghurt2423`. Satu, dari 29-04-2024 di "Apa alamat surat penggemar Janet Jackson?", menyediakan "townhouse saudara perempuan saya di 4444 Victory Drive, Unit 14 Houston, Texas 77088" bersama dengan kode gerbang. Yang kedua, mulai 30-05-2024, menyediakan alamat kotak surat. Townhouse adalah properti yang dimaksud.

"Unit 14" bukanlah salah satu dari unit kondominium empat digit (1101–1319) di nomor jalan yang sama. Lapisan parsel Harris County memperjelasnya. Mengkueri layanan HCAD ArcGIS `Parcels` untuk `site_str_num='4444'` dan `site_str_name='VICTORY'` menghasilkan 63 catatan; unit 14 adalah akun `1186140010014`, legal `LT 14 BLK 1 HUNTINGTON PARK PAR R/P` — 6.164 kaki persegi, satu lantai seluas 1.456 kaki persegi yang dibangun pada tahun 2005. Persil tersebut berada di sudut barat daya blok townhouse, dan pagar halaman belakangnya membentang di sepanjang batas selatan (E–F, ~20,1 m).

`GEHistoricalImagery` menampilkan 35 tangkapan z20 yang berbeda pada paket antara tahun 2002 dan 2026:

Unduhan GEHistoricalImagery --kiri-bawah=29.86772,-95.47102 \
  --kanan atas=29.86832,-95.47048 -z 20 -d 01/08/2015 \
  -o 01-08-2015.png --dari=PNG

Melapisi cincin parsel HCAD pada setiap pengambilan dan membuat rata-rata profil kecerahan tegak lurus di sepanjang E–F akan menemukan lokasi pagar: pita yang diterangi matahari (sisi selatan pagar) dengan pita bayangan gelap tepat di utaranya. Offset dalam satuan meter, negatif di dalam lot:

| capture | fence face | shadow |
|---|---|---|
| 2011-03-11 | −0.60 | −1.30 |
| 2012-10-28 | −0.60 | −1.35 |
| 2013-08-08 | −1.25 | −1.95 |
| 2014-04-09 | −1.45 | −2.15 |
| 2015-06-01 | −2.15 | −2.85 |
| 2015-08-01 | −2.15 | −2.85 |Pada tahun 2011 pagar tersebut berada di jalur persil. Wajah dan bayangannya kemudian bergerak ke utara bersama-sama dan semakin progresif, mencapai ~2 m di dalam lahan pada tahun 2015, yang menyisakan sebidang tanah terbuka antara pagar dan jalan pribadi. Karena wajah terang dan bayangan gelap bergerak beriringan, inilah pagar itu sendiri yang bergerak, bukan kamera yang bergeser.

Penangkapan setelah tahun 2020 adalah jebakan. Mulai tahun 2024, pengukuran yang sama melonjak ±1–2 m (20-02-2024 terbaca +1,8 m, 27-02-2026 terbaca −1,2 m, 06-10-2026 terbaca +0,9 m). Itu adalah paralaks pagar setinggi 1,5–2 m yang dilihat dari sudut pengambilan berbeda, bukan gerakan kedua. Satu-satunya perubahan monoton di seluruh rangkaian adalah kemunduran 2011→2015, itulah sebabnya tidak ada pasangan tahun berdekatan yang berfungsi: dua tahun dalam kelompok yang dapat digunakan tidak berurutan.

Pagar di halaman belakang dipindahkan antara tahun 2011 dan 2015.