# Poisoned Well

| | |
|---|---|
| **Bagian** | wave 2 |
| **Kategori** | web |
| **Jumlah Solve** | 579 |

## Ikhtisar

Daily Apapun disajikan di belakang cache respons yang hanya dikunci pada jalur permintaan dan cookie sesi. Tautan kanonik halaman, tautan alternatif RSS, dan URL Grafik Terbuka dihasilkan dari header `X-Forwarded-Host`, yang tidak diperhitungkan oleh cache. Oleh karena itu, satu permintaan yang membawa host palsu menyimpan beranda yang diracuni di bawah kunci cache normal, dan pengunjung berikutnya menerima URL yang dikendalikan penyerang.

## Penyelesaian

Masuk di `/login` dengan token sesi tim mengembalikan cookie sesi bertanda tangan yang mengkodekan pengidentifikasi tim dan nonce tim. Halaman beranda kemudian merender metadatanya dari host permintaan:

```html
<link rel="canonical" href="https://poisoned-well.pointeroverflowctf.com/">
<link rel="alternate" href="https://poisoned-well.pointeroverflowctf.com/rss.xml" type="application/rss+xml">
<meta property="og:url" content="https://poisoned-well.pointeroverflowctf.com/">
```

Halaman itu sendiri mengiklankan kelemahannya: cache dikunci secara ketat pada jalur permintaan ditambah cookie sesi, dan semua header pengubah permintaan lainnya diperlakukan sebagai informasi. Menyediakan host yang berbeda melalui `X-Forwarded-Host` tercermin dalam ketiga URL tersebut sementara responsnya masih disimpan di bawah jalur biasa dan kunci cookie:

```bash
curl -b cookies.txt -D - \
  -H 'X-Forwarded-Host: news.example' \
  https://poisoned-well.pointeroverflowctf.com/
```

Respons tersebut muncul kembali dengan `X-Cache: MISS` dan dengan nilai kanonik, alternatif, dan `og:url` yang menunjuk pada `https://news.example/`. Permintaan kedua tanpa header kemudian mengenai entri beracun:

```bash
curl -b cookies.txt -D - https://poisoned-well.pointeroverflowctf.com/
```

Kali ini cache menjawab dengan `X-Cache: HIT`, dan HTML masih berisi host yang disediakan penyerang dalam metadatanya, meskipun permintaan tersebut membawa host yang sah. Header `Vary: Cookie` menjelaskan mengapa entri tersebut tercakup dalam sesi tim, sehingga halaman beracun yang sama disajikan untuk setiap permintaan berikutnya yang dibuat dengan cookie tersebut, termasuk milik penyerang.

Titik akhir verifikasi membaca host kanonik dari respons yang di-cache dan mengonfirmasi peracunan:

```bash
curl -b cookies.txt https://poisoned-well.pointeroverflowctf.com/verify
```

Ini melaporkan `"cached_host": "news.example/"` bersama dengan `"ok": true` dan penanda tim. Jika respons bersih secara tidak sengaja di-cache terlebih dahulu, `/reset` akan menghapus entri sehingga peracunan dapat terulang. Flag yang dikirimkan adalah POCTF{82.1814.████████████████.████████████████} dan diterima.