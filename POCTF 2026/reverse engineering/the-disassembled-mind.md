# The Disassembled Mind

| | |
|---|---|
| **Bagian** | wave 2 |
| **Kategori** | reverse engineering |
| **Jumlah Solve** | 405 |

## Ikhtisar

Spesimennya adalah Python 3.13 `.pyc`. Fungsi `check()` berisi lima konstanta 59-byte sebaris; dua adalah umpan dan tiga XOR bersama-sama menjadi bendera.

## Penyelesaian

Setelah header 16-byte, `marshal` memuat objek kode modul. Objek kode `check()` yang disarangkan memiliki lima konstanta `bytes` 59-byte di `co_consts`. Menguji sepuluh kombinasi 3 elemen dan melakukan XOR pada setiap byte, kombinasi `(0, 2, 3)` menghasilkan string 59 byte yang dapat dicetak yang dimulai dengan `POCTF{`; kombinasi lainnya tidak. Konstanta 1 dan 4 adalah umpannya.

Flag yang dikirimkan adalah POCTF{117.1814.████████████████.████████████████} dan diterima.