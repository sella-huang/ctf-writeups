# Noise Below the Signal

| | |
|---|---|
| **Bagian** | wave 6 |
| **Kategori** | crypto |
| **Jumlah Solve** | 195 |

## Ringkasanan

Nexus Communications mengirimkan pertukaran kunci LWE dengan dua cacat: batasan kesalahan terlalu kecil (`B = 3`) dan rahasianya diambil dari distribusi sempit yang sama (`s ∈ {−3,…,3}ⁿ`). Selebaran ini memberikan matriks publik `A` (30 × 15), vektor `b = A·s + e mod q`, dan parameter `n = 15`, `q = 1601`, `m = 30`, `B = 3`.

Dengan istilah kebisingan yang kecil dan rahasia kecil, masalahnya bukanlah masalah pembelajaran sama sekali: ini adalah masalah vektor terdekat yang targetnya lebih dekat daripada jarak kisi, sehingga reduksi kisi tunggal akan memulihkan `s` dengan tepat.

## Penyelesaian

Relasi `b = A·s + e (mod q)` berarti bahwa `b` terletak di dalam `√(m)·B ≈ 16` dari titik kisi `A·s + q·k` untuk vektor bilangan bulat yang sesuai `k`. Kisi

```text
L = { A·s + q·k : s ∈ Z^n, k ∈ Z^m } ⊂ Z^m
```

memiliki determinan `q^(m−n) = 1601^15`, sehingga vektor terpendek Gaussian-heuristiknya bernorma di sekitar `√(m / (2πe)) · q^((m−n)/m) ≈ 53`. Perbedaan norma 16 sudah di bawah itu, itulah sebabnya parameternya rusak.

Untuk mengubah CVP menjadi SVP, sematkan target dengan penyematan Kannan dalam dimensi `m + n + 1 = 46`:

```text
[ q·I_m      0      0 ]   (m rows)
[ A           I_n    0 ]   (n rows, one per column of A)
[ b           0      1 ]   (target row)
```

Vektor kisi yang dibangun dari basis ini dengan koordinat terakhir `t = ±1` adalah `(A·s + q·k + t·b, s, t)`, yang mana untuk `t = 1` sama dengan `(−e, s, 1)` karena `b ≡ −A·s + e`. Normanya paling banyak adalah `√(m·B² + n·B² + 1) ≈ 21`, jauh di bawah ≈ 120 LLL yang dapat dipaksa untuk diterima: vektor pendek yang ditanam adalah urutan besarnya lebih pendek daripada basis tereduksi lainnya.

Jadi basis 46 baris dikurangi LLL pada bilangan bulat (baris bilangan bulat, titik mengambang Gram – Schmidt dipertahankan melalui dekomposisi QR, pengurangan ukuran ditambah kondisi Lovász), dan baris tereduksi dipindai untuk baris yang koordinat terakhirnya adalah `±1`. Baris tersebut membawa vektor kesalahan di posisi `m` pertamanya dan kandidat rahasia (mungkin terbalik) di `n` berikutnya.

Tepatnya satu baris tereduksi sependek itu, dan keluar dengan `t = 1`, jadi rahasianya adalah negasi dari blok tengahnya. Memeriksa kandidat terhadap seluruh 30 sampel menghasilkan residu terpusat di `[−3, 3]`, mengonfirmasi pemulihan. Mengirimkan array 15 bilangan bulat yang dihasilkan akan mengembalikan bendera.

Flag yang dipulihkan adalah ████████████████.