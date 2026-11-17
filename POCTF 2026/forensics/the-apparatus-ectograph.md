# The Apparatus, Ectograph

| | |
|---|---|
| **Bagian** | wave 6 |
| **Kategori** | forensics |
| **Jumlah Solve** | 183 |

## Ikhtisar

Direktori data Threshold yang dipulihkan berisi `threshold.db`, `config.json`, folder `attachments/`, dan dua file teks lepas. Skema SQLite kecil: `users`, `devices`, `sessions`, `threads`, `thread_participants`, `messages`, `message_edits`, `reactions`, dan `attachments`. Ada 58 pesan di sepuluh thread, 21 baris perangkat, dan 40 sesi; `message_edits` dan `reactions` kosong. Tabel yang penting adalah `messages`, karena setiap baris menyimpan `sender_id` — akun — dan `sender_device_id`, klien yang sebenarnya mengirimkannya. Perintah tersebut menjelaskan dengan tepat ketidakcocokan antara dua kolom tersebut: pesannya terlihat seperti akun Elena, namun tidak seperti salah satu perangkat Elena.

Masalah satu jam harus diselesaikan sebelum pertanyaan apa pun. Perintahnya menggunakan `2026-06-28 22:47`, tetapi nilai `sent_at` database adalah periode 2024. Rentangnya adalah `1719080400`–`1719962400`, atau `2024-06-22 18:20:00`–`2024-07-02 23:20:00` UTC. `WHERE sent_at >= 1782686820` literal mengembalikan nol baris. Batasan yang berguna bukanlah stempel waktu naratif; itu adalah pesan terakhir dari perangkat asli Elena, setelah itu pesan akun berikutnya datang dari klien yang berbeda.

## Penyelesaian

Mulailah dengan tabel perangkat untuk `u_elena` / `@e.marchetti`:

```bash
sqlite3 threshold.db <<'SQL'
.headers on
.mode column
SELECT id, device_name, device_fingerprint,
       datetime(registered_at, 'unixepoch') AS registered,
       datetime(last_seen_at, 'unixepoch')  AS last_seen,
       is_active
FROM devices
WHERE user_id = 'u_elena'
ORDER BY registered_at;
SQL
```

Hasilnya adalah empat baris:

| device id | device name | registered (UTC) | last seen (UTC) | active |
|---|---|---|---|---|
| `6XQDEDUHPWFMNMVE` | Elena's MacBook Pro | 2024-02-03 22:40:00 | 2024-07-02 17:20:00 | 1 |
| `GZSNOSIHIU62F773` | Elena's iPhone 15 Pro | 2024-02-03 22:50:00 | 2024-07-02 17:50:00 | 1 |
| `████████████████` | Elena's MacBook | 2024-07-02 18:21:00 | 2024-07-03 02:00:00 | 1 |
| `DWA2CYMFMTOEFMWK` | Elena's iPad | 2024-07-03 09:46:40 | 2024-06-27 18:20:00 | 0 |

Tiga baris memiliki aktivitas sebelum kejadian: MacBook Pro, iPhone, dan iPad, yang `last_seen_at`-nya adalah `2024-06-27 18:20:00`. `registered_at` iPad lebih baru dari `last_seen_at`, yang tidak mungkin dilakukan untuk perangkat yang benar-benar baru; itu tidak aktif dan, seperti yang ditunjukkan di bawah, tidak memiliki sesi dan pesan. Anomali itu adalah umpannya. Baris keempat, `████████████████`, tidak memiliki aktivitas sebelum insiden: baris tersebut dibuat di `2024-07-02 18:21:00`, setelah pesan asli terakhir Elena, dan merupakan satu-satunya perangkat yang mengirim pesan setelah titik tersebut.

Kelompokkan pesan Elena berdasarkan perangkat pengirim:

```sql
SELECT sender_device_id, COUNT(*) AS n,
       datetime(MIN(sent_at), 'unixepoch') AS first,
       datetime(MAX(sent_at), 'unixepoch') AS last
FROM messages
WHERE sender_id = 'u_elena'
GROUP BY sender_device_id;
```

Outputnya hanya dua perangkat:

```text
6XQDEDUHPWFMNMVE  20  2024-06-22 19:20:00  2024-07-02 18:00:01
████████████████   3  2024-07-02 19:20:00  2024-07-02 23:20:00
```

MacBook Pro mengirim 20 pesan dan berhenti di `2024-07-02 18:00:01`. Pesan terakhir itu adalah `m_f4vcbsc2yesi`, pesan lampiran untuk `20260628_ectograph_transcript.txt`. Tiga pesan berikutnya dari akun Elena semuanya dari `████████████████`:

| sent (UTC) | thread | body | read at |
|---|---|---|---|
| 2024-07-02 19:20:00 | `th_delacroix` | "You're right. Deleting. Please stop contacting me. I'm dropping the research. Delete everything I sent you." | `NULL` |
| 2024-07-02 20:20:00 | `th_rennick` | "Aldous, I've thought about it. I'm withdrawing from the program effective immediately. I'll be unreachable for a few weeks. Please don't try to contact me." | `NULL` |
| 2024-07-02 23:20:00 | `th_renata` | "Ren I'm going up to Marion's for the weekend. Research overnight, phone off. Don't worry." | `NULL` |Ini adalah tiga pesan yang dijelaskan oleh prompt. Mereka dikoordinasikan di tiga thread berbeda, semuanya dalam waktu empat jam, dan semuanya dari perangkat non-asli yang sama. Isinya juga konsisten dengan upaya untuk menutup penyelidikan: pesan Delacroix menghancurkan rangkaian transkrip, pesan Rennick menjelaskan penarikan mendadak, dan pesan Renata memberikan lokasi palsu dan alasan untuk tidak dapat dijangkau. Nilai `read_at` adalah `NULL`; pesan asli telah dibaca, jadi ketidakhadirannya merupakan inkonsistensi kecil lainnya.

