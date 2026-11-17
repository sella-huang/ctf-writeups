# What Goes Unwritten

| | |
|---|---|
| **Bagian** | wave 3 |
| **Kategori** | forensics |
| **Jumlah Solve** | 313 |

## Ikhtisar

Stasiun kerja perpustakaan umum dihentikan secara paksa selama sesi Firefox. Satu-satunya artefak adalah minidump Windows, `library-workstation-firefox.dmp`. Tugasnya adalah memulihkan token pemulihan sesi base32 16 karakter yang disimpan Firefox di memori tetapi tidak pernah ditulis ke disk.

Dump tersebut adalah MDMP sintetis dengan empat aliran. Aliran yang berguna adalah `Memory64List` (tipe 9): 480 halaman 4K, semuanya dipetakan pada interval 4096 byte. Token tersebut bukan string 16 karakter yang bersebelahan. Ini dibagi menjadi dua catatan UTF-16LE dengan awalan panjang yang bertemu pada batas halaman. Oleh karena itu, pencarian `strings` langsung menemukan umpan yang masuk akal tetapi tidak menemukan tokennya.

## Penyelesaian

Direktori aliran ada di RVA `0x20`. Entri keempat adalah aliran `Memory64List` di RVA `0x21c`, ukuran `0x1e10`. 16 byte pertamanya adalah:

- Jumlah rentang `uint64` = 480
- Basis data `uint64` RVA = `0x202c`

Deskriptor mengikuti di `0x22c`, masing-masing 16 byte (`uint64 start`, `uint64 size`). Data halaman dimulai pada file offset `0x202c` dan disimpan dalam urutan deskriptor.

Catatannya menggunakan format:

```text
[uint32 total_len][uint32 magic][UTF-16LE string]
```

`total_len` menyertakan header 8 byte, sehingga string menempati byte `total_len - 8`. Sebagian besar catatan membawa keajaiban umpan `de c0 ed 5d` dan berisi frasa acak yang tampak seperti Firefox; ini mendominasi `strings -el`. Hanya dua catatan yang menggunakan keajaiban nyata `ba b0 ad de`.

Pemindai minimal pada halaman `Memory64List`:

```python
import re, struct

b = open("library-workstation-firefox.dmp", "rb").read()
n, base = struct.unpack_from("<QQ", b, 0x21c)
magic = b"\xba\xb0\xad\xde"

for _ in range(n):
    start, size = struct.unpack_from("<QQ", b, 0x21c + 16 + _ * 16)
    page = b[base:base + size]
    base += size

    for m in re.finditer(re.escape(magic), page):
        length = struct.unpack_from("<I", page, m.start() - 4)[0]
        raw = page[m.start() + 4 : m.start() + length - 4]
        print(f"{start + m.start() - 4:#x} len={length:#x} "
              f"{raw.decode('utf-16le').rstrip(chr(0))!r}")
```

Pemindai menghasilkan dua pukulan:

```text
0x14100228fbe len=0x42 'sessionRestoreToken:YZRNXWAB'
0x14100229000 len=0x40 'ETEGUEP3|expires:1751659200'
```

Catatan pertama dimulai pada `0x14100228fbe`, 66 byte sebelum akhir halaman di `0x14100229000`. Yang kedua dimulai tepat di halaman berikutnya. Oleh karena itu, urutan alamat virtual tidak ambigu: nilai `sessionRestoreToken` didahulukan, dan data `expires` mengikutinya.

Catatan pertama menyediakan delapan karakter setelah `sessionRestoreToken:`. Yang kedua memasok delapan karakter sebelum `|expires:`. Menggabungkan dua nilai delapan karakter dalam urutan alamat akan menghasilkan token 16 karakter. Pencarian apa pun untuk string base32 16 karakter yang berdekatan gagal karena batas halaman membagi nilainya; `strings` juga kehilangan urutan rekaman dan awalan panjangnya.

Token yang dipulihkan adalah `████████████████`.