# Velocity

| | |
|---|---|
| **Bagian** | wave 4 |
| **Kategori** | steganography |
| **Jumlah Solve** | 257 |

## Ikhtisar

Artefaknya adalah file MIDI Standar format-0 dengan satu track, 480 tick per not seperempat, dan 64 event note. Catatan tersebut membentuk sketsa étude C-mayor; nadanya adalah sampulnya. Payloadnya ada dalam kecepatan note-on: setiap event menggunakan `0x60` (96) atau `0x30` (48), jadi setiap note membawa satu bit.

## Penyelesaian

`file etude-in-c-1814.mid` melaporkan `Standard MIDI data (format 0) using 1 track at 1/480`. Lagu ini berisi peristiwa meta untuk tempo, tanda birama, tanda kunci (C mayor), dan nama lagu, diikuti oleh 64 peristiwa catatan `90 <note> <velocity>`. Peristiwa yang dicatat adalah `80 <note> <velocity>` dan tidak relevan.

Kecepatan hanya mempunyai dua nilai:

```text
0x60 = 96  -> 1
0x30 = 48  -> 0
```

Baca peristiwa catatan dalam urutan file dan kelompokkan 64 bit yang dihasilkan menjadi 16 camilan, bit paling signifikan terlebih dahulu. Empat nada pertama mempunyai kecepatan `0x60, 0x60, 0x60, 0x30`, sehingga menghasilkan `1110` = `e`:

```text
1110 1111 1010 1111 0110 0011 0010 0000
0110 0011 1000 0001 0011 1000 1111 1110
```

Ekstraksi minimal:

```python
from pathlib import Path

b = Path("etude-in-c-1814.mid").read_bytes()
vel = [b[i + 2] for i in range(len(b) - 2) if b[i] == 0x90]
bits = "".join("1" if v == 0x60 else "0" for v in vel)
assert len(bits) == 64

nonce = "".join(f"{int(bits[i:i + 4], 2):x}" for i in range(0, 64, 4))
print(bits)
print(nonce)
```

Keluaran:

```text
1110111110101111011000110010000001100011100000010011100011111110
████████████████
```

Pemindaian byte aman untuk file ini karena trek menggunakan byte status catatan eksplisit dan tidak ada `0x90` yang muncul sebagai byte data; parser umum masih harus melacak status pengoperasian MIDI.

Nonce yang dipulihkan adalah `████████████████`.