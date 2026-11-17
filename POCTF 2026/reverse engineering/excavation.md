# Excavation

| | |
|---|---|
| **Bagian** | wave 1 |
| **Kategori** | reverse engineering |
| **Jumlah Solve** | 459 |

## Ikhtisar

Empat file simpanan dari RPG fiksi "Makam Kematian" disediakan. Tantangannya meminta token 16 karakter, bukan flag biasa.

## Penyelesaian

Setiap file dimulai dengan header 12-byte: `9e e1 c7 21 02 00 02 00` ajaib 8-byte diikuti dengan panjang little-endian 4-byte. Payload di-XOR dengan kunci berulang 8-byte.

Teks biasa adalah data rekaman ASCII, sehingga setiap byte kunci dipulihkan secara independen dengan memberi skor pada kolom yang didekripsi terhadap distribusi byte nama bidang dan deskripsi yang dapat dicetak. Kunci yang dihasilkan mendekripsi penyimpanan ke dalam catatan item, masing-masing dengan nama dan deskripsi yang diawali panjang.

Huruf pertama nama benda membentuk akrostik. Tiga referensi menyimpan mantra `TORCH`, `SILVERMOON`, dan `NIGHTENDSOON`, mengonfirmasi skema; penyimpanan tim memiliki enam belas nama item yang inisialnya mengeja token.

Flag yang dikirimkan adalah `████████████████` dan diterima.