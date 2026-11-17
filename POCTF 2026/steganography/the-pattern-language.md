# The Pattern Language

| | |
|---|---|
| **Bagian** | wave 8 |
| **Kategori** | steganography |
| **Jumlah Solve** | 50 |

## Ikhtisar

`pattern.png` adalah contoh tartan berukuran 440 kali 440 dengan batas 20 piksel dan kisi 25 piksel berukuran 16 kali 16. Setiap sel adalah kotak-kotak satu piksel yang terbuat dari dua warna. Nuansanya merupakan kelipatan dari `0x11`, sehingga merupakan camilan `0x0` hingga `0xf` yang dirender sebagai skala abu-abu.

Tenunan polos adalah petunjuknya: di setiap persimpangan, piksel yang terlihat persis merupakan salah satu dari dua warna benang. Warna yang umum pada sebuah kolom adalah benang lusi; warna yang umum pada satu baris adalah benang pakan. Membaca kolom dan baris menghasilkan dua set 16 camilan:

```text
warp: 7899d6bac0a05dfc
weft: ebf0bfd3fca1fb61
```

## Penyelesaian

Nonce bukanlah gabungan dari himpunan. Gambar tersebut merupakan konstruksi kriptografi visual dua bagian: lungsin dan pakan adalah bagian yang independen, dan rahasianya adalah XOR-nya. Interpretasikan setiap sett sebagai nilai 8-byte dan XOR mereka:

```python
warp = int("7899d6bac0a05dfc", 16)
weft = int("ebf0bfd3fca1fb61", 16)
print(f"{warp ^ weft:016x}")
```

Hasilnya adalah nonce mentah 16-hex-karakter. Pengiriman nilai tersebut pada halaman tantangan diterima. Nonce mentah yang diterima adalah `████████████████`.