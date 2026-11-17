# Against the Clock

| | |
|---|---|
| **Bagian** | wave 8 |
| **Kategori** | reverse engineering |
| **Jumlah Solve** | 61 |

## Ikhtisar

`clock` adalah ELF non-PIE x86-64 yang dilucuti. Ia memanggil `time()` dua kali, melakukan loop sibuk singkat, dan kemudian mendapatkan kunci dekripsi dari stempel waktu. Teks sandinya adalah gumpalan 59 byte pada file offset `0x2020`, dan aliran kuncinya adalah SHA-256 dalam mode penghitung:

```text
SHA256(seed_le64 || 0x542947a8_le64 || counter_le32)
```

Delapan byte pertama dari setiap blok di-XOR dengan ciphertext. Seed adalah waktu Unix saat ini, kecuali loop sibuk memakan waktu setidaknya dua detik, dalam hal ini seed di-XOR dengan `0x5a17eddecade5a17`.

## Penyelesaian

Konstanta `0x542947a8` merupakan pemisah domain dan epoch target. Dalam desimal adalah `1411983272`, yaitu 29-09-2014 11:51:04 UTC. Oleh karena itu, cara yang dimaksudkan untuk menjalankan biner adalah:

```bash
faketime "@1411983272" ./clock
```

Alternatifnya, putar ulang dekripsi dengan Python:

```python
import hashlib, struct

epoch = 0x542947a8
ct = open("clock", "rb").read()[0x2020:0x2020 + 0x3b]

stream = b""
for counter in range(3):
    stream += hashlib.sha256(
        struct.pack("<Q", epoch) +
        struct.pack("<Q", 0x542947a8) +
        struct.pack("<I", counter)
    ).digest()

plaintext = bytes(a ^ b for a, b in zip(ct, stream))
print(plaintext)
```

Teks biasa berukuran 59 byte adalah flag-nya. Flag yang diterima adalah `████████████████`.