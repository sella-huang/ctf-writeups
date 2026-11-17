# The Wrong Channel

| | |
|---|---|
| **Bagian** | wave 6 |
| **Kategori** | steganography |
| **Jumlah Solve** | 161 |

## Ikhtisar

Artefaknya adalah PNG RGBA 8-bit 128×96. Saluran merah, hijau, dan biru memiliki bit orde rendah dengan entropi tinggi, sehingga pemindaian RGB LSB standar menghasilkan kandidat yang berisik dan positif palsu. Muatan sebenarnya tidak ada dalam bit tersebut dan tidak dalam potongan PNG tambahan: file hanya berisi `IHDR`, satu `IDAT`, dan `IEND`. Payloadnya ada di saluran alfa.

## Penyelesaian

Jalan potongan tidak menunjukkan metadata `tEXt`, `zTXt`, atau `iTXt`, dan data gambar yang didekompresi persis dengan ukuran yang diharapkan: byte filter `128 * 96 * 4 + 96`. LSB RGB adalah umpannya.

Saluran alfa konstan pada 255 kecuali delapan piksel di awal gambar:

```text
(0,0) a=239   (0,1) a=222   (0,2) a=125   (0,3) a=118
(0,4) a=4     (0,5) a=204   (0,6) a=77    (0,7) a=112
```

Baca dalam urutan raster, nilai alfa tersebut adalah byte:

```text
ef de 7d 76 04 cc 4d 70
```

Ini adalah nonce 8-byte, atau 16 karakter hex:

```text
████████████████
```

Nilainya adalah byte alfa itu sendiri, bukan bit paling tidak signifikan, itulah sebabnya pemindaian LSB khusus RGB tidak dapat menangkap nilai tersebut.

Pemeriksaan minimal:

```python
from PIL import Image

im = Image.open("channel-1814.png").convert("RGBA")
w, h = im.size
px = im.load()
raw = bytes(
    px[x, y][3]
    for y in range(h)
    for x in range(w)
    if px[x, y][3] != 255
)
print(raw.hex())
```

Keluaran:

```text
████████████████
```