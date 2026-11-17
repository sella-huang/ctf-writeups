# A Map of Small Things

| | |
|---|---|
| **Bagian** | wave 4 |
| **Kategori** | crypto |
| **Jumlah Solve** | 320 |

## Ringkasanan

Layanan ini menggunakan varian Diffie-Hellman kurva elips khusus yang disebut MapCrypt. Kurvanya adalah `y² = x³ + x (mod p)` dengan `p = 17759930830448937043`, dan titik dasarnya adalah `G = (2, 2130375071143573863)`. Tugasnya adalah memulihkan kunci privat `k` dari kunci publik `Q = kG`. Parameternya lemah: kurvanya memiliki `j = 1728`, `p ≡ 3 (mod 4)`, dan urutan grupnya mulus.

## Penyelesaian

Karena `p ≡ 3 (mod 4)` dan kurvanya adalah `y² = x³ + ax` dengan `a = 1`, `b = 0`, maka orde grupnya adalah

```text
#E(Fp) = p + 1 = 17759930830448937044
       = 2² · 38603 · 121123 · 949584469
```

Titik dasar memiliki orde penuh `p + 1`. Misalnya, kunci publiknya adalah

```text
Q = (12583775432034319774, 16756047493174394364)
```

Gunakan Pohlig-Hellman. Untuk setiap pangkat prima `q^e`, hitunglah

```text
P  = (N / q^e) · G
Q' = (N / q^e) · Q
```

dan selesaikan `Q' = xP` di subgrup berorde `q^e`. Tiga faktor ganjil tersebut cukup kecil untuk baby-step giant-step; subgrup terbesar memiliki orde `949584469`, sehingga tabel BSGS memiliki sekitar entri `sqrt(949584469) ≈ 30815`. Residunya adalah:

```text
k mod 4         = 3
k mod 38603     = 13177
k mod 121123    = 33957
k mod 949584469 = 831978970
```

CRT memberi

```text
k = 10255475597827708535
```

dan pemeriksaan lokal `k · G == Q` berhasil. Tantangannya menerima bilangan bulat desimal mentah, jadi mengirimkan `10255475597827708535` akan mengembalikan flag standar:

```text
████████████████
```

Kunci pribadinya adalah k = 10255475597827708535.