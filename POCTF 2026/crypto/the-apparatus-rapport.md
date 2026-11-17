# The Apparatus, Rapport

| | |
|---|---|
| **Bagian** | wave 2 |
| **Kategori** | crypto |
| **Jumlah Solve** | 487 |

## Ringkasanan

Tantangan tersebut memberikan dua artefak: transkrip dari peralatan tic-tac-toe mekanis, dan satu blok teks tersandi. Transkrip tersebut mencantumkan pergerakan peralatan berdasarkan koordinat papan, bukan berdasarkan nomor giliran, jadi masalah pertama adalah mengurutkannya. Masalah kedua adalah alfabet: ciphertext berisi tanda baca dan kurung kurawal, sehingga tidak bisa menjadi Vigenère di atas A–Z.

## Penyelesaian

Papan peralatan berbentuk kotak 3×3 berlabel A – I. Mengurutkan tabel perpindahan berdasarkan indeks gilirannya menghasilkan urutan sel:

A G E B I

Urutan itu adalah kuncinya. Dengan A = 0 … Z = 25 dipetakan ke pergeseran 0, 6, 4, 1, 8.

Tabel Vigenère standar hanya mencakup 26 huruf, tetapi byte teks tersandi mencakup seluruh rentang ASCII yang dapat dicetak (32–126, 95 simbol). Sandinya adalah skema aditif yang sama pada alfabet itu:

p_i = 32 + ((c_i - 32) - (k_i - 65)) mod 95

dengan `k_i` huruf kunci sebagai kode ASCII-nya, berputar setiap lima karakter. Pengurangannya adalah modulo 95, bukan modulo 26.

Lima karakter ciphertext pertama adalah `PUGUN`. Menerapkan shift:

P - 0 = P
kamu - 6 = HAI
G - 4 = C
kamu - 1 = T
N - 8 = F

Oleh karena itu, teks biasa membuka `POCTF{`, yang merupakan awalan yang diharapkan dan mengonfirmasi kunci dan fasenya: kunci dimulai pada karakter teks sandi pertama, bukan setelah awalan.

Satu jalan buntu layak untuk dicatat. Karena kuncinya menggunakan lima huruf berbeda, mengurutkannya berdasarkan abjad menjadi `ABEGI` terlihat tidak berbahaya, tetapi sandinya bergantung pada posisinya. Dengan `ABEGI`, karakter kedua didekripsi sebagai `U - 1 = T`, menghasilkan `POTTF{`, yang bukan merupakan flag yang valid. Urutan yang diambil dari transkrip harus dipertahankan.

Blok cipherteks kedua, yang didekripsi dengan kunci dan fase yang sama, menghasilkan teks rasa:

> Melalui garam dan abu, melalui nafas yang ditarik namun tidak kembali, kita membuka jalan.

Dekripsi penuh adalah satu baris setelah alfabet diperbaiki:

```python
p = "".join(chr(32 + ((ord(c) - 32) - (ord(k[i % 5]) - 65)) % 95)
            for i, c in enumerate(ct))
```

Flag-nya adalah `POCTF{████…}`.