# A Cabin Worth

| | |
|---|---|
| **Bagian** | wave 2 |
| **Kategori** | forensics |
| **Jumlah Solve** | 434 |

## Ikhtisar

Profil Linux Joplin 2.14.20 dicitrakan saat aplikasi sedang berjalan. Basis data utama adalah satu halaman 4 KiB; `database.sqlite-wal` 156.592 byte menampung 38 frame dari 4 halaman KiB. Tabel `notes` yang terlihat berisi empat catatan langsung dan tabel `deleted_items` berisi satu baris, namun catatan yang dihapus tidak lagi ada di halaman catatan saat ini. WAL masih menyimpan gambar halaman yang telah dihapus sebelumnya.

## Penyelesaian

Header WAL pada offset 0 adalah:

```text
magic 0x377f0682  version 3007000  page_size 4096
salt1 0xdb84c003  salt2 0x87bc0766
```

Setiap frame adalah header 24-byte diikuti oleh 4096 byte data halaman. Header bingkai terdiri dari enam nilai uint32 big-endian: nomor halaman, ukuran basis data setelah penerapan (nol untuk bingkai non-komit), garam-1, garam-2, checksum-1, checksum-2. Semua 38 frame melewati checksum WAL. Frame 38 adalah komit terakhir dan menyetel ukuran database menjadi 16 halaman.

Setelah menerapkan WAL, tabel `notes` saat ini adalah halaman 4 dengan empat baris. Tabel `deleted_items` ada di halaman 12:

```text
rowid 1: item_type=1  item_id=4b7c9e1f3a5b7c9e1f3a5b7c9e1f3a5b
         deleted_time=1751326272000 (2025-06-30 23:31:12 UTC)  sync_target=0
```

`sqlite_sequence` mengonfirmasi satu penghapusan. ID item yang dihapus tidak ada di halaman catatan saat ini, tetapi WAL berisi versi halaman 4 yang terakhir dihapus sebelumnya: bingkai 25. Bingkai 35 adalah halaman pasca-penghapusan 4. Mengurai bingkai 25 sebagai halaman daun tabel menghasilkan lima catatan; yang kelima adalah:

```text
id: 4b7c9e1f3a5b7c9e1f3a5b7c9e1f3a5b
parent_id: a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6  (Dissertation)
title: Delacroix — 06/28 confirmation
body: ""
encryption_cipher_text: 1772 base64 characters
encryption_applied: 1
```

Tabel `settings` menyediakan parameter enkripsi:

```text
note.encryption.algorithm      AES-256-GCM
note.encryption.kdf            PBKDF2-HMAC-SHA256
note.encryption.kdf_iterations 200000
note.encryption.blob_layout    salt(16) || nonce(12) || ciphertext || tag(16), base64
note.password_hint             my good widdle boy!
```

Petunjuknya menunjuk pada Horatio, anjing yang disebutkan dalam catatan live `shopping`. Kata sandinya adalah `horatio`.

Gumpalan catatan diterjemahkan menjadi garam 16-byte, nonce 12-byte, teks sandi 1.285 byte, dan tag GCM 16-byte. Mendapatkan kunci 32-byte dengan PBKDF2-HMAC-SHA256 melalui `horatio` dan garam, kemudian mendekripsi dengan AES-256-GCM, mengungkapkan isi catatan yang dihapus. Teks biasa berisi tanda kerja.

Ekstraktor/dekripsi minimal:

```python
import base64, hashlib, struct
from cryptography.hazmat.primitives.ciphers.aead import AESGCM

PAGE = 4096
WAL = open("database.sqlite-wal", "rb").read()

def varint(b, o):
    v = 0
    for i in range(8):
        c = b[o + i]
        v = (v << 7) | (c & 0x7f)
        if not c & 0x80:
            return v, o + i + 1
    return (v << 8) | b[o + 8], o + 9

def nbytes(t):
    if t == 0: return 0
    if 1 <= t <= 4: return t
    if t == 5: return 6
    if t == 6: return 8
    if t == 7: return 8
    if t in (8, 9): return 0
    return (t - 12) // 2 if t % 2 == 0 else (t - 13) // 2

def record(p):
    hl, o = varint(p, 0)
    types = []
    while o < hl:
        t, o = varint(p, o)
        types.append(t)
    o = hl
    out = []
    for t in types:
        n = nbytes(t)
        raw = p[o:o + n]
        o += n
        if t == 0:
            v = None
        elif t in (8, 9):
            v = t - 8
        elif t == 7:
            v = struct.unpack(">d", raw)[0]
        elif t >= 12:
            v = raw.decode() if t % 2 else raw
        else:
            v = int.from_bytes(raw, "big", signed=True)
        out.append(v)
    return out

# frame 25 = page 4, last pre-delete notes page
off = 32 + (25 - 1) * (24 + PAGE)
page = WAL[off + 24:off + 24 + PAGE]
nc = struct.unpack_from(">H", page, 3)[0]

for i in range(nc):
    ptr = struct.unpack_from(">H", page, 8 + 2 * i)[0]
    plen, o = varint(page, ptr)
    rowid, o = varint(page, o)
    row = record(page[o:o + plen])
    if row[0] == "4b7c9e1f3a5b7c9e1f3a5b7c9e1f3a5b":
        blob = base64.b64decode(row[21])
        salt, nonce, tag = blob[:16], blob[16:28], blob[-16:]
        key = hashlib.pbkdf2_hmac("sha256", b"horatio", salt, 200000, 32)
        print(AESGCM(key).decrypt(nonce, blob[28:-16] + tag, None).decode())
```

Flag yang dipulihkan adalah `POCTF{████…}`.