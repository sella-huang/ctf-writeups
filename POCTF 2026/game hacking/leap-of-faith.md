# Leap of Faith

| | |
|---|---|
| **Bagian** | wave 2 |
| **Kategori** | game hacking |
| **Jumlah Solve** | 352 |

## Ikhtisar

Unduhannya adalah ekspor Godot 4.7.1 x86-64. PCK tertanam dimulai pada file offset `0x4611138` dan berisi 79 file. Skrip yang relevan adalah `player.gd`, `goal.gd`, `GameController.gd`, `SceneManager.gd`, `end_screen.gd`, `auth_check.gd`, dan `FlagToken.gd`.

`player.gd` adalah `CharacterBody3D` dengan `move_speed=6.0`, `jump_velocity=17.5`, dan `gravity=18.0` yang diekspor. Platform terakhir sengaja di luar jangkauan lompatan itu. `goal.gd` adalah `Area3D` yang mengeluarkan `goal_reached` saat tubuh pemain memasukinya. `GameController.gd` menghubungkan sinyal tersebut ke `Scenes.goto_end()`, yang memuat `end_screen.tscn`. Layar akhir segera memanggil `FlagService.request_marker()`, dan `FlagToken.gd` POST ke `/api/challenges/47/complete` dengan header tim dan nonce dari `/api/challenges/47/instance`.

Halaman tantangan mencatat bahwa biner yang diunggah sebenarnya adalah uji coba dan meremehkan lompatan yang dimaksudkan. Hal ini berguna, namun titik akhir penyelesaian adalah tautan yang lebih lemah: titik akhir tersebut tidak menerima bukti bahwa pemain pernah mencapai tujuan, menyelesaikan level, atau bahkan meluncurkan permainan. Ini hanya memvalidasi kredensial dan nonce instans.

## Penyelesaian

Minta nonce dengan kredensial dari `credentials.zip`:

```http
GET /api/challenges/47/instance HTTP/1.1
Host: pointeroverflowctf.com
X-POCTF-GAME-TOKEN: ████████████████
X-POCTF-TEAM-KEY: ████████████████
```

Responsnya berisi nonce:

```json
{"challenge_id":47,"download_count":0,"nonce":"████████████████","ok":true,"team_id":1814}
```

Kemudian panggil titik akhir penyelesaian yang sama yang digunakan oleh `FlagToken.gd`:

```http
POST /api/challenges/47/complete HTTP/1.1
Host: pointeroverflowctf.com
X-POCTF-GAME-TOKEN: ████████████████
X-POCTF-TEAM-KEY: ████████████████
X-POCTF-CHALLENGE-NONCE: ████████████████
Content-Type: application/json

{}
```

Server mengembalikan penanda:

```json
{"challenge_id":47,"marker":"████████████████","team_id":1814}
```

Mengirimkan penanda ke halaman tantangan akan menghasilkan `{"correct":true,"message":"Landed. Nicely done."}`.

Rute sisi permainan yang dimaksudkan juga jelas dari skripnya. `jump_velocity` adalah properti yang diekspor, sehingga debugger atau PCK yang di-patch dapat meningkatkannya cukup untuk menghilangkan kesenjangan terakhir. Alternatifnya, menyetel `global_position` pemain ke posisi gawang akan membuat area gawang menembakkan `goal_reached`, yang mengarah ke layar akhir dan permintaan penanda yang sama. Perubahan mana pun bersifat lokal bagi klien; server tidak memverifikasi lompatan.

Flag yang diterima adalah ████████████████.