# A Peculiar Game

| | |
|---|---|
| **Bagian** | wave 4 |
| **Kategori** | misc |
| **Jumlah Solve** | 215 |

## Ikhtisar

Tantangan ini membagikan PDF per tim dengan tampilan kolom catur tahun 1887, *The Vienna Correspondence Monthly*. Ini melaporkan permainan korespondensi singkat antara "Signor Marchetti" dan "Dr. Whitmore", diberi penjelasan langkah demi langkah, menyimpulkan 1/2-1/2. Prompt tersebut menegaskan bahwa ada sesuatu tentang gerakan tersebut yang sangat aneh dan bahwa permainan tersebut "tidak sepenuhnya tentang catur". Tujuannya adalah nonce 16-hex.

## Penyelesaian

Permainan yang ditranskrip dari PDF dalam urutan PGN adalah:

```
1. b4   Nxb3    2. Nxd3  h4      3. Nxe3  Nxg3    4. Nxg3  Nxf3
5. Nxc3 Nxb3    6. c4    Nxa3    7. Nxg3  Nxg3    8. f4    f4
```

Dua properti langsung menonjol.

Pertama, permainan catur bukanlah catur yang sah. Hitam tidak bisa memainkan `1...Nxb3` pada langkah pertama, kedua belah pihak "menangkap" pada `g3`, dan keduanya memainkan `f4` pada langkah terakhir. Notasi tersebut digunakan sebagai pembawa, bukan sebagai permainan.

Kedua, dan yang menentukan, setiap kotak tujuan terletak pada peringkat 3 atau peringkat 4. Enam belas setengah gerakan hanya mengunjungi pita kotak `a3`–`h4`, yang berisi tepat enam belas kotak. Enam belas kotak untuk nonce 16 hex berarti setiap kotak tujuan adalah digit hex.

Pengindeksan alami dari pita 2x8 tersebut adalah berdasarkan huruf file dan kemudian berdasarkan peringkat, yaitu urutan abjad dari nama persegi:

```
a3=0  a4=1  b3=2  b4=3  c3=4  c4=5  d3=6  d4=7
e3=8  e4=9  f3=a  f4=b  g3=c  g4=d  h3=e  h4=f
```

sehingga perpindahan seperti `b4` dipetakan ke `1*2 + (4-3) = 3`, sedangkan `f4` dipetakan ke `5*2 + (4-3) = b`.

Menerapkan tabel ini, dalam urutan permainan, ke enam belas kotak tujuan

```
b4  b3  d3  h4  e3  g3  g3  f3  c3  b3  c4  a3  g3  g3  f4  f4
```

menghasilkan enam belas camilan nonce. Mengirimkan nonce mentah ke titik akhir tantangan diterima ("Permainan selesai."), yang mengonfirmasi pembacaan: para pemain tidak bermain catur sama sekali, mereka mengeja string heksadesimal pada enam belas kotak yang pernah disentuh permainan.

Nonce 16-hex adalah `████████████████`.