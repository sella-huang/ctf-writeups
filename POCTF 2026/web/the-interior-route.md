# The Interior Route

| | |
|---|---|
| **Bagian** | wave 6 |
| **Kategori** | web |
| **Jumlah Solve** | 148 |

## Ikhtisar

Bittner & Cross menjalankan alat pratinjau tautan: `POST /preview` dengan `{"url": "..."}` membuat server mengambil URL itu dan mengembalikan isi URL tersebut. Layanan ini mengklaim hanya dapat menjangkau host eksternal dan tidak dapat melihat jaringan pribadi apa pun. Klaim tersebut adalah keseluruhan tantangannya — titik akhir pratinjau adalah pemalsuan permintaan sisi server yang primitif, dan tandanya berada pada layanan metadata instans yang seharusnya dicakup oleh filter.

## Penyelesaian

Permintaan dasar untuk `https://example.com` mengembalikan halaman terekam (“Pratinjau tautan tidak tersedia untuk URL eksternal dalam mode demo”), yang hanya merupakan perilaku demo. Meminta `http://127.0.0.1/` mengembalikan halaman kalengan yang sama: alamat loopback dikenali dan dihubung pendek. Filternya adalah daftar host terlarang, bukan pemeriksaan yang dilakukan pada alamat yang diselesaikan.

Ruang tautan-lokal tidak ada dalam daftar itu. Permintaan `http://169.254.169.254/latest/meta-data/` menjangkau layanan metadata alih-alih stub demo, dan layanan menjawab `404 NOT FOUND` — respons HTTP nyata, membuktikan permintaan tersebut meninggalkan pekerja pratinjau dan mencapai titik akhir metadata.

Cabang IAM hadir. `/latest/meta-data/iam/security-credentials/` mengembalikan satu nama peran, `poctf-service-role`, dan mengambil `/latest/meta-data/iam/security-credentials/poctf-service-role` mengembalikan dokumen kredensial lengkap dengan `Code`, `LastUpdated`, `Type`, `AccessKeyId`, `SecretAccessKey`, `Token` dan `Expiration`.

`AccessKeyId` bukanlah kunci sama sekali: ini adalah penanda bendera, ditanam di tempat yang hanya dapat dibagikan oleh layanan metadata. Kredensial di sekitarnya jelas merupakan placeholder palsu, jadi tidak diperlukan apa pun lagi — tidak ada panggilan S3, tidak ada penandatanganan — yang diperlukan.

Flag-nya adalah ████████████████.