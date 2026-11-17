# Search log — Following the Money

Ini adalah versi bersih dari catatan yang saya simpan selama penyelesaian. Sengaja dibuat lebih detail daripada tulisannya: jalan buntu itulah intinya. Kredensial, cookie sesi, kunci tim, dan flag dihilangkan.

## 0. Artefak

Tantangan tersebut mengirimkan ZIP dengan tiga puluh fragmen PNG. Hash arsipnya adalah:

```text
864cbc23a8756ed94d793f6f81259b47857b71d16268ca3ad0d2ca7f9cd6b90a
```

Setiap PNG adalah potongan robek dengan topeng alfa. Potongan-potongannya bukanlah kisi-kisi sederhana: sobekannya saling tumpang tindih dan ujung-ujungnya tidak beraturan.

## 1. Jigsaw (malam 1, ~1 jam)

Upaya pertama: urutkan berdasarkan kotak pembatas alpha-mask dan letakkan potongannya pada kotak 6x5. Ini mendapatkan tata letak umum tetapi membiarkan garis terukir tidak sejajar. Caranya adalah dengan mencocokkan bagian tepi yang robek, bukan kotak pembatas persegi panjang: topeng alfa adalah bentuknya, dan garis yang diukir terus melintasi sobekan.

Setelah rekonstruksi, serial tersebut terlihat di dua tempat. Pangkas kanan atas adalah yang paling bersih:

```text
E10093751B
```

Uang kertas tersebut adalah seri $1 tahun 2017 dengan tanda tangan Jovita Carranza. Awalan `E` adalah surat distrik Federal Reserve untuk Richmond; `B` di belakangnya adalah huruf besar.

## 2. Di mana George (malam 1, ~1 jam)

Tantangannya memberi `b:Fuzhs8rJ7u`. URL laporan langsungnya adalah:

```text
https://www.wheresgeorge.com/report.php?b=Fuzhs8rJ7u
```

URL langsung membuka halaman tagihan, namun riwayat perjalanan tidak terlihat tanpa pencarian. Formulir pencarian situs menerima kode pos dan serial; menggunakan ZIP dari riwayat tagihan mengonfirmasi bahwa serial tersebut ada. Riwayat kemudian menampilkan entri, termasuk Entri Awal. Catatan tersebut menunjuk pada:

```text
https://pointeroverflowctf.com/challenges/following-the-money/astroblog/
```

Kami menggunakan formulir situs satu kali dan tidak menambahkan entri lebih lanjut. Catatan tersebut merupakan satu-satunya bagian laporan yang digunakan dalam rantai; riwayat perjalanannya sendiri tidak.

## 3. Blog (1 malam, ~30 menit)

Blog tersebut memiliki satu postingan yang relevan, #137, "Jembatan, akhirnya bersih". Judulnya memberi:

```text
2024-10-12T21:18:38Z
```

Halaman tersebut menyatakan bahwa gambar tersebut memiliki tinggi 1620 piksel dengan FOV vertikal 45,9°. Postingan tersebut mencakup dua gambar: bidang bintang dan versi "bertanda cakrawala". Gambar kedua adalah lelucon redaksi: panel ungu `[REDACTED]` menutupi bagian bawah, deretan gambar kartun berada di atasnya, dan pita hitam berlabel `HORIZON` menandai baris 1443. Cakrawala tidak digambar sebagai garis; band adalah penandanya.

Terdapat perbedaan stempel waktu: EXIF ​​gambar yang diedit menyatakan `2024:10:12 22:44:26`, namun keterangan blog menyatakan `21:18:38Z`. Judulnya adalah nilai yang menghasilkan garis bujur yang diterima; waktu EXIF ​​adalah metadata lokal/yang diedit dan tidak digunakan.Postingan #136, "Leonids di atas taman belakang", menyebutkan lampu jalan di Harrow Road. Kami mencoba menggunakannya untuk menandai kota, tetapi tidak menentukan garis bujur; itu adalah teks rasa, bukan koordinat.

## 4. Jalan memutar jembatan (malam 2, ~6 jam)

Garis lintang dalam tantangan ini adalah 54.1116 N. Langkah yang jelas adalah mengidentifikasi jembatan dan menggunakan koordinatnya. Kami mencoba:

