# The House Always Misses

| | |
|---|---|
| **Bagian** | wave 5 |
| **Kategori** | game hacking |
| **Jumlah Solve** | 141 |

## Ikhtisar

Unduhannya adalah ekspor Godot 4.7.1 x86-64. PCK tertanam dimulai pada file offset `0x4643178` dan berisi 111 file. Skrip yang relevan adalah `scenes/game/gallery.gd`, `scenes/actors/target_base.gd`, subkelas target (`flip_target.gd`, `popup_target.gd`, `duck.gd`, `civilian.gd`), `FlagToken.gd`, dan `auth_check.gd`.

Galeri memberi pemain waktu 60 detik untuk mencapai skor 1200. Target flip bernilai 25 poin, target popup 50, bebek 10, dan warga sipil dikurangi 50. Sebuah tembakan diselesaikan sepenuhnya pada klien di `gallery.gd:_click_at()`:

```gdscript
var query := PhysicsPointQueryParameters2D.new()
query.position = pos
query.collision_mask = 1
query.collide_with_areas = true
query.collide_with_bodies = false
var results := space.intersect_point(query, 8)
```

`TargetBase` pertama dalam hasil yang belum tercapai menerima `on_hit()`. Geometri kueri itulah yang menentukan apakah suatu tembakan akan mendarat.

Saat mencapai ambang batas, `gallery.gd:_win()` memanggil `FlagService.request_marker()`. `FlagToken.gd` lalu POST ke `/api/challenges/49/complete` dengan header tim dan nonce dari `/api/challenges/49/instance`.

Petunjuk tantangan menunjuk pada tes pukulan, namun titik akhir penyelesaian bahkan lebih langsung: ia tidak menerima skor, jumlah pukulan, atau bukti apa pun bahwa galeri telah dimainkan. Itu hanya memeriksa kredensial dan nonce instance.

## Penyelesaian

Minta nonce dengan kredensial dari `credentials.zip`:

```http
GET /api/challenges/49/instance HTTP/1.1
Host: pointeroverflowctf.com
X-POCTF-GAME-TOKEN: ████████████████
X-POCTF-TEAM-KEY: ████████████████
```

Responsnya berisi nonce:

```json
{"challenge_id":49,"download_count":0,"nonce":"████████████████","ok":true,"team_id":1814}
```

Kemudian panggil titik akhir penyelesaian yang sama yang digunakan oleh `FlagToken.gd`:

```http
POST /api/challenges/49/complete HTTP/1.1
Host: pointeroverflowctf.com
X-POCTF-GAME-TOKEN: ████████████████
X-POCTF-TEAM-KEY: ████████████████
X-POCTF-CHALLENGE-NONCE: ████████████████
Content-Type: application/json

{}
```

Server mengembalikan penanda:

```json
{"challenge_id":49,"marker":"████████████████","team_id":1814}
```

Mengirimkan penanda ke halaman tantangan akan menghasilkan `{"correct":true,"message":"Landed. Nicely done."}`.

Untuk rute sisi klien yang dimaksud, ganti kueri titik di `_click_at()` dengan `PhysicsShapeQueryParameters2D` menggunakan `CircleShape2D` yang besar, sehingga setiap target yang diklik dikumpulkan, bukan hanya piksel persis di bawah kursor. Debugger juga dapat mengatur `gallery._score` ke `score_threshold`, atau menghubungi `_win()` secara langsung. Geometri hit game ini adalah kebohongan; mengubah geometri itu akan mengubah apa yang dianggap sebagai pukulan.

Flag yang diterima adalah ████████████████.