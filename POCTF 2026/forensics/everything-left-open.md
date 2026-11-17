# Everything Left Open

| | |
|---|---|
| **Bagian** | wave 1 |
| **Kategori** | forensics |
| **Jumlah Solve** | 801 |

## Ikhtisar

README membingkai artefak: map `MARCHETTI / 2026-14`, profil Firefox yang diambil dari stasiun kerja kantor Elena di Universitas Hollis, ditemukan tidak terkunci dengan halaman asal SPR di tab aktif terakhir, sesi terputus di tengah peninjauan. Analis K. Vance, dipertahankan oleh R. Marchetti. Tujuannya adalah "menemukan apa yang hendak dia serahkan".

Arsip berisi profil Firefox standar:

```
k-vance-profile/
├── places.sqlite
├── logins.json
├── formhistory.sqlite
├── prefs.js
└── sessionstore-backups/
    └── recovery.jsonlz4
```

"Akan dikirim" mengesampingkan file yang sudah jelas. `places.sqlite` menyimpan riwayat dan bookmark, `logins.json` menyimpan kredensial yang disimpan, dan `formhistory.sqlite` hanya menerima nilai dari formulir yang benar-benar dikirimkan, untuk pelengkapan otomatis. Formulir yang dibiarkan setengah terisi tidak ada satupun.

`prefs.js` menjelaskan mengapa data tetap ada di disk:

```js
user_pref("browser.startup.page", 3);
user_pref("browser.sessionstore.resume_from_crash", true);
```

Restore-on-startup plus crash recovery berarti Firefox secara berkala membuat serial status terkini setiap tab — URL, posisi gulir, dan konten langsung bidang formulir — ke dalam `sessionstore-backups/recovery.jsonlz4`, baik ada yang pernah dikirimkan atau tidak.

## Penyelesaian

File pemulihan bukan aliran LZ4 normal. Firefox menggunakan `mozLz4`: `mozLz40\0` ajaib 8-byte, panjang little-endian 4-byte dari payload yang didekompresi, dan kemudian blok LZ4. Header 12-byte itulah yang menyebabkan `cat` dan `lz4 -d` keduanya gagal.

```python
import json, lz4.block

raw = open("k-vance-profile/sessionstore-backups/recovery.jsonlz4", "rb").read()
assert raw[:8] == b"mozLz40\0"
sess = json.loads(lz4.block.decompress(raw[12:]))

for w in sess["windows"]:
    for t in w["tabs"]:
        for e in t["entries"]:
            if e.get("formdata"):
                print(e["url"])
                for k, v in e["formdata"].items():
                    print(f"  {k:<14} = {v}")
```

`dejsonlz4` melakukan pembukaan bungkus yang sama jika diinstal. Tab tunggal dengan data yang belum terkirim adalah entri katalog SPR:

```
https://catalog.spr.org.uk/apparatus/provenance/lssf
```

dengan empat bidang:

| Field | Value |
|---|---|
| `id` | `hollis-office-desktop-elena` |
| `analyst-name` | `K. Vance` |
| `artifact-flag` | `POCTF{...}` |
| `case-notes` | Subject: E. Marchetti disappearance. Chain of custody initiated 2026-06-29. See docket MARCHETTI/2026-14. |

`artifact-flag` adalah flag-nya:

```
POCTF{██████████████████████████████████████████████████}
```

Sebagai pemeriksaan kewarasan, `cat` pada `.jsonlz4` mentah sudah menunjukkan fragmen JSON yang dapat dibaca - LZ4 menyimpan byte yang tidak cocok sebagai literal, sehingga string pendek bertahan kata demi kata di dalam blok terkompresi bahkan sebelum dekompresi.

Pernyataan singkat "perhatikan detailnya" bukanlah omong kosong. `logins.json` berisi dua entri yang `encryptedUsername` dan `encryptedPassword` tidak dienkripsi sama sekali, hanya base64:

```
hollis.edu   → e.marchetti@hollis.edu
pastebin.com → anon-analyst / hunter2
```

Semua ini tidak diperlukan untuk bendera, tetapi akun tersebut berada di profil yang sama, dan akun "anon-analis" adalah akun yang akan diambil pada tahap selanjutnya.