# Where the Light Fails to Fall

| | |
|---|---|
| **Bagian** | wave 1 |
| **Kategori** | osint |
| **Jumlah Solve** | 516 |

## Ikhtisar

Laporan singkatnya adalah foto seekor merpati - Sir Marcus Skuttlebanks, tampaknya sering bepergian - dengan dua anotasi: panah merah menunjuk ke utara, dan bayangan merpati jatuh ke kanannya. Pengamatan diberi stempel waktu `2026-06-20 · 19:55 · UTC+02:00`. Pertanyaannya adalah kotanya.

Hanya ada satu bukti kuat dalam gambar tersebut: arah bayangan relatif ke utara. Segala sesuatu yang lain adalah pemandangan. Sebuah bayangan menunjuk tepat jauh dari matahari, dan azimuth matahari pada saat tertentu merupakan fungsi dari garis lintang dan garis bujur, sehingga foto tersebut bukanlah soal foto melainkan soal geometri matahari dengan jawaban yang unik.

## Penyelesaian

Panah merah mendefinisikan bingkai, sehingga arah bayangan dapat diukur langsung pada bidang gambar. Hamparkan busur derajat dengan garis dasarnya pada panah — yang ada di layar seperti `onlineprotractortool.org` sudah cukup — dan baca searah jarum jam dari utara:

```
shadow bearing ≈ 109°
```

Bayangan merupakan arah yang berlawanan dengan matahari, sehingga matahari berada pada :

```
sun azimuth = shadow bearing + 180° = 289°
```

Itu adalah matahari sore di barat laut, yang cocok dengan stempel waktu.

`2026-06-20 19:55 UTC+02:00` adalah `17:55 UTC`. Pada tanggal 20 Juni pada jam tersebut matahari masih terbit di seluruh Eropa bagian barat dan tengah, jadi siang hari saja bukanlah sebuah diskriminator. Offset pada stempel waktu adalah: jam berjalan CEST, yang menempatkan lokasi di sabuk UTC+2 dan menghapus London, kota UTC+1. Yang tersisa di antara kota-kota besar dengan matahari masih terbit adalah Paris dan Amsterdam.

Memasukkan tanggal, waktu dan zona waktu CEST ke dalam `suncalc.org`, lalu mengklik masing-masing kandidat dan membaca azimuth dari panel info:

| City | Sun azimuth | Implied shadow bearing |
|---|---|---|
| Paris | 284° | 104° |
| Amsterdam | 289° | **109°** |

Amsterdam sama persis dengan fotonya, Paris berjarak lima derajat. Lima derajat itu kecil, itulah sebabnya sudutnya harus diukur daripada diperkirakan dengan mata, dan mengapa filter zona waktu layak dilakukan terlebih dahulu: filter ini mengubah "kota mana pun di Eropa" menjadi daftar dua item sebelum busur derajat keluar.

Panah utara inilah yang membuat gambar dapat dipecahkan: ia mengubah gambar menjadi sebuah bantalan, dan sebuah bantalan ditambah stempel waktu memotong sebuah titik di globe.

Kotanya adalah **Amsterdam**.