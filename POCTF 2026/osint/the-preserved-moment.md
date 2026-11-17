# The Preserved Moment

| | |
|---|---|
| **Bagian** | wave 5 |
| **Kategori** | osint |
| **Jumlah Solve** | 138 |

## Ikhtisar

Subjeknya adalah Econo Foods, sebuah toko kelontong di kota kecil yang dimulai sebagai koperasi toko yang dimiliki secara mandiri. Ketika distributor yang lebih besar pindah untuk membeli merek tersebut, masing-masing toko memutuskan sendiri apakah akan menerimanya, sehingga nama yang sama berakhir di bawah beberapa pemilik, di beberapa domain, selama beberapa tahun. Laporan singkatnya menggarisbawahi poinnya: satu merek, banyak toko, dan penekanan pada tanggal.

Pertanyaannya adalah satu angka dari tahun 2006: berapa banyak waktu yang diklaim dapat dihemat oleh resep Ham dan Kentang Au Gratin dengan menggunakan bahan-bahan beku. Halaman yang menjawabnya sudah tidak ada lagi di web langsung, jadi ini adalah masalah arsip — dan dua variabelnya adalah domain dan tahun.

## Penyelesaian

Spanduk tersebut telah ada di lebih dari satu domain, dan nama tersebut juga telah digunakan oleh perusahaan yang tidak terkait di negara lain, jadi langkah pertama adalah memasang pin pada situs era 2006 daripada memercayai situs favorit mesin pencari. Untuk pertengahan tahun 2000-an situs yang relevan adalah `econofoods.com`.

Daripada mengklik kalender Wayback bulan demi bulan, hitunglah apa yang sebenarnya disimpan dalam arsip tersebut. CDX API mengembalikan setiap URL yang diambil untuk domain dalam rentang tanggal:

```
https://web.archive.org/cdx/search/cdx?url=econofoods.com&matchType=domain&from=20060101&to=20061231&fl=original&collapse=urlkey&limit=200
```

Di antara hasilnya, situs tahun 2006 memperlihatkan direktori resep dengan satu file per hidangan:

```
/recipes.html
/recipes/cass_hamhashb.htm
/recipes/cass_hamtatoesaugr.htm
/recipes/cass_maccheese.htm
/recipes/cass_reuben.htm
```

`cass_` adalah awalan casserole, dan `hamtatoesaugr` adalah hidangannya. Menjelajahi indeks adalah jalan buntu — segelintir tautan yang sama disajikan di setiap pengambilan — tetapi masing-masing file resep adalah URL terpisah, dan daftar CDX menamainya secara langsung.

Ambil tangkapan mentah, tanpa toolbar arsip:

```
https://web.archive.org/web/20060101000000id_/http://www.econofoods.com/recipes/cass_hamtatoesaugr.htm
```

Halaman yang diarsipkan mencantumkan bahan-bahannya — bawang bombay beku yang dipotong dadu, kentang goreng beku yang diparut, kacang polong dan wortel beku — dan ditutup dengan baris yang dikutip dari tantangan tersebut:

```
Time Saved Using Frozen Ingredients: 25 Minutes!
```

Pelajarannya adalah arsip adalah daftar URL, bukan situs web. Penjelajahan kalender hanya menampilkan laman indeks yang tertaut pada hari ketika seseorang merayapinya; CDX API menunjukkan segala sesuatu yang pernah diambil, termasuk halaman daun yang indeksnya diam-diam berhenti ditautkan.

Jawabannya adalah **25 menit**.