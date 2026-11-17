# The Apparatus, Levitation

| | |
|---|---|
| **Bagian** | wave 5 |
| **Kategori** | web |
| **Jumlah Solve** | 168 |

## Ikhtisar

Portal ruang tamu LSSF menukar token sesi tim dengan JWT bertanda tangan yang membawa klaim peran. Hanya peran `SPIRIT` yang dapat membaca penanda yang dipulihkan dari titik akhir transkrip. Aplikasi ini memverifikasi token dengan kunci publik RSA tetapi juga menerima token yang ditandatangani HMAC, menggunakan materi kunci publik yang sama dengan rahasia HMAC — sebuah kelemahan kebingungan algoritma yang memungkinkan token `SPIRIT` dipalsukan.

## Penyelesaian

Halaman tantangan mengeluarkan token tim berumur pendek. Mempostingnya ke titik akhir sesi portal membuka sesi `MEDIUM`:

```http
POST /session HTTP/1.1
Host: levitation.pointeroverflowctf.com
Content-Type: application/json

{"token": "<team session token>"}
```

Responsnya menetapkan cookie `levitation_session` yang berisi RS256 JWT dengan header `{"alg":"RS256","typ":"JWT","kid":"spirit-key-1"}` dan muatan yang membawa `team_id`, nonce per tim, dan `"role":"medium"`.

Portal mengiklankan materi verifikasinya di `/.well-known/jwks.json`, dan juga menampilkan kunci yang sama dengan PEM mentah di `/spirit-key.pem`. File itu berisi kunci publik RSA normal. Namun, pemverifikasi menerima token HS256 yang rahasia HMAC-nya merupakan konten persis dari file PEM publik tersebut, termasuk baris baru di belakangnya. Dengan header yang masih bernama `kid: spirit-key-1`, kebingungan algoritma diterima dan klaim peran dipercaya.

Saya membuat token pengganti dengan `team_id` yang sama dan klaim nonce yang sama dengan sesi yang dikeluarkan, tetapi dengan `"role":"spirit"`, dan menandatanganinya sebagai HS256 menggunakan byte PEM mentah sebagai kuncinya:

```python
header  = {"alg": "HS256", "typ": "JWT", "kid": "spirit-key-1"}
payload = {"team_id": 1814, "nonce": "<issued nonce>", "role": "spirit", ...}
signing_input = b64url(json(header)) + "." + b64url(json(payload))
token = signing_input + "." + b64url(hmac_sha256(pem_bytes, signing_input))
```

Meminta `/api/transcript` dengan cookie palsu ini akan mengembalikan transkrip `SPIRIT` dan bukan teks kehadiran medium saja. Transkrip diakhiri dengan penanda yang dipulihkan, yang berhasil dikirimkan.

Flag yang dikirimkan adalah POCTF{80.1814.████████████████.████████████████} dan diterima.