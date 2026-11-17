# The Geometry of Secrets

| | |
|---|---|
| **Bagian** | wave 7 |
| **Kategori** | steganography |
| **Jumlah Solve** | 109 |

## Ikhtisar

Artefaknya adalah OBJ Wavefront 35 baris. Ini mendeklarasikan 24 simpul dan 3 sisi, dengan header `# Marchetti Modeling Suite — export`, `# Geometry, decorative. Provenance uncertain.`, dan `# Model: piece-1814`. Komentar `# reference geometry follows` memisahkan 16 simpul pertama dari 8 simpul terakhir. 8 simpul terakhir adalah sebuah kubus di `±2.0`, dan ketiga permukaan tersebut hanya merujuk pada simpul-simpul tersebut (`17`–`24`):

```
f 17 18 19 20
f 21 22 23 24
f 17 18 22 21
```

16 simpul pertama direferensikan tanpa wajah. Dalam mesh normal, setiap titik digunakan; di sini 16 orang pertama menjadi yatim piatu. Itu adalah petunjuk pertama: geometri adalah pembawa, dan simpul yang tidak digunakan adalah muatannya.

Petunjuk kedua adalah koordinat 16 simpul tersebut. Setiap koordinat mempunyai magnitudo yang seragam: ketiga koordinatnya adalah `±0.5`, atau ketiganya adalah `±1.0`. Jadi setiap titik adalah sudut kubus (tiga bit tanda) ditambah bit skala (kulit dalam `0.5` vs kulit luar `1.0`). Tiga bit ditambah satu bit adalah satu gigitan, dan 16 simpul menghasilkan tepat 16 digit hex — panjang tim nonce. Kubus referensi di `±2.0` adalah shell ketiga yang lebih besar: membuat file tampak seperti model dekoratif, namun bukan bagian dari pesan.

## Penyelesaian

Inventaris file mengonfirmasi pemisahan:

```bash
grep -c '^v ' piece-1814.obj   # 24
grep -c '^f ' piece-1814.obj   # 3
grep -n '^#' piece-1814.obj    # comments, including "reference geometry follows"
```

16 simpul pertama adalah:

```
v -0.5000 -0.5000 -0.5000
v -1.0000 -1.0000  1.0000
v  1.0000 -1.0000 -1.0000
v  0.5000 -0.5000 -0.5000
v -1.0000  1.0000 -1.0000
v  0.5000 -0.5000 -0.5000
v  1.0000 -1.0000 -1.0000
v  1.0000 -1.0000 -1.0000
v  0.5000  0.5000  0.5000
v -1.0000  1.0000  1.0000
v  0.5000 -0.5000 -0.5000
v -1.0000  1.0000  1.0000
v -1.0000  1.0000 -1.0000
v -0.5000 -0.5000 -0.5000
v -1.0000 -1.0000  1.0000
v  0.5000  0.5000  0.5000
```

Dekodenya adalah satu gigitan per titik, dengan bit-bit sesuai urutan koordinat yang muncul: tanda `x`, tanda `y`, tanda `z`, lalu bit skala. Positif adalah `1`, negatif adalah `0`; `1.0` adalah `1`, `0.5` adalah `0`. Gigitannya adalah:

```
nibble = (x > 0) << 3 | (y > 0) << 2 | (z > 0) << 1 | (abs(x) == 1.0)
```

