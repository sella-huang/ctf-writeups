# Leave No Trace

| | |
|---|---|
| **Bagian** | wave 7 |
| **Kategori** | misc |
| **Jumlah Solve** | 104 |

## Ikhtisar

Targetnya adalah beranda pribadi statis kecil untuk "M. Lange", terakhir diperbarui pada tahun 2019. Halaman yang dirender adalah daftar bacaan dan catatan kontak; petunjuknya mengatakan bahwa halaman tersebut bukan situsnya. Flag nonce terdiri dari enam belas karakter, dibagi menjadi empat fragmen empat karakter, masing-masing disembunyikan di permukaan HTTP berbeda dan diberi label berdasarkan posisinya. Oleh karena itu, solusinya tidak ada di halaman sama sekali: solusinya ada di sumber HTML, header respons, `robots.txt`, dan metadata favicon.

Situs ini disajikan di bawah `/challenges/leave-no-trace/site/` dan cookie sesi platform diperlukan. Fragmen ditemukan dengan `curl -i` dan sejumlah kecil inspeksi biner, bukan dengan kekerasan direktori.

## Penyelesaian

Permukaan pertama adalah badan HTML. Melihat sumbernya menunjukkan komentar di atas `<body>`:

```html
<!-- site: personal-static; fragment 1/4: ec15 -->
```

Itu adalah fragmen `1/4: ec15`.

Permukaan kedua adalah header respons HTTP. `GET` atau `HEAD` di root situs kembali, di antara header keamanan biasa:

```
x-powered-by: handwritten HTML
x-site-fragment: 2/4:b7b6
```

Header `x-site-fragment` khusus membawa `2/4: b7b6`. Itu tidak terlihat di halaman yang dirender dan di sumber isi; hanya header respons yang memaparkannya.

Permukaan ketiga adalah `robots.txt`:

```
# Personal site — please respect crawl-delay.
User-agent: *
Crawl-delay: 5
Disallow: /drafts/
Disallow: /notes/private/
Disallow: /.rev-0de5/     # build tag; ignore
Disallow: /_backup/

Sitemap: /sitemap.xml
```

Baris `Disallow: /.rev-0de5/` menyamarkan fragmen sebagai tag revisi/pembuatan. `0de5` adalah token empat karakter, dan komentar `# build tag; ignore` adalah penyesatan: hal yang penulis katakan untuk diabaikan adalah apa yang tertinggal. Ini mengisi slot `3/4` yang tersisa.

Permukaan keempat adalah favicon. Halaman tersebut menghubungkannya sebagai `favicon.png`; mengambil PNG dan menjalankan `strings` di atasnya menunjukkan dua potongan `tEXt`:

```
tEXtDescription
personal-site favicon; ignore
tEXtFragment
4/4:a5c9
```

Potongan `tEXtFragment` adalah `4/4: a5c9`. `tEXtDescription` juga mengatakan "abaikan"; fragmennya ada di bongkahan tetangga, bukan di deskripsi. `exiftool favicon.png` menampilkan potongan teks PNG yang sama dalam bentuk yang lebih terstruktur.

Keempat bagian tersebut secara berurutan adalah:

```
1/4  ec15
2/4  b7b6
3/4  0de5
4/4  a5c9
```

Digabungkan, nonce adalah:

```
████████████████
```

Titik akhir pengiriman adalah `POST /challenges/leave-no-trace/submit` dengan isi JSON:

```http
POST /challenges/leave-no-trace/submit
Content-Type: application/json

{"answer":"████████████████"}
```

Tanggapannya adalah:

```json
{"correct":true,"message":"Correct. Marker recorded."}
```

Titik akhir menerima pemisah atau tidak sama sekali dan tidak peka huruf besar-kecil; ia juga menerima penanda `POCTF{...}` lengkap. Mengirimkan nonce saja sudah cukup.

Perintah yang digunakan, dengan cookie sesi dihilangkan:

```bash
BASE='https://pointeroverflowctf.com/challenges/leave-no-trace/site'
curl -i "$BASE/" -b 'session=<cookie>'
curl -i "$BASE/robots.txt" -b 'session=<cookie>'
curl -s "$BASE/favicon.png" -b 'session=<cookie>' | strings -n 4
```Jalan buntu yang bersifat instruktif adalah “permukaan” yang tersisa yang tampak menjanjikan namun kosong. `sitemap.xml`, yang direferensikan oleh `robots.txt`, menghasilkan 404: peta situs adalah salah satu jejak yang telah dihapus, bukan fragmennya. Direktori yang tidak diizinkan (`/drafts/`, `/notes/private/`, `/_backup/`, dan `/.rev-0de5/`) semuanya menghasilkan 404; `/.rev-0de5/` bukan direktori untuk dicantumkan, ini adalah tag build yang membawa fragmen. `OPTIONS /` mengembalikan `Allow: GET, HEAD, OPTIONS` tanpa fragmen, `TRACE` mengembalikan 405, dan `/.well-known/security.txt` dan `humans.txt` mengembalikan 404. Nama file yang memaksa tidak diperlukan; keempat permukaan sudah terlihat dari permintaan normal.

Polanya adalah inti dari tantangannya: situs statis lebih dari sekadar tubuh yang dirender. Header respons, kebijakan perayap, dan aset biner adalah permukaan HTTP yang terpisah, dan masing-masing dapat menyimpan sesuatu yang dilupakan oleh pembuatnya. Dua komentar "abaikan" — di `robots.txt` dan di deskripsi favicon — adalah penyesatan yang sama, dan fragmennya berada di sebelahnya.

Nonce-nya adalah `████████████████`, dan penandanya adalah `POCTF{████…}`.