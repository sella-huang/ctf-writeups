# Panoptican

| | |
|---|---|
| **Bagian** | wave 3 |
| **Kategori** | reverse engineering |
| **Jumlah Solve** | 333 |

## Ikhtisar

Biner `panoptican` yang dilucuti membaca gambar PGM, membandingkannya dengan identitas referensi yang telah didaftarkan sebelumnya, dan mencetak kode otorisasi ketika gambar tersebut cocok.

## Penyelesaian

Jalur kecocokan menghitung jumlah perbedaan kuadrat antara masukan dan gambar referensi 128-byte, menerima skor paling banyak `0x7d0` (2000). Jika berhasil, kode di `0x137c` meng-XOR gambar referensi dengan kunci 128-byte dan mencetak hasilnya sebagai `AUTH: %s`.

Kedua array berada di `.data`: kunci di alamat virtual `0x4010` (file offset `0x3010`) dan referensi di `0x4090` (file offset `0x3090`), masing-masing 128 byte. Meng-XOR-kan dua array yang diekstraksi byte demi byte akan menghasilkan tanda penuh. Bitmap 16×8 yang tertanam adalah referensi terdaftar; kode otorisasi adalah flag-nya.

Flag yang dikirimkan adalah POCTF{120.1814.████████████████.████████████████} dan diterima.