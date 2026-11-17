# The Cipher in the Stars

| | |
|---|---|
| **Bagian** | wave 8 |
| **Kategori** | crypto |
| **Jumlah Solve** | 73 |

## Ringkasan

Transkrip tersebut berisi 1.400 catatan foton, empat pasangan negara bagian yang diumumkan, 128 baris pemeriksaan paritas, dan sindrom 128-bit. Setiap pasangan berisi satu keadaan bujursangkar (`+`) dan satu keadaan diagonal (`x`). Pengirim mengumumkan pasangan tersebut; penerima mencatat dasar dan hasil.

Sebuah catatan bersifat konklusif hanya jika hasil yang diperoleh penerima bersifat ortogonal terhadap keadaan yang diumumkan di dasar penerima. Dalam hal ini pengirim harus mengirimkan negara lain yang diumumkan, jadi bitnya adalah bit yang diayak. Semua catatan lainnya akan dibuang.

## Penyelesaian

Terapkan aturan diskriminasi negara yang jelas pada 1.400 catatan. Ada 347 bit konklusif. 256 yang pertama adalah kunci mentah yang diayak yang digunakan oleh lapisan rekonsiliasi.

Untuk masing-masing dari 128 baris pemeriksaan paritas yang diterbitkan, hitung paritas bit-bit yang sesuai dalam 256 bit pertama yang diayak. Bandingkan paritas ini dengan sindrom yang dipublikasikan; yang membedakannya adalah sindrom kesalahan transmisinya. Siapkan 128 persamaan linier melalui GF(2) dan selesaikan vektor kesalahan berbobot rendah dengan penghancur SAT. Solusi sparse membalik posisi yang diindeks nol berikut ini:

```text
1, 18, 54, 91, 95, 106, 137, 164, 173, 211
```

Balikkan sepuluh bit tersebut, lalu kemas 256 bit bit paling signifikan yang telah dikoreksi terlebih dahulu menjadi 32 byte. String hex yang dihasilkan adalah kunci bersama yang diinginkan:

```text
████████████████
```

Jawaban yang diterima adalah kunci yang menyampaikan `████████████████`.