# No Structure Required

| | |
|---|---|
| **Bagian** | wave 3 |
| **Kategori** | web |
| **Jumlah Solve** | 340 |

## Ikhtisar

SecureNotes v1.0 adalah aplikasi penyimpanan dokumen kecil yang didukung oleh “Whitmore Storage Engine.” Titik akhir loginnya menerima JSON dan meneruskan nilai yang diberikan langsung ke objek kueri. Hal ini memungkinkan injeksi operator untuk melewati autentikasi, dan titik akhir penelusuran kemudian menampilkan dokumen dari koleksi yang tidak boleh dijangkau oleh pengguna yang masuk.

## Penyelesaian

Formulir login memposting JSON ke `login` dengan `username` dan `password`. Komentar sumber mengonfirmasi bahwa titik akhir menerima `application/json` dan kedua kunci diteruskan ke kueri apa adanya. Menyediakan operator kueri alih-alih string melewati pemeriksaan kredensial:

```json
{"username": {"$ne": null}, "password": {"$ne": null}}
```

Responsnya menetapkan cookie sesi dan melaporkan keberhasilan masuk sebagai pengguna biasa:

```json
{"message": "Signed in as j.abernathy.", "next": ".../dashboard", "ok": true}
```

Dasbor ditautkan ke `browse?col=notes`, tempat parameter `col` memilih koleksi dan dokumen mentah dirender. Catatan halaman tersebut menyatakan bahwa koleksi lain mungkin ada tergantung pada peran, namun pengendali penelusuran tidak menerapkan batasan kepemilikan atau peran pada koleksi yang diminta.

Menghitung nama koleksi melalui `browse?col=<name>` mengekspos data pendukung. Koleksi `users` mengungkapkan nama pengguna, kata sandi, dan peran, termasuk akun administrator. Koleksi `secrets` mengembalikan satu dokumen berlabel flag tim, dengan penanda di bidang `flag`. Mengakses koleksi itu tidak memerlukan sesi administrator; bypass otentikasi ditambah parameter pengumpulan tidak terbatas sudah cukup.

Flag yang dikirimkan adalah POCTF{78.1814.████████████████.████████████████} dan diterima.