Baris perangkat dan sesinya menjelaskan mengapa `████████████████` adalah pena:

```sql
SELECT id, device_name, device_fingerprint,
       datetime(registered_at, 'unixepoch') AS registered,
       datetime(last_seen_at, 'unixepoch')  AS last_seen,
       is_active
FROM devices WHERE id = '████████████████';

SELECT datetime(started_at, 'unixepoch') AS started, ip_address
FROM sessions WHERE device_id = '████████████████'
ORDER BY started_at;
```

Perangkat tersebut diberi nama "Elena's MacBook", sidik jari `MacBookAir-M2 macOS 14.5`, terdaftar di `2024-07-02 18:21:00` dan terakhir terlihat di `2024-07-03 02:00:00`. Pendaftarannya sekitar 21 menit setelah pesan terakhir dari `6XQDEDUHPWFMNMVE`, dan sekitar 59 menit sebelum pesan palsu pertama. Itu adalah jendela pendaftaran. Sesinya adalah:

```text
2024-07-02 18:21:00   203.0.113.113
2024-07-05 18:21:00   203.0.113.66
```

Bukti jaringan sangat bersih: 38 dari 40 sesi dalam database menggunakan `192.0.2.x`, sedangkan dua sesi untuk `████████████████` menggunakan `203.0.113.x`. Kedua rentang dokumentasi — TEST-NET-1 dan TEST-NET-3 — sengaja dipisahkan. Perangkat telah didaftarkan, segera membuka sesi dari rentang kedua, mengirim tiga pesan, dan terlihat lagi tiga hari kemudian dari rentang yang sama. Sesi kedua adalah setelah pesan terakhir database, sehingga perangkat tetap aktif setelah pemalsuan.

IPad adalah jalan buntu utama. `DWA2CYMFMTOEFMWK` memiliki `last_seen_at = 2024-06-27 18:20:00` tetapi `registered_at = 2024-07-03 09:46:40`, ditandai `is_active = 0`, dan tidak memiliki sesi atau pesan:

```sql
SELECT COUNT(*) FROM messages WHERE sender_device_id = 'DWA2CYMFMTOEFMWK'; -- 0
SELECT COUNT(*) FROM sessions WHERE device_id = 'DWA2CYMFMTOEFMWK';        -- 0
```

Jika satu-satunya kriteria adalah “perangkat yang seharusnya tidak ada”, iPad sepertinya jawabannya karena stempel waktunya tidak mungkin. Bukan: ia tidak menulis satu pun dari ketiga pesan tersebut. Kolom yang menentukan adalah `messages.sender_device_id`, bukan `devices.registered_at`. IPad adalah umpan, dan mengirimkan ID-nya akan mengembalikan `Incorrect`.

Lampiran dan file teks yang lepas juga merupakan umpan. Hanya satu lampiran yang benar-benar direferensikan oleh sebuah pesan: `att_ecto_cpvzcjyj` pada `m_f4vcbsc2yesi`, transkrip ektograf. Empat lainnya — `20260615_rennick_deadline.png`, `20260620_redoubt_receipt.jpg`, `20260625_spr_confirmation.pdf`, dan `20260626_yee_subtweet.png` — memiliki `message_id = NULL`. Itu adalah file rasa yatim piatu, bukan bagian dari grafik pesan. Nilai `content_hash`-nya terdiri dari 16 karakter dan terlihat seperti base32, namun merupakan hash konten, bukan ID perangkat; hal yang sama juga berlaku untuk ID sesi 16 karakter. `unaccounted_transmission.txt` adalah fragmen ASCII out-of-band dan tidak berisi pengenal perangkat.Transkripnya adalah petunjuk naratif, bukan muatannya:

```text
The hand is not the medium's.
The signal comes from behind the wall.
The pen has never left the drawer.
The words were spoken in another room.
```

Itu adalah pengamatan yang sama yang dilakukan oleh database: pengetikan tangan bukan milik Elena, pena bukanlah salah satu perangkatnya, dan kata-kata itu berasal dari ruangan lain. `README.txt` menyatakannya secara lebih langsung: "Tidak semua yang Anda baca di sini dikatakan oleh siapa yang melihatnya."

Pemutusan stempel waktu adalah jalan buntu lainnya. Mengkueri zaman literal 2026 tidak menghasilkan apa-apa, dan menanyakan zaman 28-06-2024 22:47 mengembalikan banyak pesan asli dari `6XQDEDUHPWFMNMVE`; itu tidak mengisolasi ketiga yang palsu. Poros yang andal adalah peralihan perangkat: setelah pesan `6XQDEDUHPWFMNMVE` terakhir, akun tersebut memiliki tepat tiga pesan, dan ketiganya membawa `sender_device_id` baru yang sama.

Perangkat nakal tersebut adalah `████████████████`. Ini adalah 16 karakter dan base32 yang valid. Di halaman artefak, mengirimkan token mentah atau `POCTF{████████████████}` akan mengembalikan:

```json
{"correct":true,"message":"Correct."}
```

Penanda lengkapnya adalah `POCTF{████…}`.

Pena yang bukan mediumnya adalah `████████████████`, dan flag-nya adalah `POCTF{████…}`.