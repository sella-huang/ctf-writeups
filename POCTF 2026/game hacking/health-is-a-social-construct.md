# Health Is a Social Construct

| | |
|---|---|
| **Bagian** | wave 4 |
| **Kategori** | game hacking |
| **Jumlah Solve** | 206 |

## Ikhtisar

Unduhannya adalah ekspor Godot 4.7.1 x86-64. Logika permainan disimpan di PCK tertanam pada file offset `0x4611138`, yang berisi 398 file. Skrip yang relevan adalah `scenes/auth_check.gdc`, `scenes/game/game_controller.gdc`, `scenes/game/room03.gdc`, `scenes/actors/player.gdc`, `scenes/actors/enemy_basic.gdc`, `FlagToken.gdc`, dan `Ticker.gdc`.

Game mengautentikasi dengan `game.token` dan `team.key`, memanggil `GET /api/challenges/48/instance` untuk mendapatkan nonce, lalu memulai ticker HMAC 2,5 detik. Ruang terakhir memunculkan bos dengan `max_hp=3000` dan `attack_damage=999`, sedangkan pemain memiliki `max_hp=100`, jadi permainan biasa tidak bisa menang. Saat bos mati, `room03` mengeluarkan `room_cleared`; `game_controller._finalize_run()` memanggil `FlagService.request_marker()`, yang POST ke `/api/challenges/48/complete` dengan header autentikasi dan nonce. `marker` yang dikembalikan ditampilkan oleh overlay bendera.

Pengamatan penting adalah bahwa titik akhir penyelesaian tidak menerima bukti kemenangan. Ini hanya memvalidasi kredensial tim dan nonce instans. Ticker tidak diperlukan untuk penyelesaian, dan server tidak mengetahui HP bos, status ruangan, atau apakah klien pernah diluncurkan. "Konstruk sosial" adalah pemeriksaan kemenangan sisi klien itu sendiri: lewati saja dan minta penandanya pada server.

## Penyelesaian

Ekstrak kredensial dari `credentials.zip` dan minta instance nonce:

```http
GET /api/challenges/48/instance HTTP/1.1
Host: pointeroverflowctf.com
X-POCTF-GAME-TOKEN: ████████████████
X-POCTF-TEAM-KEY: ████████████████
```

Responsnya berisi nonce yang digunakan oleh game:

```json
{"challenge_id":48,"download_count":0,"nonce":"████████████████","ok":true,"team_id":1814}
```

Kemudian panggil titik akhir penyelesaian yang sama yang digunakan `FlagToken.gd`, tanpa harus memulai permainan:

```http
POST /api/challenges/48/complete HTTP/1.1
Host: pointeroverflowctf.com
X-POCTF-GAME-TOKEN: ████████████████
X-POCTF-TEAM-KEY: ████████████████
X-POCTF-CHALLENGE-NONCE: ████████████████
Content-Type: application/json

{}
```

Server mengembalikan penanda:

```json
{"challenge_id":48,"marker":"████████████████","team_id":1814}
```

Mengirimkan penanda itu ke halaman tantangan akan menghasilkan `{"correct":true,"message":"Landed. Nicely done."}`.

Rute sisi klien yang dimaksud juga terlihat di skrip. `room03.boss_max_hp` dan `boss_attack_damage` adalah variabel yang diekspor, dan `enemy_basic.apply_damage()` memercayai argumennya. Debugger atau PCK yang dipatch dapat menyetel HP bos ke 1, menaikkan HP pemain, atau memanggil `apply_damage(999999)` pada bos. Itu akan membuat `room03` memancarkan `room_cleared`, mencapai `_finalize_run()`, dan mencetak penanda yang sama melalui hamparan normal. Namun titik akhir adalah tautan yang lebih lemah: karena `complete` tidak memverifikasi bahwa semua itu terjadi, permainan dapat dilewati seluruhnya.

Flag yang diterima adalah ████████████████.