# The Apparatus, Dissolution

| | |
|---|---|
| **Bagian** | wave 7 |
| **Kategori** | reverse engineering |
| **Jumlah Solve** | 122 |

## Ikhtisar

`apparatus_dissolution.c` menyematkan output terlarut di `DATA[]` dan mengekspos cipher maju. Masing-masing dari empat lintasan menggantikan setiap byte melalui S-box 256 byte (`TBL`), lalu memutar buffer yang tersisa sebanyak satu byte.

## Penyelesaian

Balikkan lintasan dalam urutan terbalik. Untuk setiap lintasan, putar buffer ke kanan sebanyak satu byte, lalu terapkan S-box terbalik (`inv[TBL[i]] = i`). Setelah empat iterasi, `DATA[]` mendekripsi ke bendera.

Flag yang dikirimkan adalah POCTF{116.1814.████████████████.████████████████} dan diterima.