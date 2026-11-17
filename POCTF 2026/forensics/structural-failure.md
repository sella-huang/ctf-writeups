# Structural Failure

| | |
|---|---|
| **Bagian** | wave 7 |
| **Kategori** | forensics |
| **Jumlah Solve** | 136 |

## Ikhtisar

Artefaknya adalah ZIP 2.568 byte bernama `meridian-correspondence.zip`. Direktori pusat dan catatan Direktori Akhir Pusat telah ditimpa dengan sampah; tiga header file lokal dan aliran DEFLATE-nya masih ada. Oleh karena itu, arsip tersebut pada prinsipnya dapat dibaca, tetapi tidak dapat dibaca oleh alat yang dimulai di EOCD. `file` salah mengidentifikasinya sebagai kunci rahasia OpenPGP, dan `zipfile` Python memunculkan `BadZipFile: File is not a zip file`.

Ada satu kerutan tambahan. Byte pertama dari setiap tanda tangan header lokal rusak. Tanda tangan yang diharapkan adalah `50 4b 03 04` (`PK\x03\x04`), tetapi arsipnya berisi:

```text
00000000: 94 4b 03 04 ...
000002a9: 53 4b 03 04 ...
00000451: 24 4b 03 04 ...
```

Ekor `4b 03 04` bertahan di ketiga header, sehingga header masih dapat ditemukan dengan memindai ketiga byte tersebut dan mundur satu byte. Sisa dari setiap header lokal — metode kompresi, CRC, ukuran terkompresi, ukuran tidak terkompresi, panjang nama file, dan nama file — masih utuh. Itu cukup untuk mendekompresi entri tanpa membangun kembali direktori pusat.

## Penyelesaian

Mulailah dengan mengonfirmasi kerusakan:

```bash
file meridian-correspondence.zip
python3 -c 'import zipfile; zipfile.ZipFile("meridian-correspondence.zip")'
```

Perintah pertama mencetak `OpenPGP Secret Key Version 3`; cetakan kedua `BadZipFile: File is not a zip file`. Pencarian untuk `PK` atau `PK\x03\x04` tidak menemukan apa pun. Pencarian untuk `4b 03 04` yang masih ada menemukan tepat tiga kejadian, pada offset `1`, `682`, dan `1106`; mengurangi satu menghasilkan tiga header lokal dimulai: `0`, `681`, dan `1105`.

Format tajuk lokal adalah:

```text
signature            4 bytes
version needed       2 bytes
flags                2 bytes
compression method   2 bytes
mod time             2 bytes
mod date             2 bytes
CRC-32               4 bytes
compressed size      4 bytes
uncompressed size    4 bytes
filename length      2 bytes
extra length         2 bytes
filename             variable
extra field          variable
compressed data      variable
```

Mengurai tiga header menghasilkan:

| entry | header offset | data offset | compressed | uncompressed | CRC-32 |
|---|---|---:|---:|---:|---|
| `01_halberd_correspondence.txt` | `0` | `59` | 622 | 1128 | `f63692eb` |
| `02_steward_response.txt` | `681` | `734` | 371 | 663 | `2494a223` |
| `03_marchetti_annotations.md` | `1105` | `1162` | 1119 | 2121 | `64ef8dae` |

Ketiga entri menggunakan metode `8` (DEFLATE) dengan flag `0`, sehingga ukuran terkompresi ada di header lokal dan tidak ada deskriptor data. Data entri terakhir berakhir pada offset `2281`; byte `287` yang tersisa adalah direktori pusat dan EOCD yang ditimpa. Tidak ada `PK\x01\x02` atau `PK\x05\x06` di bagian ekor itu, itulah sebabnya alat standar menolak file tersebut.

Skrip pemulihan minimal menemukan header, mendekompresi aliran DEFLATE mentah, dan memeriksa setiap CRC:

