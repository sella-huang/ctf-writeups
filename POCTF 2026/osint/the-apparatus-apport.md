# The Apparatus, Apport

| | |
|---|---|
| **Bagian** | wave 4 |
| **Kategori** | osint |
| **Jumlah Solve** | 274 |

## Ikhtisar

Sebuah paket tanpa alamat pengirim tiba di Liminal Society for Spectral Fellowship. Ini berisi lima file yang dipulihkan: empat file disatukan di sekitar satu orang, dan satu item yang tidak dapat dijelaskan oleh petugas arsip. Tidak ada yang menyebutkan nama subjek secara langsung, sehingga gambar tersebut harus direkonstruksi melalui referensi silang. Pengirimannya adalah `SURNAME_YEAR`, dengan tahun yang merupakan peristiwa biografi penting yang juga muncul di suatu tempat di file (misalnya, `BLAVATSKY_1875`).

## Penyelesaian

Arsip ini menampung lima file:

```
coordinates.gpx
correspondence.eml
recovered.jpg
transcript.pdf
unaccounted_transmission.txt
```

`coordinates.gpx` adalah kerangka terkuat. Empat titik jalannya sudah memberikan garis besar sebuah kehidupan:

| waypoint | time |
|---|---|
| Birthplace (Nashua) | 1857-06-27 |
| Boston, James's first sitting | 1885-05-01 |
| Cambridge (Hodgson) | 1889-11-01 |
| Death (Boston) | 1950-07-03 |

`correspondence.eml` adalah balasan dari `archives@harvard.edu` ke `researcher@lssf.org.uk`, subjek `Re: William James — SPR correspondence file`. Badannya menyatakan bahwa James memulai penyelidikan pada tahun 1885 atas saran ibu mertuanya; bahwa mediumnya adalah seorang ibu rumah tangga Boston yang lahir di Nashua, New Hampshire, pada tahun 1857; dan bahwa Richard Hodgson dari Society for Psychical Research menjalankan penyelidikan Amerika sampai kematiannya pada tahun 1905.

`transcript.pdf` bukan teks biasa: aliran kontennya adalah ASCII85 plus Flate, jadi `pdftotext` dapat melewatkannya secara diam-diam. Menguraikan aliran secara langsung memulihkan tubuh:

```python
import re, zlib, base64

d = open('transcript.pdf', 'rb').read()
s = re.search(rb'stream\r?\n(.*?)endstream', d, re.S).group(1).strip().rstrip(b'~>')
print(zlib.decompress(base64.a85decode(s)).decode('latin1'))
```

Teks tersebut adalah catatan dari seorang profesor filsafat Harvard kepada rekannya di Cambridge tentang penyelidikan berkelanjutannya — dimulai pada tahun 1885 — terhadap media Boston yang kendali utamanya menamakan dirinya "Dr. Phinuit". Kutipannya: "Saya yakin Ny. P. mempunyai kekuatan yang belum dapat dijelaskan. Saya telah menyelidikinya selama dua puluh tahun..."

`recovered.jpg` membawa waktu EXIF `1887:11:15 21:30:00`, data GPS, dan deskripsi adegan ruang tamu: seorang wanita berpakaian polos duduk di kursi berlengan, tangan di pangkuan, mata tertutup, sementara seorang pria akademis membuat catatan di meja kecil. Tidak ada nama yang terukir pada gambar itu; itu adalah pembuktian, bukan label.

```
exiftool recovered.jpg
strings -a recovered.jpg | head
```

`unaccounted_transmission.txt` merupakan item kelima. Ini adalah fragmen `SEANCE.EXE` tanpa pengirim dan komentar samar tentang lingkaran tertutup; itu tidak menyebutkan nama orang, tempat atau tanggal. Ini adalah file yang tidak dapat dipertanggungjawabkan oleh para pengarsip, dan ini adalah sebuah kesalahan besar.Referensi silang dari empat file yang koheren menghasilkan seorang medium trance Amerika, seorang ibu rumah tangga Boston, lahir di Nashua, New Hampshire, pada tanggal 27 Juni 1857, yang kendali utamanya adalah "Dr. Phinuit", diselidiki oleh William James dari tahun 1885 dan oleh Richard Hodgson untuk SPR. Itu adalah Leonora Piper (née Leonora Evelina Simonds). Nama belakangnya adalah `PIPER`.

Tahun ini adalah keputusan kedua. File-file tersebut berisi tahun 1857, 1885, 1887, 1889, 1905 dan 1950. Pilihan yang jelas – 1857, tahun kelahiran – ada dan benar secara biografis, tetapi ini bukanlah peristiwa yang menjadi tantangannya. Tahun 1885 muncul dalam dua dari empat file yang koheren (email dan transkrip) dan diberi label secara eksplisit di GPX sebagai "Boston, James's first Sitting". Insiden inilah yang memulai penyelidikan dan membuat media tersebut terkenal, sesuai dengan "kelahiran, pendirian, publikasi, insiden penting". Acara yang dimaksud adalah sidang pertama tahun 1885.

Jawaban yang diterima adalah PIPER_1885.