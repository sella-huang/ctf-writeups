# Archaeology of Mistakes

| | |
|---|---|
| **Bagian** | wave 5 |
| **Kategori** | forensics |
| **Jumlah Solve** | 194 |

## Ikhtisar

README memberikan gambar partisi mentah (~40 MB) dari SSD laptop Elena dan menyatakan bahwa metadata sistem file tidak meyakinkan, tetapi bloknya tidak. Elena memotret sesuatu pada 06/05, menghapus foto itu beberapa jam kemudian, dan kemudian menulis lampiran penelitian yang cukup besar untuk menggunakan kembali bagian dari blok yang dibebaskan, sehingga merusak bagian tengah JPEG.

Kalimat penutupnya — *yang terpenting bukanlah apa yang ditampilkan* — adalah desain tantangannya. JPEG dimulai dengan segmen `FFD8` dan `APP1`/Exif; gambar itu sendiri adalah data pemindaian berkode entropi yang muncul setelahnya. Penimpaan di tengah akan menghancurkan gambar dan membiarkan metadata tidak tersentuh.

Gambar tidak menampilkan dirinya sebagai sistem file:

```
$ file disk.raw
disk.raw: data

$ exiftool disk.raw
File Type        : URL
MIME Type        : application/x-mswinurl
```

ExifTool membaca byte terdepan, memutuskan seluruh file 42 MB adalah pintasan Windows `.url`, dan tidak melaporkan apa pun yang berguna. `head` menunjukkan alasannya: blok tersebut merupakan campuran datar dari file konfigurasi, baris log, dan XML dari file yang tidak terkait.

## Penyelesaian

`binwalk -e disk.raw` melaporkan tidak ada tanda tangan yang dapat digunakan dan tidak mengekstrak apa pun, jadi file harus diukir:

```bash
photorec disk.raw
```

Pengukiran tersebut menghasilkan `recup_dir.1` dengan ratusan file XML dan delapan JPEG. Masing-masing membawa blok EXIF ​​lengkap:

```bash
exiftool -DateTimeOriginal -UserComment *.jpg
```

| DateTimeOriginal | UserComment |
|---|---|
| 2026:06:02 10:12:03 | SPR reading room shelving, sealed folio area. token=VVXWD4RLUUNWMIAP |
| 2026:05:28 14:03:11 | Halberd lobby — waiting for curator, afternoon. token=ECQYLNUPY5XAI6PG |
| 2026:05:19 15:44:10 | Dissertation notebook, ch.3 marginalia. token=KXV77EBHNIX6CVOS |
| 2026:05:12 11:04:22 | Hollis Special Collections archive, west aisle. token=CXGQVX6ZAVZA3GTH |
| 2026:04:15 10:40:15 | Private library annex — east wing shelving. token=4IKEMTUJCDYLSZQ4 |
| 2026:04:28 09:16:57 | Latte at Redoubt Coffee, before advisor meeting. token=WOZ24DYJ7UUE5PVW |
| 2026:05:31 13:31:42 | SPR reading room monograph photocopy p.47. token=GDBAG3425RFRSNX3 |
| 2026:06:08 09:21:40 | Archive reviewer notes — Hollis manuscript annex. token=6T6OKAAEP27GX4VT |

Delapan foto, delapan token, dan tidak satu pun dari 06/05. File yang hilang adalah jawabannya.

PhotoRec hanya mengeluarkan file yang dapat dibatasi, dan foto yang dihapus kehilangan data pemindaiannya saat ditimpa, sehingga tidak ada JPEG kesembilan — tetapi header EXIF-nya berada sebelum kerusakan dan masih ada di dalam gambar. Pencarian string pada gambar mentah menunjukkan **sembilan** token untuk delapan file:

```bash
strings disk.raw | grep -i "token="
```

Yang tambahan adalah header yang tidak memiliki file:

```
Private library annex
 east wing shelving. token=████████████████ .Google Pixel 8 rear camera .
```

Ini dikenali sebagai rekaman EXIF nyata, bukan noise, karena `grep -C 2` menunjukkannya dikelilingi oleh tetangga yang sama seperti komentar lainnya — penanda tipe `ASCII`, model lensa, lalu data skor biner. Ini adalah file yang salah dirender: tidak dapat dirender sama sekali, itulah sebabnya metadatanya adalah satu-satunya yang tersisa untuk dibaca.Ada dua jebakan yang perlu diperhatikan. Setiap foto membawa token, jadi mengambil `token=` akan menghasilkan sembilan kandidat, delapan di antaranya adalah umpan. Dan satu foto utuh bulan April memuat teks `Private library annex — east wing shelving` *sama* dengan token berbeda (`4IKEMT…`), sehingga string lokasinya saja pun ambigu. Diskriminatornya adalah hitungannya: sembilan header, delapan file.

```
POCTF{████████████████}
```

Untuk media yang telah dihapus dan kemudian ditimpa sebagian, bagian file yang dapat diandalkan adalah bagian tepinya — header terlebih dahulu. Pengukiran memberikan serangkaian kandidat, dan artefak yang layak dikejar sering kali adalah artefak yang ditolak oleh pemahat untuk diproduksi.