```python
import struct, zlib

b = open("meridian-correspondence.zip", "rb").read()

starts = [i - 1 for i in range(len(b) - 2) if b[i:i+3] == b"\x4b\x03\x04"]

for start in starts:
    sig, ver, flags, method, mtime, mdate, crc, csize, usize, fnlen, extralen = \
        struct.unpack_from("<4sHHHHHIIIHH", b, start)
    name = b[start+30:start+30+fnlen].decode("utf-8")
    data_start = start + 30 + fnlen + extralen
    data = b[data_start:data_start+csize]
    out = zlib.decompress(data, -15)
    assert len(out) == usize, (name, len(out), usize)
    assert (zlib.crc32(out) & 0xffffffff) == crc, name
    open(name, "wb").write(out)
    print(f"{name}: {len(out)} bytes, CRC {crc:08x} OK")
```

Outputnya adalah:

```text
01_halberd_correspondence.txt: 1128 bytes, CRC f63692eb OK
02_steward_response.txt: 663 bytes, CRC 2494a223 OK
03_marchetti_annotations.md: 2121 bytes, CRC 64ef8dae OK
```

Pemeriksaan CRC adalah bagian yang penting: header lokal berisi CRC-32 yang diharapkan untuk setiap entri, dan setiap aliran yang didekompresi cocok. Pemulihannya bukanlah sebuah dugaan.Tiga file yang dipulihkan menceritakan kisah Lingkaran Meridian. `01_halberd_correspondence.txt` adalah permintaan Marion Halberd kepada Pengurus Circle: peralatan Frame IV masih merespons, dia telah mendokumentasikan tujuh sesi yang telah selesai, dan dia ingin restu Circle untuk meresmikan operasi sebelum "subjek untuk sesi mendatang" digunakan. `02_steward_response.txt` adalah penolakan C. Whitmore: perjanjian tidak dapat diubah, permintaan ditolak, dan salinan arsip tanggapan tidak akan disimpan. `03_marchetti_annotations.md` adalah kumpulan anotasi Elena. Baris pertamanya berisi referensi map:

```text
# MARCHETTI 2026-14 docket ref: ████████████████
```

Anotasi tersebut juga mencatat rantai asal — Halberd ke Steward pada 09/06, Steward ke Halberd pada 11/06, Delacroix ke Marchetti pada 22/06, Marchetti ke dirinya sendiri pada 28/06 — dan instruksi Elena bahwa berkas perkara harus sampai ke saudara perempuannya terlebih dahulu, penasihatnya yang kedua, dan bukan Society, kantor hukum universitas, atau Halberd.

Jalan buntu itu berguna. `file` menebak OpenPGP karena byte pertama dari tanda tangan pertama adalah `0x94` dan bukan `0x50`; arsipnya masih berupa ZIP. `zipfile` dan alat berbasis EOCD lainnya berhenti sebelum membaca header lokal, sehingga tidak dapat mencantumkan arsip meskipun entri masih utuh. Membangun kembali direktori pusat tidak diperlukan: header lokal berisi nama, metode, ukuran, dan CRC untuk setiap entri. Klaim README bahwa file terakhir dilihat berukuran 240 KB adalah narasi; artefak sebenarnya adalah 2.568 byte, dan tiga entri lokal mencakup semuanya kecuali ekor sampah 287 byte. Aliran DEFLATE mentah harus didekompresi dengan ukuran jendela negatif (`zlib.decompress(data, -15)`); `zlib.decompress(data)` normal mengharapkan header zlib dan gagal karena alirannya DEFLATE mentah.

Di halaman artefak, mengirimkan token mentah atau `POCTF{████████████████}` akan mengembalikan:

```json
{"correct":true,"message":"Correct."}
```

Tokennya adalah referensi map 16 karakter dari anotasi Elena, dan penanda lengkapnya adalah `POCTF{████…}`.

Referensi map dalam anotasi Elena adalah `████████████████`, dan flag-nya adalah `POCTF{████…}`.