| Kandidat | Lintang | Bujur | Hasil |
|---|---:|---:|---|
| Ribblehead | 54.2104 | -2.3702 | visually close, ~11 km north of the target latitude |
| Clapham / Wenning | 54.1041 | -2.4085 | close in latitude, wrong arch count and curve |
| Skipton & Wennington (unnamed) | 54.1066 | -2.4494 | close in latitude, wrong perspective |
| Helwith Bridge group | 54.1235 | -2.2909 | too far east, wrong setting |
| Bentham line | 54.1125 | -2.4911 | close in latitude, wrong viaduct |

Ribblehead adalah jebakannya: ini adalah jembatan yang terkenal, berada di area umum kanan, dan bidang bintang tampak masuk akal di atasnya. Namun letaknya 0,0988° utara dari garis lintang yang disebutkan, sekitar 11 km. Tantangannya tidak pernah meminta jembatan. Ia menanyakan garis bujur kamera.

Kami juga menguji hipotesis bulan: postingan tersebut menyebutkan bulan akan datang saat musim gugur, sehingga posisi bulan pada waktu pengambilan gambar mungkin membatasi azimuth. Itu tidak menghasilkan garis bujur yang unik.

## 5. Kuburan API (malam 2, ~2 jam)

Untuk memeriksa kandidat yang kami coba:

- Overpass API: 406 Tidak Dapat Diterima dengan agen pengguna default, lalu batas waktu pada kueri yang lebih besar.
- Wikidata SPARQL: 429 Terlalu Banyak Permintaan setelah beberapa pertanyaan.
- API Geografi: 403 Dilarang.
- Cookie sesi Flask di situs tantangan menjadi basi di tengah penyelesaian; mengautentikasi ulang memperbaikinya.

Data yang berguna berasal dari ubin OSM dan Wikipedia, bukan dari API.

## 6. Astrometri (malam 2, ~2 jam)

Astrometry.net memecahkan bidang bintang. Bintang referensi WCS ada di:

```text
CRVAL1 =   9.5355366203°   (RA)
CRVAL2 =  31.2660622861°   (Dec)
CRPIX1 =  63.2815876007
CRPIX2 = 660.067301432
```

Kesalahan pertama adalah menggunakan bintang di pusat gambar. Bintang referensi WCS bukanlah pusatnya; itu adalah bintang di `CRPIX`. Kesalahan kedua adalah menggunakan cabang bola penuh: dengan ketinggian dan garis lintang, Anda dapat memulihkan sudut jam dan kemudian garis bujur, namun petunjuk tantangan mengurangi masalah ke hubungan meridian. `GST - RA` literal yang menggunakan bintang referensi WCS adalah jalur yang diterima.

## 7. Rumusnya (malam 2, ~30 menit)

Ketinggian:

```text
alt = (1443 - 660.067301432) * 45.9 / 1620
    = 22.1831°
```

Waktu sideris:

```text
GST = 280.46061837 + 360.98564736629 * (t - J2000) / 86400  (mod 360)
    = 341.595636°
```

Garis bujur:

```text
longitude = GST - RA
          = 341.595636 - 9.535537
          = 332.060099°  ->  -27.939901°
```

Mengirimkan `-27.939901` mengembalikan bendera. Flag tersebut disunting dalam tulisan publik.

## 8. Garis Waktu

| Phase | Time |
|---|---:|
| Jigsaw | ~1 h |
| Where's George | ~1 h |
| Blog | ~0.5 h |
| Viaduct detour | ~6 h |
| API graveyard | ~2 h |
| Astrometry | ~2 h |
| Formula | ~0.5 h |
| **Total** | **~13 h** |

## 9. Apa yang akan saya lakukan secara berbeda1. Baca petunjuknya secara harfiah. `GST - RA` adalah keseluruhan formula; cabang bola adalah gangguan.
2. Gunakan bintang referensi WCS, bukan pusat gambar.
3. Perlakukan jembatan sebagai perasa, bukan sebagai sumber koordinat. Keleluasaan diberikan; garis bujur tidak diketahui.
4. Simpan log pencarian dari menit pertama. Jalan buntu adalah tulisannya.