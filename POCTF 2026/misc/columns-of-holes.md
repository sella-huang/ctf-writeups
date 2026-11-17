# Columns of Holes

| | |
|---|---|
| **Bagian** | wave 3 |
| **Kategori** | misc |
| **Jumlah Solve** | 261 |

## Ikhtisar

Selebaran tersebut adalah memorandum operasi batch dari "Layanan Pemrosesan Data Meridian" dengan dua tiket pekerjaan 80 kolom terlampir, akhiran A dan B. Memo tersebut menyatakan bahwa nonce 16 karakter disisipkan melalui wilayah muatan setiap tiket dengan langkah yang diberikan oleh penanda kartu `PAT`, dan bahwa fragmen tersebut harus digabungkan dalam urutan abjad sufiks (A, lalu B). Baris tercetak yang dapat dibaca manusia di bagian atas setiap kartu digambarkan sebagai rusak, sehingga informasinya harus diambil kembali dari kolom yang dilubangi itu sendiri.

## Penyelesaian

Kedua kartu tersebut disematkan sebagai gambar 1600×640 di halaman 2 PDF (xrefs 6 dan 8). Mengambang batas gambar dan mengekstraksi komponen yang terhubung akan mengisolasi lubang berlubang (kira-kira 27×12 px elips) dari mesin terbang yang dicetak (kira-kira 8×5 px). Lubang-lubang tersebut terletak pada kisi-kisi biasa:

- 12 baris lubang pada y ≈ 111, 153.5, 196.5, 239, 281, 323.5, 366.5, 409, 451, 493.5, 536.5, 579 — ini adalah baris standar `12, 11, 0, 1, …, 9`;
- 80 kolom lubang dimulai dari x ≈ 59,5 dengan pitch ≈ 19 px.

Membaca kumpulan pukulan dalam kode Hollerith (pukulan tunggal `0`–`9` = digit; zona `12` + digit = `A`–`I`; zona `11` + digit = `J`–`R`; zona `0` + digit = `S`–`Z`; `8,3` = `=`, `0,1` = `/`) memulihkan kedua tumpukan kartu:

```
CARD A: JOB=1814A   PID1819801/05/67OP?TQW59PAT=54  <36-char payload>
CARD B: JOB=1814B   PID2986325/07/67OP?KAR32PAT=23  <36-char payload>
```

Wilayah muatan dimulai pada kolom 44; nilai `PAT` adalah pasangan dua digit `(offset, stride)` yang digunakan untuk mengambil sampelnya. Untuk kartu A pasangan `(5, 4)` memilih delapan posisi, dan untuk kartu B `(2, 3)` memilih delapan posisi; ini adalah pilihan yang membuat setiap karakter yang diekstraksi menjadi digit heksadesimal. Oleh karena itu, setiap kartu menyumbangkan fragmen delapan karakter.

Menggabungkan fragmen dalam urutan sufiks — A terlebih dahulu, lalu B — menghasilkan nonce 16 hex.

Nonce 16-hex adalah `████████████████`.