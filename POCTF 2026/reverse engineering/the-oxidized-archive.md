# The Oxidized Archive

| | |
|---|---|
| **Bagian** | wave 6 |
| **Kategori** | reverse engineering |
| **Jumlah Solve** | 161 |

## Ikhtisar

Handoutnya adalah Linux ELF x86-64 (`oxidized_archive`) yang dilucuti ditambah README yang menjanjikan "instruksi sumber dan pembuatan" tetapi tidak mengirimkan keduanya. Prompt mengatakan utilitas tidak membaca masukan, memulihkan arsip dan mencetaknya, namun tidak mencetak apa pun, karena arsip dan kuncinya dimasukkan ke dalam biner dan kripto pembuatnya bukanlah kripto.

Menjalankannya akan mengonfirmasi deskripsi: tidak ada keluaran, status keluar 1. Analisis statis biner Rust sudah cukup untuk memulihkan kedua konstanta dan mengimplementasikan kembali sandi mainan dalam selusin baris Python.

## Penyelesaian

Karena biner dihilangkan, batasan fungsi berasal dari daftar FDE `.eh_frame` dan bukan dari tabel simbol. Kode pengguna berada di awal `.text`: satu FDE mencakup `0x15791`-`0x16261`, dan rentang tersebut adalah `main` program.

Membacanya dari atas ke bawah:

1. `main` memanggil `std::env::args()` dan bercabang pada jumlah argumen. Dengan tepat satu argumen tambahan, ia menghitung `FNV-1a-64` dari `argv[1]` (`offset basis 0xcbf29ce484222325`, `prime 0x100000001b3`) dan membandingkan intisari dengan nilai yang diwujudkan melalui slot GOT. Ketidakcocokan mencetak `archive sealed. supplied token does not match.` dan keluar. Arsipnya tidak pernah disentuh, itulah sebabnya program tidak bersuara saat dijalankan secara normal.
2. Dua blok konstan kemudian disalin dari `.rodata` dengan `movups`: 128 byte "arsip" di `0x5480` dan kunci 24-byte yang diakhiri NUL di `0x5500`.
3. Kunci dimasukkan melalui rutinitas FNV-1a-64 yang sama untuk menghasilkan benih 64-bit.
4. Seed tersebut menggerakkan LCG dengan konstanta PCG32:

```text
state = seed
state = state * 0x5851f42d4c957f2d + 0x14057b7ef767814f   (mod 2^64)
keystream_byte = (state >> 33) & 0xff
```

   Kompiler telah memvektorisasi generator, kemudian melakukan XOR 128 byte arsip terhadap 128 byte keystream dengan SSE (`xorps` pada jalur 16-byte) ditambah ekor skalar.
5. Hasilnya dipindai untuk `0x00` pertama (pencarian panjang string `str::from_utf8`/C di `0x16120`), dan potongan itu diserahkan ke `miniz_oxide`.

Langkah terakhir adalah umpan: mendekripsi 128 byte dengan Python dengan seed yang diimplementasikan ulang dan LCG segera menghasilkan ASCII yang dapat dicetak dimulai dengan `POCTF{`, jadi tidak diperlukan dekompresi dan buffer sudah menjadi teks biasa yang seharusnya dicetak oleh utilitas.

Teks biasa dan flag-nya adalah ████████████████.