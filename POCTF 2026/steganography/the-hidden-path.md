# The Hidden Path

| | |
|---|---|
| **Bagian** | wave 5 |
| **Kategori** | steganography |
| **Jumlah Solve** | 160 |

## Ikhtisar

Artefak tersebut adalah spesimen tipe satu halaman, `specimen.pdf`, dengan enam belas garis mesin terbang. Setiap mesin terbang berbentuk seperti rumah: persegi panjang dengan atap. Dinding vertikal kanan berakhir di atas atau di bawah puncak atap. Kedua keadaan tersebut adalah satu-satunya perbedaan antara mesin terbang, dan masing-masing mengkodekan satu bit. Enam belas bit menghasilkan nonce heksadesimal empat digit.

## Penyelesaian

Ekstrak jalur mesin terbang dari aliran konten PDF. Setiap jalur memiliki bentuknya

```text
n x0 y0 m x1 y1 l x2 y2 l x3 y3 l x4 y4 l h
```

Titik `l` kedua adalah puncak tembok kanan, dan titik `l` ketiga adalah puncak atap. Dalam PDF, koordinat titik sumbu y mengarah ke atas, jadi dinding kanan atas di atas puncak adalah 0 dan satu di bawahnya adalah 1. Bacalah mesin terbang dalam urutan baris-utama.

```python
import re, zlib, base64

raw = open('specimen.pdf', 'rb').read()
stream = re.search(rb'stream\r?\n(.*?)endstream', raw, re.S).group(1).strip()
if stream.endswith(b'~>'):
    stream = stream[:-2]
content = zlib.decompress(base64.a85decode(stream)).decode('latin1')

paths = re.findall(
    r'n ([\d.]+) ([\d.]+) m ([\d.]+) ([\d.]+) l '
    r'([\d.]+) ([\d.]+) l ([\d.]+) ([\d.]+) l '
    r'([\d.]+) ([\d.]+) l h',
    content,
)

bits = ''.join('0' if float(p[5]) > float(p[7]) else '1' for p in paths)
print(bits, f'{int(bits, 2):04X}')
```

Enam belas perbandingan menghasilkan bit string

```text
████████████████
```

Halaman ini menerima nonce mentah, jadi tidak diperlukan pembungkus `POCTF{...}`:

```text
████████████████
```

Spesimen mengkodekan nonce mentah ████████████████.