| # | vertex | signs (x,y,z) | scale | nibble |
|---|---|---|---|---|
| 1 | -0.5 -0.5 -0.5 | 000 | 0.5 | 0 |
| 2 | -1.0 -1.0 1.0 | 001 | 1.0 | 3 |
| 3 | 1.0 -1.0 -1.0 | 100 | 1.0 | 9 |
| 4 | 0.5 -0.5 -0.5 | 100 | 0.5 | 8 |
| 5 | -1.0 1.0 -1.0 | 010 | 1.0 | 5 |
| 6 | 0.5 -0.5 -0.5 | 100 | 0.5 | 8 |
| 7 | 1.0 -1.0 -1.0 | 100 | 1.0 | 9 |
| 8 | 1.0 -1.0 -1.0 | 100 | 1.0 | 9 |
| 9 | 0.5 0.5 0.5 | 111 | 0.5 | e |
| 10 | -1.0 1.0 1.0 | 011 | 1.0 | 7 |
| 11 | 0.5 -0.5 -0.5 | 100 | 0.5 | 8 |
| 12 | -1.0 1.0 1.0 | 011 | 1.0 | 7 |
| 13 | -1.0 1.0 -1.0 | 010 | 1.0 | 5 |
| 14 | -0.5 -0.5 -0.5 | 000 | 0.5 | 0 |
| 15 | -1.0 -1.0 1.0 | 001 | 1.0 | 3 |
| 16 | 0.5 0.5 0.5 | 111 | 0.5 | e |

Membaca camilan dalam urutan puncak menghasilkan:

```
0 3 9 8 5 8 9 9 e 7 8 7 5 0 3 e
```

yang merupakan nonce:

```
████████████████
```

Seluruh ekstraksinya kecil:

```python
verts = []
for line in open("piece-1814.obj"):
    if line.startswith("v "):
        verts.append(tuple(map(float, line.split()[1:4])))

nonce = ""
for x, y, z in verts[:16]:
    nibble = ((x > 0) << 3) | ((y > 0) << 2) | ((z > 0) << 1) | int(abs(x) == 1.0)
    nonce += format(nibble, "x")

print(nonce)
```

Keluaran:

```
████████████████
```

Pengiriman adalah formulir JSON standar:

```bash
curl -sS -X POST 'https://pointeroverflowctf.com/challenges/the-geometry-of-secrets/submit' \
  -H 'Content-Type: application/json' \
  -b 'session=<cookie>' \
  --data '{"flag":"████████████████"}'
```

dan jawabannya adalah:

```json
{"correct":true,"message":"Correct."}
```Kubus referensi adalah jalan buntu utama. Delapan simpul dan tiga sisinya mengundang dua pembacaan yang salah. Pertama, memperlakukan 16 simpul pertama sebagai bitstream mentah — 48 bit tanda diikuti oleh 16 bit skala — menghasilkan `064524ee340f6b5a` (atau `6b5a064524ee340f` dengan aliran terbalik). Ini adalah 16 karakter hex, jadi kelihatannya masuk akal, tapi ditolak. Kedua, menggunakan urutan titik kubus referensi sebagai buku kode untuk delapan sudut menghasilkan `c5febeff03e3bc50` (indeks sudut ditambah skala sebagai bit rendah) atau `6af7d7ff0979d6a0` (skala sebagai bit tinggi); keduanya ditolak. Kubus referensi persis seperti yang dikatakan dalam komentarnya: geometri referensi, bukan kunci. Pesannya ada di simpul yatim piatu, dan pengkodeannya adalah per-simpul, bukan aliran per-sumbu.

Umpan lainnya adalah komentar dan mesh. `strings` di OBJ hanya mengembalikan header Marchetti dan baris `reference geometry follows`; tidak ada fragmen teks. Ketiga sisi tersebut hanya membentuk sebagian kubus dan tidak mengkodekan apa pun. Kubus `±2.0` secara visual merupakan cangkang ketiga, tetapi muatannya adalah dua cangkang bagian dalam di `±0.5` dan `±1.0`; memasukkan simpul referensi dalam dekode akan menggeser string dan gagal.

Konstruksinya konsisten: tiga bit tanda memilih salah satu dari delapan sudut kubus, bit skala memilih cangkang, dan nilai empat bit yang dihasilkan adalah digit hex. Enam belas simpul mengisi enam belas digit hex, cocok dengan enam belas karakter nonce. Ungkapan "presisi di setiap dimensi" adalah petunjuk untuk bit keempat: besarnya adalah dimensi ekstra yang mengubah sudut 3D menjadi gigitan.

Geometri diterjemahkan menjadi `████████████████`, dan penandanya adalah `POCTF{████…}`.